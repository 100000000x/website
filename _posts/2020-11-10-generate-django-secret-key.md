---
title: How to generate Django Secret Key?
categories:
    - Django
author: Piotr Płoński
date: 2020-11-10
type: blog
---


Have you ever pushed to the repository a Django project with `SECRET_KEY`? Ups, it happens to me very often. Don't worry. This can be easily fixed.

The [`SECRET_KEY`](https://docs.djangoproject.com/en/3.1/ref/settings/#std:setting-SECRET_KEY) is used in Django for [cryptographic signing](https://docs.djangoproject.com/en/3.1/topics/signing/). It is used to generate tokens and hashes. If somebody will have your `SECRET_KEY` he can recreate your tokens.

Storing `SECRET_KEY` in the repository code is not secure. It should be removed from the code and loaded from environment variables (or some configuration).  You can use for example [python-decouple](https://github.com/henriquebastos/python-decouple) to separate configuration variables from the code.

Once `SECRET_KEY` was committed into the code repository, it needs to be generated again. Luckily, we can use Django `get_random_secret_key()` function to generate new `SECRET_KEY`.

```py
from django.core.management.utils import get_random_secret_key
# print new random secret key
print(get_random_secret_key())
```

This code can be run in the terminal as a command:

```bash
python -c 'from django.core.management.utils import get_random_secret_key; \
            print(get_random_secret_key())'
```

If you have new `SECRET_KEY` then you can use `python-decouple`.

```py
# in your settings.py file 

from decouple import config

SECRET_KEY = config("SECRET_KEY")

```

The `SECRET_KEY` can be then set as environment variable or can be saved in `.env` file (which is not tracked in the code repository).