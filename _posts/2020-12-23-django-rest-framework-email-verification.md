---
title: Django Rest Framework Email Verification
categories:
 - Django-Rest-Framework
 - Backend
 - Boilerplate
 - Email
author: Piotr Płoński
date: 2020-12-23
type: blog
---

Email verification is an important part of the SaaS application. We will contact the user by email in many cases: for a password reset, announcement of new features, or for sending the invoice. During registration, a user provides the email address. We need to check if the email belongs to the user, and that there are no typos/errors in it. This can be easily done by automatically sending the verification email with an activation link. Such a link contains the unique token assigned to the user. After opening the activation link in the web browser the request is sent to the web application (Django Rest Framework). The web server compares the token from the activation URL with the token stored in the database. If they are the same, the email address is verified.

In this tutorial you will learn:
 - how to set email as a mandatory field during registration,
 - how to set login with email and password,
 - how to send a verification email,
 - how to re-send a verification email,
 - write test cases for email verification flow,
 - the [Django Rest Framework](https://www.django-rest-framework.org/) and [Djoser](https://djoser.readthedocs.io/) packages will be used.

The user interface and production email setup will be descibed in the next posts. Fill the [form](https://forms.gle/rgAG9gkhUEH2wUVt5) to be notified about future posts.

This article is a part of series of articles on [how to build SaaS from scratch with Django and React](/django-react/boilerplate/). I will use the code from the previous article: [Docker-Compose for Django and React with Nginx reverse-proxy and Let's encrypt certificate](/tutorial/docker-compose-django-react-nginx-let-s-encrypt/).

---

## Email required in registration

The `email` field is available in `User` model in the Django, but it is not mandatory. To set `email` as required field and use it for login (`email` + `password`) we need to do below steps.

Please update the `models.py` file in `accounts` application. We will update the `email` field in the database to make it required.

```python
# /backend/server/apps/accounts/models.py
from django.contrib.auth.models import User

User._meta.get_field('email')._unique = True
User._meta.get_field('email').blank = False
User._meta.get_field('email').null = False
```

We need to update Django application `settings.py`:

```python
# backend/server/server/settings.py

# ...
# configure Djoser
DJOSER = {
    "USER_ID_FIELD": "username",
    "LOGIN_FIELD": "email"
}

# ...
```

With above changes the `email` is required during the registration and login. Remember to make migrations and apply them to the database (we changed the database models). 

## Activation email sending in the Django

The next step will be to configure the activation email sending.

We need to update `settings.py` file:

```python
# backend/server/server/settings.py

# ...
# configure Djoser
DJOSER = {
    "USER_ID_FIELD": "username",
    "LOGIN_FIELD": "email",
    "SEND_ACTIVATION_EMAIL": True,
    "ACTIVATION_URL": "activate/{uid}/{token}",
    'SERIALIZERS': {
        'token_create': 'apps.accounts.serializers.CustomTokenCreateSerializer',
    },
}

EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
SITE_NAME = "SaaSitive"

# ...
```

We added in the [Djoser](https://djoser.readthedocs.io/) configuration:
- enabled send activation email by `"SEND_ACTIVATION_EMAIL": True`;
- setup the activation email link URL `"ACTIVATION_URL": "activate/{uid}/{token}"`. Please notice that there are `uid` and `token` in the activation URL. Both are required to activate the account. The link is pointing to the URL address in our fronted (we will add React route for it). There is no such endpoint on the backend;
- defined the custom token create serializer `token_create` - it will be needed to allow the user to login even with unverified email.

We set the `EMAIL_BACKEND` as the Django `django.core.mail.backends.console.EmailBackend`. This backend is simply displaying all emails in the console. It is usefull only for development. (How to setup Django email backend for production will be described in the future posts.) The nice thing about the Django is switchable email backend. For the testing purposes Django will automatically switch email backend to `django.core.mail.backends.locmem.EmailBackend` and give us easy access to email outbox.

We also set the `SITE_NAME = "SaaSitive"`. The site name variable will be used in the activation email. You can set here your application name.

The example of activation email:

```email
Subject: Account activation on SaaSitive

Body:

You're receiving this email because you need to finish activation process on SaaSitive.

Please go to the following page to activate account:
http://testserver/activate/MQ/afc0m5-e3336fd5fa02874a588c9085d9bdf881

Thanks for using our site!

The SaaSitive team
```

You can overwrite the text in the activation email by setting different email template in the Djoser configuration.


## Custom create token serializer

The last thing is to add the custom create token serializer. Please add `serializers.py` file in the `backend/server/apps/accounts` directory. Our cusom token will simply overwrite the [`TokenCreateSerializer`](https://github.com/sunscrapers/djoser/blob/2862ea4c80d7e95ad246fe646173a7c82e2a9189/djoser/serializers.py#L101) from the Djoser package and will allow to obtain the `auth_token` for user without activated account.

```python
# backend/server/apps/accounts/serializers.py

from django.contrib.auth import authenticate, get_user_model
from djoser.conf import settings
from djoser.serializers import TokenCreateSerializer

User = get_user_model()

class CustomTokenCreateSerializer(TokenCreateSerializer):

    def validate(self, attrs):
        password = attrs.get("password")
        params = {settings.LOGIN_FIELD: attrs.get(settings.LOGIN_FIELD)}
        self.user = authenticate(
            request=self.context.get("request"), **params, password=password
        )
        if not self.user:
            self.user = User.objects.filter(**params).first()
            if self.user and not self.user.check_password(password):
                self.fail("invalid_credentials")
        # We changed only below line
        if self.user: # and self.user.is_active: 
            return attrs
        self.fail("invalid_credentials")
```

## Testing email varification flow

The email verfication flow in our application is descibed below. Let's write test cases to check them. We can also check this manually in the web browser by using DRF browsable API. Writing tests will take for some time but have many advantages:
- we have tests for user registration flow, so we are confident that all works well,
- when developing a new feature writing tests first might be faster than repeated manual tests.

![Email verification flow](email_verification_flow.png)

In our tests we will use following endpoints:

- endpoint to register the new user: `/api/v1/users/` with `POST` request,
- endpoint to verify the email: `/api/v1/users/activation` with `POST` request,
- endpoint to resend verification email: `/api/v1/users/resend_activation/` with `POST` request,
- endpoint to login: `/api/v1/token/login/` with `POST` request,
- endpoint to get user details: `/api/v1/users/` with `GET` request.


Let's define the empty test in `tests.py` file in the `backend/server/apps/accounts/` directory:

```python
# backend/server/apps/accounts/tests.py

from django.core import mail
from rest_framework import status
from rest_framework.test import APITestCase

class EmailVerificationTest(APITestCase):

    def test_register_with_email_verification(self):
        pass

```

We are using `APITestCase` from the Django Rest Framework (see the [docs](https://www.django-rest-framework.org/api-guide/testing/) for more details). To run this empty test let's run in the `backend/server` directory:

```bash
./manage.py test apps 
```

The expected output:

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
```

The first test will check registration with email activation before login.

```python
# backend/server/apps/accounts/tests.py

from django.core import mail
from rest_framework import status
from rest_framework.test import APITestCase

class EmailVerificationTest(APITestCase):

    # endpoints needed
    register_url = "/api/v1/users/"
    activate_url = "/api/v1/users/activation/"
    login_url = "/api/v1/token/login/"
    user_details_url = "/api/v1/users/"
    # user infofmation
    user_data = {
        "email": "test@example.com", 
        "username": "test_user", 
        "password": "verysecret"
    }
    login_data = {
        "email": "test@example.com", 
        "password": "verysecret"
    }

    def test_register_with_email_verification(self):
        # register the new user
        response = self.client.post(self.register_url, self.user_data, format="json")
        # expected response 
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        # expected one email to be send
        self.assertEqual(len(mail.outbox), 1)
        
        # parse email to get uid and token
        email_lines = mail.outbox[0].body.splitlines()
        # you can print email to check it
        # print(mail.outbox[0].subject)
        # print(mail.outbox[0].body)
        activation_link = [l for l in email_lines if "/activate/" in l][0]
        uid, token = activation_link.split("/")[-2:]
        
        # verify email
        data = {"uid": uid, "token": token}
        response = self.client.post(self.activate_url, data, format="json")
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)

        # login to get the authentication token
        response = self.client.post(self.login_url, self.login_data, format="json")
        self.assertTrue("auth_token" in response.json())
        token = response.json()["auth_token"]

        # set token in the header
        self.client.credentials(HTTP_AUTHORIZATION='Token ' + token)
        # get user details
        response = self.client.get(self.user_details_url, format="json")
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.json()), 1)
        self.assertEqual(response.json()[0]["email"], self.user_data["email"])
        self.assertEqual(response.json()[0]["username"], self.user_data["username"])

```

The steps of the first test:
- create new user,
- parse the verification email,
- activate the account by sending `token` and `uid` from verification email,
- login to get `auth_token`,
- get user details.

When you run test the expected output is below:

```bash
> ./manage.py test apps 

Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.330s

OK
Destroying test database for alias 'default'...
```

Let's add the next test with login without email verification and re-sending the verification email.

```python
# backend/server/apps/accounts/tests.py

# ... the rest of EmailVerificationTest code ...

    def test_register_resend_verification(self):
        # register the new user
        response = self.client.post(self.register_url, self.user_data, format="json")
        # expected response 
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        # expected one email to be send
        self.assertEqual(len(mail.outbox), 1)

        # login to get the authentication token
        response = self.client.post(self.login_url, self.login_data, format="json")
        self.assertTrue("auth_token" in response.json())
        token = response.json()["auth_token"]

        # set token in the header
        self.client.credentials(HTTP_AUTHORIZATION='Token ' + token)
        # try to get user details
        response = self.client.get(self.user_details_url, format="json")
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)

        # clear the auth_token in header
        self.client.credentials()
        # resend the verification email
        data = {"email": self.user_data["email"]}
        response = self.client.post(self.resend_verification_url, data, format="json")
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        
        # there should be two emails in the outbox
        self.assertEqual(len(mail.outbox), 2)

        # parse the last email
        email_lines = mail.outbox[1].body.splitlines()
        activation_link = [l for l in email_lines if "/activate/" in l][0]
        uid, token = activation_link.split("/")[-2:]
        
        # verify the email
        data = {"uid": uid, "token": token}
        response = self.client.post(self.activate_url, data, format="json")
        # email verified
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
```

The steps of second test:
- create new user,
- login the get `auth_token`,
- try to get user details and expect for `HTTP_401_UNAUTHORIZED`,
- resend the verification email,
- parse the last email to the `uid` and `token`,
- activate the account by veryfing the email.


I will additionally add two tests:
- to test the response for wrong email address during re-sending the verification,
- to test activation with wrong `uid` and `token`

```python
# backend/server/apps/accounts/tests.py

# ... the rest of EmailVerificationTest code ...

    def test_resend_verification_wrong_email(self):
        # register the new user
        response = self.client.post(self.register_url, self.user_data, format="json")
        # expected response 
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        
        # resend the verification email but with WRONG email
        data = {"email": self.user_data["email"]+"_this_email_is_wrong"}
        response = self.client.post(self.resend_verification_url, data, format="json")
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
        

    def test_activate_with_wrong_uid_token(self):
        # register the new user
        response = self.client.post(self.register_url, self.user_data, format="json")
        # expected response 
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        
        # verify the email with wrong data
        data = {"uid": "wrong-uid", "token": "wrong-token"}
        response = self.client.post(self.activate_url, data, format="json")
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
    
```

The expected output:

```bash
> ./manage.py test apps 

Creating test database for alias 'default'...
System check identified no issues (0 silenced).
....
----------------------------------------------------------------------
Ran 4 tests in 0.963s

OK
Destroying test database for alias 'default'...
```

### Commit your changes

Remember to commit all the code changes and add a new file:

```bash
git add apps/accounts/serializers.py 
git commit -am "DRF email verification"
git push
```

---

## Summary

- We wrote the backend for email verification with Django Rest Framework and Djoser.
- The new functionality is tested so we are confident that it works as expected.
- In the next post we will write the user interface with React for email verification functionality.

The code for this tutorial is available at [Github repository](https://github.com/saasitive/django-react-boilerplate).