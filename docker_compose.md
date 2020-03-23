Dockerfile
```
version: "3"
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432" // HOST_PORT:CONTAINER_PORT
```

```shell
> pwd
  myapp  
> docker-compose up
```
- A network called myapp_default is created.
- A container is created using web’s configuration. It joins the network myapp_default under the name web.
- A container is created using db’s configuration. It joins the network myapp_default under the name db.

web to db: `postgres://db:5432`
host to db: `postgres://{DOCKER_IP}:8001`

