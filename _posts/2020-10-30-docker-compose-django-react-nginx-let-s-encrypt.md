---
title: Docker-Compose for Django and React with Nginx reverse-proxy and Let's encrypt certificate
categories:
    - Docker-Compose
    - Django
    - React
    - Nginx
    - Let's encrypt
    - Boilerplate
author: Piotr Płoński
date: 2020-10-30
type: blog
---

Coming soon.

What will we do:
- We will use docker-compose to pack all parts. 
- We will build React application and serve static files with nginx. 
- We will use nginx as reverse-proxy to REST API calls to Django server. 
- We will use a script to issue Let's encrypt certificate.
- We will add Let's encrypt certbot running in the docker-compose for certificate renewal. 











```py
# backend/server/server/settings.py

# ...

DEBUG = False

ALLOWED_HOSTS = ["0.0.0.0"]

# ...

MEDIA_URL = '/media/'
STATIC_URL = '/django_static/' 
STATIC_ROOT = BASE_DIR / 'django_static'
```





```bash
chmod +x docker/backend/wsgi-entrypoint.sh
```



```bash
docker-compose -f docker-compose-dev.yml build
docker-compose -f docker-compose-dev.yml up
```
```bash
docker-compose -f docker-compose-dev.yml up --build
```