---
title: Token Based Authentication with Django Rest Framework and Djoser
categories: 
 - Django
 - DRF
 - Token based authentication
 - Djoser
 - backend
author: Piotr Płoński
date: 2020-10-26
type: blog
---

### By the end of this post you shoud

- You should have prepared local development environment (install required libraries). 
- You should have setup the text editor or Integrated Development Environment.
- You should have a code repository set for the project.
- You will start a Django and React projects.

### What will be in the next post

- You will create autentication of users in Django (with Djoser package).

### Commit code to the repository

Your project structure should look like this:

```
.
├── backend
│   ├── server
│   └── venv
├── frontend
│   ├── node_modules
│   ├── package.json
│   ├── public
│   ├── README.md
│   ├── src
│   └── yarn.lock
├── LICENSE
└── README.md
```

There are two directories:
- `backend` for storing the Django server code.
- `frontend` for React user interface code.

Let's add them into `git`:

```
git add frontend
git add backend
```

To see what files will be added run:
```
git status

On branch main
Your branch is up-to-date with 'origin/main'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   backend/server/manage.py
	new file:   backend/server/server/__init__.py
	new file:   backend/server/server/asgi.py
	new file:   backend/server/server/settings.py
	new file:   backend/server/server/urls.py
	new file:   backend/server/server/wsgi.py
	new file:   frontend/.gitignore
	new file:   frontend/README.md
	new file:   frontend/package.json
	new file:   frontend/public/favicon.ico
	new file:   frontend/public/index.html
	new file:   frontend/public/logo192.png
	new file:   frontend/public/logo512.png
	new file:   frontend/public/manifest.json
	new file:   frontend/public/robots.txt
	new file:   frontend/src/App.css
	new file:   frontend/src/App.js
	new file:   frontend/src/App.test.js
	new file:   frontend/src/index.css
	new file:   frontend/src/index.js
	new file:   frontend/src/logo.svg
	new file:   frontend/src/reportWebVitals.js
	new file:   frontend/src/setupTests.js
	new file:   frontend/yarn.lock
```

We need to commit the changes and push to repository server:

```
git commit -am "initalize backend and frontend"
git push
```

You can check the repository in the browser (in my case it will be [https://github.com/saasitive/django-react-boilerplate](https://github.com/saasitive/django-react-boilerplate)), all changes should be available.

## Summary

- The hardest part is to start. We already started! :tada: 
- We have backend and frontend applications initilized. For both we are able to run development servers.
- The code is in the code repository.

## What's next

In the nest post we will write authentication for backend. We will use Token-based authentication from Django Rest Framework and Djoser package. The authnetication backend will be able to:
- create (signup) a new user,
- login and logout the user,
- return user information.