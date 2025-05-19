# geeks3-api-gateway
GEEKS 3 : API Gateway

## Lab 0 : docker image pull
```sh
sudo docker image pull postgres:13
sudo docker image pull kong:3.7.0
```
## Lab 1 : ติดตั้ง KONG
สร้าง folder ใหม่ ชื่อ kong
```
mkdir kong
cd kong
```
สร้างไฟล์ docker-compose.yml ด้วยเนื้อหาดังนี้
```yml
version: '3'
volumes:
  kong_data: {}
networks:
  kong-net:
    external: false
services:
  kong-database:
    image: postgres:13
    container_name: kong-database
    networks:
      - kong-net
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kongpass
    volumes:
      - kong_data:/var/lib/postgresql/data
  kong-migrations:
    image: kong:3.7.0
    command: kong migrations bootstrap
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_PASSWORD: kongpass
    depends_on:
      - kong-database
    networks:
      - kong-net
  kong:
    image: kong:3.7.0
    networks:
      - kong-net
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kongpass
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001, 0.0.0.0:8444 ssl
      KONG_ADMIN_GUI_URL: http://localhost:8002
    ports:
      - 8000:8000
      - 8443:8443
      - 8001:8001
      - 8002:8002
      - 8444:8444
```
ใช้คำสั่ง
```sh
docker-compose up -d
```
ถ้าการติดตั้งสมบูรณ์จะสามารถเข้าดู KONG manager ได้ผ่าน Web browser ที่
```
http://localhost:8002
```

# Lab เพิ่มเติม : Admin API

```python
import requests

# ตั้งค่าพื้นฐาน
KONG_ADMIN_URL = "http://localhost:8001"
consumer_username = "example_user"
api_key_value = "my-custom-api-key"  # หรือ None ให้ Kong generate ให้เอง

# 1. สร้าง Consumer
def create_consumer(username):
    url = f"{KONG_ADMIN_URL}/consumers"
    payload = {
        "username": username
    }
    response = requests.post(url, json=payload)

# 2. สร้าง API Key ให้ Consumer
def create_api_key(username, custom_key=None):
    url = f"{KONG_ADMIN_URL}/consumers/{username}/key-auth"
    payload = {}
    if custom_key:
        payload["key"] = custom_key
    response = requests.post(url, json=payload)

# 3. แสดง API Key ทั้งหมดของ Consumer
def list_api_keys(username):
    url = f"{KONG_ADMIN_URL}/consumers/{username}/key-auth"
    response = requests.get(url)
    if response.status_code == 200:
        data = response.json()
        keys = data.get("data", [])
        if keys:
            for i, key_data in enumerate(keys, start=1):
                print(f"{key_data['key']}")

# เรียกใช้งาน
create_consumer(consumer_username)
create_api_key(consumer_username, api_key_value)
list_api_keys(consumer_username)

```
