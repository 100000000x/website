---
title: Docker-Copose for Django and React with Nginx reverse-proxy and Let's encrypt certificate
categories:
    - Docker-Compose
    - Django
    - React
    - Nginx
    - Let's encrypt
author: Piotr Płoński
date: 2020-10-28
type: blog
---

In this post, we will create a docker-compose for our project. 
- We will use docker-compose to pack all parts. 
- We will build React application and server its static files with nginx. 
- We will use nginx to reverse-proxy API calls to Django server. 
- We will use a script to issue Let's encrypt certificate.
- We will add Let's encrypt certbot running in the docker-compose for certificate renewal. 

