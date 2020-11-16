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

The most exciting moment of web application development is deployment. Your app is going live. It can also be nerve-wracking. Unfortunately. There are many options, many variables, configurations. It is easy to miss something ... In this article, I will show you how to pack Django and React application into containers and deploy them with `docker-compose`. The presented approach can be reused on any Cloud Provider (AWS, DigitalOcean, Linode, GCP, Heroku) - you just need a Virtual Private Server (VPS).

In this article:
 - We will create two `docker-compose` configuration files. One for development (easier version) and one for production (with SSL certificate from [Let's encrypt](https://letsencrypt.org/)).
 - The React static files will be served by `nginx`.
 - The Django static files (from admin and DRF browsable API) will be served by `nginx`.
 - The `nginx` will be reverse-proxy to Django server.
 - In production `docker-compose` we will add [`certbot`](https://certbot.eff.org/) to renew the certificate. To issue a certificate we will use bash script. To issue the certificate you need to have a domain.

## Create Dockerfile for Django and Nginx

Let's start by adding a new directory `docker` in the main directory of the project. Your project structure should look like below:

```bash
.
├── backend
├── docker
├── frontend
├── LICENSE
└── README.md
```

In `docker` directory please add `backend` directory with two files:

- `docker/backend/Dockerfile` - it will define how to build Django container,
- `docker/backend/wsgi-entrypoint.sh` - it will be entrypoint for container. It will apply migrations, collect static files and run WSGI server with [`gunicorn`](https://gunicorn.org/). 


### Django Dockerfile

The `Dockerfile` to build Django container:

```docker
# docker/backend/Dockerfile

FROM python:3.8.3-alpine

WORKDIR /app
ADD ./backend/requirements.txt /app/backend/


RUN pip install --upgrade pip
RUN pip install gunicorn
RUN pip install -r backend/requirements.txt

ADD ./docker /app/docker
ADD ./backend /app/backend
```

In this `Dockerfile` the `python:3.8.3-alpine` is used. Let's decode this image name and tag. It containes python in version `3.8.3`. It is running Linux distribution [`Alpine`](https://alpinelinux.org/) which is lightweight. The purpose of using lightweight Linux is to have small image which will result in faster builds. You can of course use different base image. For purpose of this tutorial the `python:3.8.3-alpine` is sufficient.

We create `/app` directory in the container and copy `/backend/requirements.txt`. Then, `gunicorn` and all needed packages are installed. The backend source code is copied at the end of the container build. It is on purpose. It makes building faster. When you change something in the code (without changing `requirements.txt`), only the last line of `Dockerfile` will be executed and the rest will be read from cache (of course if available).

The `Dockerfile` is not executing any command, we will use for this purpose `wsgi-entrypoint.sh`.

### Docker entrypoint

Please add a new file `docker/backend/wsgi-entrypoint.sh` and add execute rights to the file:

```bash
chmod +x docker/backend/wsgi-entrypoint.sh
```

The `docker/backend/wsgi-entrypoint.sh` file:

```bash
#!/bin/sh

until cd /app/backend/server
do
    echo "Waiting for server volume..."
done

until ./manage.py migrate
do
    echo "Waiting for db to be ready..."
    sleep 2
done

./manage.py collectstatic --noinput

gunicorn server.wsgi --bind 0.0.0.0:8000 --workers 4 --threads 4

#####################################################################################
# Options to DEBUG Django server
# Optional commands to replace abouve gunicorn command

# Option 1:
# run gunicorn with debug log level
# gunicorn server.wsgi --bind 0.0.0.0:8000 --workers 1 --threads 1 --log-level debug

# Option 2:
# run development server
# DEBUG=True ./manage.py runserver 0.0.0.0:8000
```

The above script is waiting till serever volume and database is ready. It runs migration on database and collect static files. It runs `gunicorn` server with IP address `0.0.0.0` and (docker default IP) and port `8000`.

The `gunicorn` is running `4` worker and `4` threads. You can set different numbers here. It depends on your machine. There are some heuristic rules that number of workers can be set as `4 * CPU cores` (see the [gunicorn docs](https://docs.gunicorn.org/en/stable/settings.html#workers)). If you are deploying application to machine with 4 CPU cores, then you can use `9` workers. (Remember, it is just heuristic not an exact rule). Then you can specify how many threads will be running in each worker. In our case, there are `4` threads ([gunicorn docs](https://docs.gunicorn.org/en/stable/settings.html#threads)). This means that we can process `4 * 4 = 16` concurrent requests. 

**1st Note:** We are using [SQLite](https://sqlite.org) for our development - it doesn't support concurrency. We will need to replace SQLite with advanced database engine like [PostgreSQL](https://www.postgresql.org/). I will replace database at the end of this tutorial to make final deployment. Although, I think it is good to deploy application already and practice deployment and code updates in production. That's why we are here.

**2nd Note:** One more thing, I added other options to run Django server in `wsgi-entrypoint.sh`. Why? You might need them for debugging. The first option add logging to the console with debug level and run `gunicorn` with one worker and single thread. The second option runs Django development server with `DEBUG=True`. It is setting environment variable and load it in `settings.py` with [`python-decouple`](https://github.com/henriquebastos/python-decouple) I will write about this in this post (don't worry!).

#### Django static files

Before going further, let's update `STATIC_URL` variable in `backend/server/server/settings.py`:

```py
# backend/server/server/settings.py

MEDIA_URL = '/media/'
STATIC_URL = '/django_static/' 
STATIC_ROOT = BASE_DIR / 'django_static'
```

We overwrite `STATIC_URL` with new value. All static files will be served with `/django_static/` in the URL. We add two new variables: 
- `MEDIA_URL` - it is URL for serving files uploaded to Django application, we will use it in future posts.
- `STATIC_ROOT` - it is a directory where static files from Django application will be stored after running `collectstatic` command. To this path will point `nginx`.

### Nginx Dockerfile and Configuration

Please add `nginx` directory in `docker` directory. There will be added three files:
- `docker/nginx/Dockerfile` - the instructions how to build `nginx` container image,
- `docker/nginx/development/default.conf` - the configuration file for `nginx` server used in development,
- `docker/nginx/production/default.conf` - the configuration file for production.

The `Dockerfile` for `nginx` container will use two stage build. 

- At first stage, the React static files will be created.
- At second stage, the static files will be copied and `nginx` server will be started. We copy React static files into `/usr/share/nginx/html` directory.


The `docker/nginx/Dockerfile` file:

```docker
# The first stage
# Build React static files
FROM node:13.12.0-alpine as build

WORKDIR /app/frontend
COPY ./frontend/package.json ./
COPY ./frontend/package-lock.json ./
RUN npm ci --silent
COPY ./frontend/ ./
RUN npm run build

# The second stage
# Copy React static files and start nginx
FROM nginx:stable-alpine
COPY --from=build /app/frontend/build /usr/share/nginx/html
CMD ["nginx", "-g", "daemon off;"]
```

The two stage build make the final image smaller.

Let's add `nginx` configuration for development `docker-compose` (without SSL):


```nginx
server {
    listen 80;
    server_name _;
    server_tokens off;
    client_max_body_size 20M;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    location /api {
        try_files $uri @proxy_api;
    }
    location /admin {
        try_files $uri @proxy_api;
    }

    location @proxy_api {
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Url-Scheme $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass   http://backend:8000;
    }

    location /django_static/ {
        autoindex on;
        alias /app/backend/server/django_static/;
    }
}
```

> What do you feel when you see `nginx` configuration file? 

No troubles! The `nginx` is here to help us. It will do what we will ask in configuration.

```nginx
server {
    listen 80;
    server_name _;
    server_tokens off;
    client_max_body_size 20M;
```

At the begining of the configuration we define a server (`server` keyword). It will listen on port `80` (it is `HTTP` port). There is no name assigned to the server (`server_name _;`). We switch off option to show server version on error pages (`server_token off;` [see docs](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_tokens)). The last setting is to set maximum request size ([see docs](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)) (`client_max_body_size 20M`). It means that requests larger than 20MB will result in error with HTTP 413 (Request Entity Too Large).

We have five `location` blocks ([location docs](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)) in the config which specify configuration for each URL (some kind of routing for requests).

The first `location /` defines what to do if the request comes from `/` (main domain or IP). The `index.html` file from `/usr/share/nginx/html` will be served as a response. It is our React static `index.html` file. 

```nginx
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }
```

The second and third locations `location /api` and `location /admin` redirect requests to `location @proxy_api`. It is a reverse-proxy:

```nginx
    location /api {
        try_files $uri @proxy_api;
    }
    location /admin {
        try_files $uri @proxy_api;
    }
```

What we have in fourth `location @proxy_api`? It is our Django application served with `gunicorn`. The nginx forwards all requests with `/api` and `/admin` in the URL to `http://backend:8000` which is the address of Django application in `docker-compose`:

```nginx
    location @proxy_api {
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass   http://backend:8000;
    }
```

The last `location /django_static/` serves static files from Django application (created with `collectstatic` command):

```nginx
    location /django_static/ {
        autoindex on;
        alias /app/backend/server/django_static/;
    }
```

That's all. We have configuration file for nginx server.

## Docker-compose for Django, Nginx and React

Our `docker-compose` will run two containers:
- `nginx` that will run `nginx` server on port 80 (default HTTP port). The `nginx` container has two volumes mounted, one with `django_static` and one with configuration file (from `docker/nginx/development` directory). The `nginx` container depends on `backend` container, which means that `nginx` will start after `backend` (Django app).
- `backend` will run container with Django application. It runs the `/app/docker/backend/wsgi-entrypoint.sh` that spin off the `gunicorn` server. The `backend` has `django_static` volume with static files. 

We will save our `docker-compose` in `docker-compose-dev.yml` file (in main project directory):

```yml
version: '2'

services:
    nginx: 
        restart: unless-stopped
        build:
            context: .
            dockerfile: ./docker/nginx/Dockerfile
        ports:
            - 80:80
        volumes:
            - static_volume:/app/backend/server/django_static
            - ./docker/nginx/development:/etc/nginx/conf.d
        depends_on: 
            - backend
    backend:
        restart: unless-stopped
        build:
            context: .
            dockerfile: ./docker/backend/Dockerfile
        volumes:
            
        entrypoint: /app/docker/backend/wsgi-entrypoint.sh
        volumes:
            - static_volume:/app/backend/server/django_static
        expose:
            - 8000        

volumes:
    static_volume: {}
```

There are few useful commands for dealing with `docker-compose`.

#### Build containers

```bash
docker-compose -f docker-compose-dev.yml build
```

Please notice that we are using `-f docker-compose-dev.yml` - it is to point custom `yml` file. By default `docker-compose` is reading `yml` configuration file from `docker-compose.yml`.

#### Run containers

```bash
docker-compose -f docker-compose-dev.yml up
```

#### Stop containers

```bash
docker-compose -f docker-compose-dev.yml down
```

You can also use `Ctrl+C` to stop containers.

#### Build and run containers

```bash
docker-compose -f docker-compose-dev.yml up --build
```

Please build container and run it with last command. You should see logs from `backend` and `nginx` in the terminal.

OK, let's go into browser and go to [`http://0.0.0.0`](http://0.0.0.0) address. You should see the `Home` view. Let's try to login:

[![Login error](login_error.png){:.image-border}](login_error.png)

After login attempt you should see toast with error: "Network Error" and some errors in the console.

We need to update our code. In the frontend we need to set in the `axios` to point to correct address of server. In the backend we need to add `0.0.0.0` to `ALLOWED_HOSTS`.

Update `frontend/src/App.js` code.

```jsx
// frontend/src/App.js
// remove this line
//axios.defaults.baseURL = "http://localhost:8000";

// new code
if (window.location.origin === "http://localhost:3000") {
  axios.defaults.baseURL = "http://127.0.0.1:8000";
} else {
  axios.defaults.baseURL = window.location.origin;
}

```

In the frontend we set a logic to point the correct address of the server:
 - in the case of development the `axios` will call server at `http://127.0.0.1:8000`,
 - in production the `axios` will call server at the same location origin as frontend. Frontend will call in fact `nginx` and it will redirect request to the Django.

In the `backend/server/server/settings.py` we need to update `ALLOWED_HOSTS` variable:

```py
# in backend/server/server/settings.py

# ...
ALLOWED_HOSTS = ['0.0.0.0']
# ...
```

After changes we need stop containers (`Ctrl+C`) and then rebuild docker containers and run them again.

```bash
docker-compose -f docker-compose-dev.yml up --build
```

Now, if you try to login you should be successful! What is more, you can use this `docker-compose` and run it on VPS in the cloud and make it available to others. BUT, it will be unsecure! It is using only HTTP. To make it secure we need to add certificate. For this we will create next `docker-compose` configuration file for production. We also add production `nginx` configuration file.

## Production docker-compose

Let's add directory `production` in `docker/nginx/` with `default.conf` file. It will be a configuration of `nginx` server to be used in production:

```nginx
server {
    listen 80;
    server_name boilerplate.saasitive.com;
    server_tokens off;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name boilerplate.saasitive.com;
    server_tokens off;

    ssl_certificate /etc/letsencrypt/live/boilerplate.saasitive.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/boilerplate.saasitive.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    client_max_body_size 20M;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    location /api {
        try_files $uri @proxy_api;
    }
    location /admin {
        try_files $uri @proxy_api;
    }

    location @proxy_api {
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Url-Scheme $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass   http://backend:8000;
    }

    location /django_static/ {
        autoindex on;
        alias /app/backend/server/django_static/;
    }
}
```

It is similar to development configuration. It has two `server` definitions. The first one:

```nginx
server {
    listen 80;
    server_name boilerplate.saasitive.com;
    server_tokens off;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}
```

The above definition listen on port `80` and redirects all requests to `HTTPS` (`return 301 https://$host$request_uri;`). In this part there is also a `server_name` filled to `boilerplate.saasitive.com`. I'm planning to run the application from this tutorial on `boilerplate.saasitive.com` domain. **You should use here your own domain.**. The `location /.well-known/acme-challenge/` is used by cerbot for issuing the certificate.

Let's look closer at second `server` in configuration:

```nginx
server {
    listen 443 ssl;
    server_name boilerplate.saasitive.com;
    server_tokens off;

    ssl_certificate /etc/letsencrypt/live/boilerplate.saasitive.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/boilerplate.saasitive.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    client_max_body_size 20M;

    #...
```

It is listening on 443 port which is default for `HTTPS`. The `server_name` is set to domain. There are added paths with certificate. You should rename them to your domain:

```nginx
    ssl_certificate /etc/letsencrypt/live/---your-domain.com---/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/---your-domain.com---/privkey.pem;
    
```

The rest of the configuration is very similar to development configuration, except that we need to set additional headers:

```nginx
    location @proxy_api {
        proxy_set_header X-Forwarded-Proto https; # additional 
        proxy_set_header X-Url-Scheme $scheme; # additional 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # additional
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass   http://backend:8000;
    }

```

Let's add production `docker-compose`. The `docker-compose.yml` file in main project directory:

```yml
version: '2'

services:
    nginx: 
        restart: unless-stopped
        build:
            context: .
            dockerfile: ./docker/nginx/Dockerfile
        ports:
            - 80:80
            - 443:443
        volumes:
            - static_volume:/app/backend/server/django_static
            - ./docker/nginx/production:/etc/nginx/conf.d
            - ./docker/nginx/certbot/conf:/etc/letsencrypt
            - ./docker/nginx/certbot/www:/var/www/certbot
        depends_on: 
            - backend
    certbot:
        image: certbot/certbot
        restart: unless-stopped
        volumes:
            - ./docker/nginx/certbot/conf:/etc/letsencrypt
            - ./docker/nginx/certbot/www:/var/www/certbot
        entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"       
    backend:
        restart: unless-stopped
        build:
            context: .
            dockerfile: ./docker/backend/Dockerfile
        entrypoint: /app/docker/backend/wsgi-entrypoint.sh
        volumes:
            - static_volume:/app/backend/server/django_static
        expose:
            - 8000        

volumes:
    static_volume: {}
```

The `nginx` server is listenning on two ports: 80 (`HTTP`) and 443 (`HTTPS`). There are added additional volumes in `nginx` with certificate. There is new container `certbot` that is responsible for certificate renewal. There are no changes in `backend` container.

OK, to be ready to run it we need to get the certificate from [`Let's encrypt`](https://letsencrypt.org/). I will use for it bash script `initletsencrypt.sh`. The script is from:
- article [Nginx and Let’s Encrypt with Docker in Less Than 5 Minutes](https://medium.com/@pentacent/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71),
- the Github repository with script: [link](https://github.com/wmnnd/nginx-certbot)

The `init-letsencrypt.sh` file:

```bash
#!/bin/bash

if ! [ -x "$(command -v docker-compose)" ]; then
  echo 'Error: docker-compose is not installed.' >&2
  exit 1
fi

domains=(boilerplate.saasitive.com www.boilerplate.saasitive.com)
rsa_key_size=4096
data_path="./docker/nginx/certbot"
email="" # Adding a valid address is strongly recommended
staging=1 # Set to 1 if you're testing your setup to avoid hitting request limits

if [ -d "$data_path" ]; then
  read -p "Existing data found for $domains. Continue and replace existing certificate? (y/N) " decision
  if [ "$decision" != "Y" ] && [ "$decision" != "y" ]; then
    exit
  fi
fi


if [ ! -e "$data_path/conf/options-ssl-nginx.conf" ] || [ ! -e "$data_path/conf/ssl-dhparams.pem" ]; then
  echo "### Downloading recommended TLS parameters ..."
  mkdir -p "$data_path/conf"
  curl -s https://raw.githubusercontent.com/certbot/certbot/master/certbot-nginx/certbot_nginx/_internal/tls_configs/options-ssl-nginx.conf > "$data_path/conf/options-ssl-nginx.conf"
  curl -s https://raw.githubusercontent.com/certbot/certbot/master/certbot/certbot/ssl-dhparams.pem > "$data_path/conf/ssl-dhparams.pem"
  echo
fi

echo "### Creating dummy certificate for $domains ..."
path="/etc/letsencrypt/live/$domains"
mkdir -p "$data_path/conf/live/$domains"
docker-compose run --rm --entrypoint "\
  openssl req -x509 -nodes -newkey rsa:1024 -days 1\
    -keyout '$path/privkey.pem' \
    -out '$path/fullchain.pem' \
    -subj '/CN=localhost'" certbot
echo


echo "### Starting nginx ..."
docker-compose up --force-recreate -d nginx
echo

echo "### Deleting dummy certificate for $domains ..."
docker-compose run --rm --entrypoint "\
  rm -Rf /etc/letsencrypt/live/$domains && \
  rm -Rf /etc/letsencrypt/archive/$domains && \
  rm -Rf /etc/letsencrypt/renewal/$domains.conf" certbot
echo


echo "### Requesting Let's Encrypt certificate for $domains ..."
#Join $domains to -d args
domain_args=""
for domain in "${domains[@]}"; do
  domain_args="$domain_args -d $domain"
done

# Select appropriate email arg
case "$email" in
  "") email_arg="--register-unsafely-without-email" ;;
  *) email_arg="--email $email" ;;
esac

# Enable staging mode if needed
if [ $staging != "0" ]; then staging_arg="--staging"; fi

docker-compose run --rm --entrypoint "\
  certbot certonly --webroot -w /var/www/certbot \
    $staging_arg \
    $email_arg \
    $domain_args \
    --rsa-key-size $rsa_key_size \
    --agree-tos \
    --force-renewal" certbot
echo

echo "### Reloading nginx ..."
docker-compose exec nginx nginx -s reload
```

You need to set two variables in the script:

- `domains=(boilerplate.saasitive.com www.boilerplate.saasitive.com)` you should enter **your domain** here,
- `staging=1` - is set to **test** the configuration first with **Let's encrypt staging environment**! It is important to not set `staging=0` before you are 100% sure that your configuration is correct. If you are sure, that all is set correctly, set `staging=0`. This is because there are limited number of retries to issue the certificate and you don't want to wait till they are reseted (one week). To read more about this, check [Let's encrypt rate limits docs](https://letsencrypt.org/docs/rate-limits/).



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
```bash
sudo docker-compose down
```

```
sudo tar -czvf certbot.tar.gz docker/nginx/certbot

```


rsync -avz -progress -e "ssh -i ~/Downloads/boilerplate.pem" ubuntu@ec2-100-26-177-134.compute-1.amazonaws.com:/home/ubuntu/app/certbot.tar.gz .certbot