Untuk memastikan solusi yang diberikan sesuai dengan soal yang diminta, mari kita ulang langkah-langkahnya dan sesuaikan dengan detail dari soal.

### Detail Tugas

1. **Tambahkan 2 instance aplikasi sehingga total menjadi 5 instance.**
2. **Endpoint `/slow` harus memiliki sleep 2 detik, dan endpoint `/fast` harus memiliki sleep 0.5 detik.**
3. **Gunakan nama database "MyDatabase_C09" dengan koleksi "C_Sembilan".**
4. **Masukkan minimal 5 data acak ke dalam koleksi dengan struktur:**
   - `name`: string
   - `age`: int
   - `favorite_game`: string
   - `genre`: list
   - `playHours`: int

### Langkah-Langkah Implementasi

#### 1. Struktur Folder dan File

```
my_web_app/
├── docker-compose.yml
├── nginx/
│   ├── Dockerfile
│   ├── nginx.conf
├── app/
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
├── app2/
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
├── app3/
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
├── app4/
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
├── app5/
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
└── locustfile.py
```

#### 2. Dockerfile untuk Aplikasi

**app/Dockerfile, app2/Dockerfile, app3/Dockerfile, app4/Dockerfile, app5/Dockerfile:**

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### 3. main.py untuk Aplikasi

**app/main.py, app2/main.py, app3/main.py, app4/main.py, app5/main.py:**

```python
from fastapi import FastAPI, Body, Request
from fastapi.encoders import jsonable_encoder
import pymongo
from pydantic import BaseModel
from bson.objectid import ObjectId
import socket
import time

app = FastAPI()

MONGO_DETAILS = "mongodb://admin:admin@mongodb:27017/"
client = pymongo.MongoClient(MONGO_DETAILS)
db = client['MyDatabase_C09']
collection = db['C_Sembilan']

class Item(BaseModel):
    name: str
    age: int 
    favorite_game: str
    genre: list
    playHours: int

@app.get('/')
async def home():
    return {"message": "This is server", "hostname": socket.gethostname()}

@app.get('/fast')
async def fast():
    time.sleep(0.5)
    return {"message": "Hello world", "opt": "fast"}

@app.get('/slow')
async def slow():
    time.sleep(2)
    return {"message": "Hello world", "opt": "slow"}

@app.get("/all")
async def get_all_data():
    return {"data": list(collection.find())}

@app.post("/create", response_model=Item)
async def create_data(request: Request, lister: Item = Body(...)):
    lister = jsonable_encoder(lister)
    new_data = collection.insert_one(lister)
    return lister

@app.get("/get/{id}")
async def get_data(id: str):
    obj_id = ObjectId(id)
    data = collection.find_one({"_id": obj_id})
    if data:
        return data
    else:
        return {"error": "Data doesn't exist"}
```

#### 4. requirements.txt

**app/requirements.txt, app2/requirements.txt, app3/requirements.txt, app4/requirements.txt, app5/requirements.txt:**

```
fastapi==0.78.0
uvicorn==0.18.2
pymongo
pydantic
uuid
```

#### 5. Dockerfile untuk Nginx

**nginx/Dockerfile:**

```Dockerfile
FROM nginx
RUN rm /etc/nginx/conf.d/default.conf
COPY ./nginx.conf /etc/nginx/conf.d/default.conf
```

#### 6. Konfigurasi Nginx

**nginx/nginx.conf:**

```nginx
upstream app {
    server app:8000 weight=1;
    server app2:8000 weight=2;
    server app3:8000 weight=3;
    server app4:8000 weight=4;
    server app5:8000 weight=5;
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://app;
    }
}
```

#### 7. docker-compose.yml

**docker-compose.yml:**

```yaml
version: '3'

services:
  app:
    build: ./app
    ports:
      - "8001:8000"
    depends_on:
      - mongodb

  app2:
    build: ./app2
    ports:
      - "8002:8000"
    depends_on:
      - mongodb

  app3:
    build: ./app3
    ports:
      - "8003:8000"
    depends_on:
      - mongodb  

  app4:
    build: ./app4
    ports:
      - "8004:8000"
    depends_on:
      - mongodb

  app5:
    build: ./app5
    ports:
      - "8005:8000"
    depends_on:
      - mongodb

  nginx:
    build: ./nginx
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - app
      - app2
      - app3
      - app4
      - app5

  mongodb:
    image: mongo
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin
    volumes:
      - mongodb_data:/data/db

  mongo-express:
    image: mongo-express
    ports:
      - "8082:8081"
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=admin
      - ME_CONFIG_MONGODB_SERVER=mongodb
      - ME_CONFIG_MONGODB_ENABLE_ADMIN=true
      - ME_CONFIG_BASICAUTH_USERNAME=admin
      - ME_CONFIG_BASICAUTH_PASSWORD=admin
    depends_on:
      - mongodb

volumes:
  mongodb_data:
    driver: local
```

### Menjalankan dan Menguji Sistem

1. **Jalankan Docker Compose**:

```bash
docker-compose up -d --build
```

2. **Masukkan Data ke MongoDB**:

   Akses MongoDB Express di `http://localhost:8082` dan masukkan data sesuai format yang diminta.

### Pengujian Kinerja Menggunakan Locust

1. **Instal Locust**:

```bash
pip install locust
```

2. **Buat file locustfile.py**:

**locustfile.py:**

```python
from locust import HttpUser, task, between

class HelloWorldUser(HttpUser):
    wait_time = between(0.5, 2.5)

    @task
    def test_index(self):
        self.client.get('/')

    @task
    def test_fast(self):
        self.client.get('/fast')

    @task
    def test_slow(self):
        self.client.get('/slow')

    @task
    def test_get_all(self):
        self.client.get('/all')

    @task
    def test_get_id(self):
        self.client.get('/get/[ID]')
```

3. **Jalankan Locust**:

```bash
locust -f locustfile.py --host=http://localhost
```

4. **Akses Locust Web Interface** di `http://localhost:8089`, masukkan jumlah pengguna dan laju spawn, dan mulai pengujian.

### Analisis Hasil Pengujian

Setelah pengujian selesai, analisis hasil pengujian dari dashboard Locust untuk melihat metrik berikut:

- **Requests per Second (RPS)**: Mengukur berapa banyak permintaan yang dapat ditangani oleh server per detik.
- **Response Time**: Mengukur waktu yang dibutuhkan server untuk merespon permintaan.

Bandingkan hasil antara konfigurasi RoundRobin dan Weighted untuk mengidentifikasi konfigurasi yang lebih efisien.

Dengan langkah-langkah ini, Anda seharusnya bisa menyelesaikan tugas yang diberikan sesuai dengan spesifikasi yang diminta. Jika ada pertanyaan lebih lanjut atau bantuan tambahan yang diperlukan, silakan tanyakan!
