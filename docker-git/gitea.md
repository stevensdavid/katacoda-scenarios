# GUI it!
There are multiple alternatives ...

```yml
version: '2'

services:
  postgres:
    image: postgres:9.5
    restart: always
    environment:
     - "POSTGRES_USER=gogsuser"
     - "POSTGRES_PASSWORD=gogspassword"
     - "POSTGRES_DB=gogs"
    volumes:
     - "db-data:/var/lib/postgresql/data"
  gogs:
    image: gogs/gogs:latest
    restart: always
    ports:
     - "10022:22"
     - "3000:3000"
    links:
     - postgres
    environment:
     - "RUN_CROND=true"
    volumes:
     - "gogs-data:/data"
    depends_on:
     - postgres


volumes:
    db-data:
      driver: local
    gogs-data:
      driver: local
```{{copy}}

https://[[HOST_SUBDOMAIN]]-3000-[[KATACODA_HOST]].environments.katacoda.com

`docker-compose up -d`{{execute T1}}
