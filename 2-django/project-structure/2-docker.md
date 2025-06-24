---
layout: default
title: Docker
parent: Project structure
grand_parent: Django
nav_order: 2
---

# Deploy files

## Core technologies for structure

- `Gunicorn`
- `Postgres`
- `Nginx`
- `Docker`

For extended configuration this could include also:

- `Celery`
- `Redis`
- `ConnectionPooling`
- `SocketConfiguration`

## Main docker file

The most basic docker compose file that the Django application could have consist in three files:
- `Dockerfile` for basic packages installations
- `docker-compose.yml` for database and application server
- `backend_nginx.staging.conf` for basic `nginx` configuratio

### Dockerfile

```yaml
FROM python:3.8

WORKDIR /code

# Allows docker to cache installed dependencies and apply migrations between builds
COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Mounts the application code to the image
COPY . .

EXPOSE 8000

# Runs the server
ENTRYPOINT ["python", "manage.py"]
CMD ["runserver", "0.0.0.0:8000"]
```

### Docker compose file

In the root directory you will have two files, for local run and for staging/production run

Docker compose file for local run:

```yaml
services:
  db:
    image: postgres
    volumes:
      - minapp_volume:/var/lib/postgresql/data
    ports:
      - '5433:5432'
    env_file:
      - path: ./.env
  api:
    build: .
    volumes:
      - .:/code
    ports:
      - '8000:8000'
    env_file:
      - path: ./.env
    depends_on:
      - db
volumes:
  minapp_volume:

```

Docker compose file for local run:

```yaml
services:
  db:
    image: postgres
    volumes:
      - minapp_volume:/var/lib/postgresql/data
    ports:
      - '5433:5432'
    env_file:
      - path: ./.env
  api:
    build: .
    volumes:
      - .:/code
    ports:
      - '8000:8000'
    env_file:
      - path: ./.env
    depends_on:
      - db
volumes:
  minapp_volume:

```

Docker compose file for staging/production environment:

```yaml
version: '3'

services:
  db:
    restart: always
    image: postgres
    volumes:
      - minapp_volume:/var/lib/postgresql/data
    ports:
      - '5433:5432'
    environment:
      - POSTGRES_DB=minapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=Min3r0S!1995$%
      - PGDATA=/var/lib/postgresql/data/backend_test/
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U postgres']
      interval: 10s
      timeout: 5s
      retries: 3
    networks:
      - backend_network

  api:
    build:
      context: .
      dockerfile: Dockerfile.staging # Use the specific Dockerfile for the build
    command: >
      sh -c "python manage.py migrate && python manage.py collectstatic --no-input && gunicorn backend.wsgi:application -b 0.0.0.0:8000"
    volumes:
      - .:/code
    environment:
      - POSTGRES_DB=minapp
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=Min3r0S!1995$%
      - POSTGRES_HOST=db
      - POSTGRES_PORT=5432
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend_network

  nginx:
    image: nginx:latest
    restart: always
    volumes:
      - ./backend_nginx.staging.conf:/etc/nginx/conf.d/default.conf
      - ./static:/code/static
    ports:
      - '80:80'
    depends_on:
      - api
    networks:
      - backend_network

volumes:
  minapp_volume:

networks:
  backend_network:
    driver: bridge
```
