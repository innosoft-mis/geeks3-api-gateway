# geeks2-api-gateway
GEEKS 2 : API Gateway

## Lab 1 : ติดตั้ง KONG

https://docs.konghq.com/gateway/3.7.x/install/docker/?install=oss

```sh
sudo docker network create kong-net
```

```sh
sudo docker run -d --name kong-database \
 --network=kong-net \
 -p 5432:5432 \
 -e "POSTGRES_USER=kong" \
 -e "POSTGRES_DB=kong" \
 -e "POSTGRES_PASSWORD=kongpass" \
 postgres:13
```

```sh
sudo docker run --rm --network=kong-net \
 -e "KONG_DATABASE=postgres" \
 -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_PASSWORD=kongpass" \
kong:3.7.0 kong migrations bootstrap
```

```sh
sudo docker run -d --name kong-gateway \
--network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_USER=kong" \
-e "KONG_PG_PASSWORD=kongpass" \
-e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
-e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
-e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
-e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
-p 8000:8000 \
-p 8443:8443 \
-p 127.0.0.1:8001:8001 \
-p 127.0.0.1:8002:8002 \
-p 127.0.0.1:8444:8444 \
kong:3.7.0
```
```sh
curl -i -X GET --url http://localhost:8001/services
```
```sh
http://localhost:8002

```
