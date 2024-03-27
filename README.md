# docker-work

you have to create entrypoint.sh file to store command you want to run at first time running image 

#!/bin/bash
python manage.py migrate --no-input

exec "$@"

In Dockerfile \*In root directory of project 

FROM python:alpine3.19
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
WORKDIR /name_of_project
COPY requirements.txt /name_of_project/
RUN pip install --upgrade pip && pip install -r requirements.txt
COPY . .
EXPOSE 8000
COPY entrypoint.sh .
RUN chmod +x entrypoint.sh
ENTRYPOINT ["sh","entrypoint.sh"]

This is only for main application of django

In  docker-compose.yml file 

version: "3.8" //version of docker-compose
services:
    name_of_app:
        container_name: container_name
        build: location_of_Dockerfile
        command: python manage.py runserver 0.0.0.0:8000
        volumes: 
            - .:/name_of_app/
        ports: 
            - "8080:8000" //mapping port to container
        depends_on: 
            - db //this webapp is dependent on database

services of postgres database

    db:
        image: postgres:latest
        restart: always
        environment:
            POSTGRES_DB: blog_db //name_of_database you want
            POSTGRES_USER: root //user you want
            POSTGRES_PASSWORD: password //password you want
        volumes:
            - postgres_data:/var/lib/postgresql/data/
        ports:
            - "5432:5432" //mapping port to default port of the database (5432)

if you want to see your database graphically by using pgadmin4 

    pgadmin:
        image: dpage/pgadmin4
        restart: always
        environment:
            PGADMIN_DEFAULT_EMAIL: admin@pgadmin.org
            PGADMIN_DEFAULT_PASSWORD: admin
        ports:
            - "5050:80"
        depends_on:
            - db

volumes:
postgres_data:

volume to store your data from database when container is siced or stopped 

In main app in settings.py put this postgresql configuration to replace sqlite3's 

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'name_of_database',
        'USER': 'ya user wa creatinz',
        'PASSWORD': 'na password',
        'HOST': 'db niba ya service ya database wayise gutya', 
        'PORT': '5432',
    }
}

To run docker-compose kubakoresha podman  
pip install podman-compose

 so ku building code ziri muri docker-compose 
podman-compose up or podman-compose build


NGINX CONFIGRATIONS
====================

create directory call it nginx
=> inside create 2 file: Dockerfile and nginx.conf
in ngnix.conf
=============

upstream django_app {
    server name_of_app_service:8000;
}


server {
      listen 80;

     location /static/ {
        alias /location of your static directory/;
    }

    location / {
        proxy_pass http://django_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
in Dockerfile
============
FROM nginx:1.25

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d

Make change also in docker-compose.yml file by adding this service tooo
=======================================================================
  nginx:
    build:
      context: ./nginx/
      dockerfile: Dockerfile
    volumes:
      - static_volume:/static/
    ports:
      - "80:80"
    depends_on:
      - web

volumes:
  static_volume:


Then in django app in settings.py make sure to add STATIC_ROOT directory to collect static in
