

# DOCKER RUBY ON RAILS PARA RASPBERRY PI

Conjunto de contenedores para subir una aplicacion a produccion una aplicacion de Ruby on Rails a un servido o en una rapsberry pi imagen compilada para las plataformas de linux/amd64,linux/arm64,linux/arm/v7 Se compone de tres contenedores, Postgres 12.2, Ruby 2.6 y Ngnix 1.17

una vez clonado el repositorio copiar nuestra aplicacion dentro del directorio web


### instalar las gemas

```
docker run -it --rm --name rails_app -v $PWD/web:/app -v $PWD/_bundle:/usr/local/bundle otrodeteruel/rails260 bundle install --without development test

```

### add credentials

```
docker run -it --rm --name rails_app -e RAILS_ENV=production -v $PWD/web:/app -v $PWD/_bundle:/usr/local/bundle otrodeteruel/rails260 bash

SECRET_KEY_BASE=production_test_key EDITOR="nano" rails credentials:edit
```

pegar el contenido con las claves

```
postgres:
  base: base
  user: user_app_db
  password: xxxxxxxxxxxxxxxxxxxx


```

crear la base de datos

```
docker-compose up -d


docker exec -it db_psql_app psql -U root

CREATE USER user_app_db;
ALTER USER user_app_db WITH PASSWORD 'xxxxxxxxxxxxxxxxxxxx';
CREATE DATABASE base_app OWNER user_app_db;
grant all privileges on database base_app to root;
\c base_app;
CREATE EXTENSION IF NOT EXISTS "unaccent";
\q

```

    docker-compose up -d


crear base de datos manualmente


docker exec -it rails_app sh

```
RAILS_ENV=production rails db:migrate

RAILS_ENV=production rails db:seed

RAILS_ENV=production rails assets:precompile
```




# docker-compose


version: '3'

services:
  db_psql:
    image: postgres:12.2
    restart: always
    container_name: db_psql_app
    volumes:
      - /home/pi/app/_db_psql:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: root

  rails:
    image: otrodeteruel/rails260
    restart: always
    container_name: rails_app
    command: bundle exec rails s -e production -b '0.0.0.0'
    depends_on:
      - db_psql
    environment:
      RAILS_ENV: production

    volumes:
      - /home/pi/app/web:/app
      - /home/pi/app/_bundle:/usr/local/bundle
    tty: true

  web:
    image: otrodeteruel/ngnix1173
    restart: always
    container_name: nginx_app
    volumes:
      - /home/pi/app/web:/app
    port:
      - 80:80
    depends_on:
      - rails
