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


We are we going to do:

- We will install Django Rest Framework (DRF) and Djoser packages.
- We will setup settings and urls.
- We will manually test our REST API with DRF browsable API.

## Install DRF and Djoser






![](login_diagram.png){:.image-50width}

<!--[![Create code repository in GitHub for your project](new_repository.png){:.image-border}](new_repository.png)-->