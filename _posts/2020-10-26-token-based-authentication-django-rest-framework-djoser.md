---
title: Token Based Authentication with Django Rest Framework and Djoser
categories: 
 - Django
 - DRF
 - Token based authentication
 - Djoser
 - Backend
author: Piotr Płoński
date: 2020-10-26
type: blog
---

In this post, we will add token-based authentication with Django Rest Framework and Djoser. The [Django Rest Framework](https://www.django-rest-framework.org/) is a package for faster building REST API with Django. The [Djoser](https://djoser.readthedocs.io) provides basic views to handle authentication actions such as create user, login, logout.

We are going to use a code from previous [post](/blog/django-react-boilerplate-saas) (it has tag [v1](https://github.com/saasitive/django-react-boilerplate/tree/v1)). We will write only backend code in this post. You can find frontend code in next post: [React Token Based Authentication to Django Backend](/blog/react-token-based-authentication-django).

What are we going to do in this post:

- We will install Django Rest Framework (DRF) and Djoser packages.
- We will setup settings and urls. 
- User will be able to signup with username and password. User will be able to login, get her profile and logout.
- We will manually test our REST API with DRF browsable API.

What are we **not** going to do in this post:
- We are not going to provide example how to do user activation and password reset. This will require email verification and sending activation/reset links with email. This topic will be covered in the future post (it will be available soon). Would you like to be notified when it will be ready? Please fill this [form](https://forms.gle/rgAG9gkhUEH2wUVt5).


## Token-based vs JWT authentication

There are many ways to implement authentication in DRF ([docs](https://www.django-rest-framework.org/api-guide/authentication/)). The popular ones are: 
- Token-based Authentication ([docs](https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication)),
- JSON Web Token (JWT) ([docs](https://www.django-rest-framework.org/api-guide/authentication/#json-web-token-authentication)).

What is the difference between them?

- Token-based authentication requires database look up on every request to check if token is valid.
- JWT is using cryptography to validate the token - no database queries.
- Token-based authentication is using the same token for all sessions.
- JWT is using different token for each session.
- Token-based doesn't have a timestamp for expiration. 
- JWT tokens expire after selected time period and needs to be refreshed.
- For Token-based authentication you can force user to logout by changing the token in the database.

Both authentication methods have pros and cons. In this post, I will use Token-based authentication. It doesn't need to be periodically refreshed and because of this, it will be much easier to use it in the frontend. If you would like to see a JWT post, please let me know by filling this [form](https://forms.gle/rgAG9gkhUEH2wUVt5).


## Install DRF and Djoser

Before install please go to `backend` directory and activate the environment:

```bash
# activate environment
cd backend
source /venv/bin/activate
```

Let's install [DRF](https://www.django-rest-framework.org/#installation), please notice that additional packages are also installed: `markdown` and `django-filter` (they will be needed for browsable API and filtering support):

```bash
pip install djangorestframework==3.11.2
pip install markdown      
pip install django-filter  
```

The [Djoser](https://djoser.readthedocs.io/en/latest/getting_started.html#installation) package installation:

```bash
pip install djoser
```

Let's update the `backend/requirements.txt` file:

```python
# requirements.txt file
djoser==2.0.5
djangorestframework==3.11.2 # https://github.com/sunscrapers/djoser/issues/541
markdown==3.3 
django-filter==2.4.0
```

**Notice:** We install DRF in version `3.11.2` because compatibility issue with Djoser (see github [issue](https://github.com/sunscrapers/djoser/issues/541)).

## Configure Django settings

Please update `backend/server/server/settings.py` file with needed configuration:

```python
# backend/server/server/settings.py file

# ... please find INSTALLED_APP list and update it

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 
    'rest_framework',
    'rest_framework.authtoken',
    'djoser'
]

#configure DRF
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ]
}

# configure Djoser
DJOSER = {
    "USER_ID_FIELD": "username"
}

# ...

```

## Add Authentication Endpoints to Django

We need to add URL endpoints to our backend. The URLs will be provided by Djoser package. There is a list of available endpoints in Djoser [documentation](https://djoser.readthedocs.io/en/latest/getting_started.html#available-endpoints).

In this post, we will use following endpoints:
- `/users/` - to signup a new user,
- `/users/me/` - to get user information,
- `/token/login/` - to get token
- `/token/logout/` - to logout

Endpoints for account activation and password rest will be used in the future post.

We can add endpoints directly into `backend/server/server/urls.py` file. However, in this series of post we would like to build a complete boilerplate for creating a SaaS, for this let's add a Django application `accounts`. It will be needed in the future, for example for account actication or payments. For now, we will just fill `urls.py`.

To add new application in Django run following:

```bash
# please run in backend/server directory
django-admin startapp accounts
mkdir apps
mv accounts apps
```

I made additional directory `apps` to keep all applications in it. It will help to keep the whole project well organized.

Please add a new file `urls.py` in `backend/server/apps/accounts` with following content:

```python
# backend/server/apps/accounts file
from django.conf.urls import url, include

accounts_urlpatterns = [
    url(r'^api/v1/', include('djoser.urls')),
    url(r'^api/v1/', include('djoser.urls.authtoken')),
]
```

The above file adds Djoser's endpoints to our application. Please notice that I've added `api/v1/` in the url definition. It will be added for all endpoints. Maybe you will need it to track API versions in the future. We need to update main urls at `backend/server/server/urls.py` to have endpoints available in the web server:

```python
from django.contrib import admin
from django.urls import path

from apps.accounts.urls import accounts_urlpatterns

urlpatterns = [
    path('admin/', admin.site.urls),
]

urlpatterns += accounts_urlpatterns # add URLs for authentication
```


```bash
.
├── apps
│   └── accounts
│       ├── admin.py
│       ├── apps.py
│       ├── __init__.py
│       ├── migrations
│       │   └── __init__.py
│       ├── models.py
│       ├── tests.py
│       ├── urls.py
│       └── views.py
├── db.sqlite3
├── manage.py
└── server
    ├── asgi.py
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py

```


To have new app available in Django we need to add it in `backend/server/server/settings.py`:

```python

# ...

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 
    'rest_framework',
    'rest_framework.authtoken',
    'djoser',
    # 
    'apps.accounts' # add new application
]

```

OK, we are ready to check our endpoints!

Please apply database migrations:

```bash
# please run in backend/server directory
./manage.py migrate
```

**Note:** We don't need to run `./manage.py makemigrations` because we do not add any new models. But we will use models from Django and DRF, that's why we need to run `migrate`.

## Test Authentication REST Endpoints

For checking REST API endpoints I will use browsable DRF API, which is amazing (in my opinion)!

Let's start the server:

```bash
# please run in backend/server directory
./manage runserver
```

Please go to [`http://127.0.0.1:8000`](http://127.0.0.1:8000) and you should see:

[![Django page not found error](django_page_not_found.png){:.image-border}](django_page_not_found.png)

It is expected because we don't have any endpoint at `/`. Please go to [`http://127.0.0.1:8000/api/v1/users`](http://127.0.0.1:8000/v1/users):

[![Django page not found error](django_create_user.png){:.image-border}](django_create_user.png)

Please fill the form at the bottom of the page and click `POST`. This will create a new user. The email field is not required, you can leave it blank.

Please change the url to [`http://127.0.0.1:8000/api/v1/token/login`](http://127.0.0.1:8000/v1/token/login):

[![DRF login](drf_login.png){:.image-border}](drf_login.png)

Please fill the form at the bottom and click `POST`. You should see a screen like below:

[![DRF token](drf_token.png){:.image-border}](drf_token.png)

The endpoint returned the token:

```python
{
    "auth_token": "dd7cfbff8525727b267411c692d08ee34478f2af"
}
```

We will need to include this token in each request that requires authentication. Please try to get the user information with [`http://127.0.0.1:8000/api/v1/users/me/`](http://127.0.0.1:8000/api/v1/users/me/) you should see:

[![DRF Authentication credentials not provided](drf_authentication_credentials_not_provided.png){:.image-border}](drf_authentication_credentials_not_provided.png)

You see the error `HTTP 401 Unauthorized` with message that authentication credentials are not provided.

The browsable DRF API doesn't support authorization with token, so there are 2 ways to enable it:
- add session based authentication for testing (I don't like it),
- add free browser plugin to inject token in request header (that's my option).

I'm using [ModHeader](https://bewisse.com/modheader/) plugin. It is availble for many browsers (Chrome, Firefox, Opera, Edge).

[![Set token in ModHeader](modheader_token.png){:.image-border}](modheader_token.png)

**Notice:** You need to set a token in the form `Token dd7cfbff8525727b267411c692d08ee34478f2af` - there is no colon here!

After setting the token in the header please just refresh the website.

[![DRF login](drf_authentication_ok.png){:.image-border}](drf_authentication_ok.png)

The endpoint returns the JSON with user information:

```python
{
    "email": "",
    "username": "piotr"
}
```

---

## Summary

- In this post, we created authentication backend.
- User is able to create account with username and password.
- There is login and logout enpoints available. And enpoint with user information.
- The user activation and password reset functionality will be presented in the future post.

**Note:** In this post, we do not cover the CORS headers problem. It will be covered in the next post.

## What's next?

We will write a frontend code for authentication. There will be signup and login forms. There will be a page that will be available only to authenticated users.

Next article: [React Token Based Authentication to Django Backend](/blog/react-token-based-authentication-django).
