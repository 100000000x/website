---
title: Starting SaaS with Django and React
categories: 
    - Django
    - React
    - Boilerplate
    - Frontend
    - Backend
author: Piotr Płoński
date: 2020-10-23
type: blog
---

Software-as-a-Service (SaaS) is a model of software delivery in which users pay a subscription to get access to centrally hosted software. Nowadays, you can easily build your own SaaS, but you need to have programming skills. There are a lot of open-source web frameworks that allow you to quickly create a web service. You can serve SaaS on rented hardware in the cloud. You can start your own SaaS at 10-50$ per month of total costs (but you need to have programming skills!). 

The main costs:
- You will need to buy a domain for your website, which costs about 12$ per year. 
- You will need to have a server machine. You can rent it from cloud providers with prices starting at 5$/month (sometimes even lower). Yes, you can run the whole app on a single machine. 

There is even a community of people building online businesses, called [Indie Hackers](https://www.indiehackers.com). You can get there a lot of help. You can check there for examples of SaaS websites that are self-funded: [link to the list](https://www.indiehackers.com/products?businessModel=subscriptions&funding=self&revenueVerification=stripe&sorting=highest-revenue).

In this series of posts, I would like to show you how to start your SaaS with Django and React from scratch. By the end of the tutorial, you will have Django + React boilerplate (a template code that can be reused to create a new SaaS). This boilerplate will be much different from existing boilerplates:
- You will write it from scratch. You will be familiar with every part of the boilerplate code.
- It will have only parts that are needed for you. If you don't need a feature you won't implement it, simple.
- The boilerplate code will be open-source on the MIT license. You may reuse it as many times as you want for free.
- A complete guide on how to deploy the SaaS boilerplate to a single server machine (and how to update the code) will be available. 

### By the end of this post you shoud ...

- You should prepare local development environment (install required libraries). 
- You should setup the text editor or Integrated Development Environment.
- You should set a code repository for the project.
- You will initialize Django and React projects.

### What will be in the next post?

- You will create user autentication in Django (with Django Rest Framework and Djoser package) in the [next post](/blog/token-based-authentication-django-rest-framework-djoser). 

![](starting_saas_with_django_react.jpg){:.image-border}

## Prepare your development machine

You need to setup your machine (laptop or desktop) for development. You will need to have 3 things:

- Python available on your machine (needed for Django).
- Node.js needed for React.
- Git for code repository.

I'm using Ubuntu (16.04) for development. You can use any other operating system (Windows, MacOS). I would recommend Ubuntu 18.04 or [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) on Windows OS.

### Text Editor

I'm using [Visual Code](https://code.visualstudio.com/) for text editing. It is free and open-source Integrated Development Environment (IDE). You can use other IDE/editors, for example PyCharm.

## Initialize the code repository

I'm using [GitHub](https://github.com) for code repository. It is free for public and private repositories. The alternative to GitHub can be: [BitBucket](https://bitbucket.org/) or [GitLab](https://gitlab.com) (both with free plans).

Let's go into [https://github.com/new](https://github.com/new) and start a new repository! 

[![Create code repository in GitHub for your project](create_repository.png){:.image-border}](create_repository.png)

You can see that I've selected the `Public` repository on the MIT license. The code I'm writing will be open to anyone on the internet. If you want to hide your code from others you should select `Private` here and don't select the license. 

The next important thing is `.gitignore` file. Notice that I'm adding `Python` template. It is important to select it because it will prevent `git` from tracking unimportant or unsafe files (like `.env` files). What about the `JavaScript` files? There will be an additional `.gitignore` in the frontend code automatically generated when we start React application.

My repository is available at [https://github.com/saasitive/django-react-boilerplate](https://github.com/saasitive/django-react-boilerplate). When you go to the repository you should see repository like in the image below.

[![Create code repository in GitHub for your project](new_repository.png){:.image-border}](new_repository.png)


## Create Django project

Before starting Django project let's get the empty repository from the Github:

```bash
git clone https://github.com/saasitive/django-react-boilerplate.git
```

Now, please navigate to this directory and create `backend` directory.

```bash
cd django-react-boilerplate
mkdir backend
```

Your project directory should look like:
```bash
├── backend
├── LICENSE
└── README.md
```

Let's create virtual environment in the `backend` directory.

```bash
cd backend
virtualenv venv
# activate virtual env
source venv/bin/activate
```

We are using virtual environment to not mess with packages from other projects. Let's check the python version:

```bash
python --version

Python 3.6.11
```

We will install Django package:

```bash
pip install django
```

Please take a look at Django version. We will need this to create `requirements.txt` file. Please add `requirements.txt` in `backend` directory with your Django version. In my case it was:

```bash
# content of backend/requirements.txt
django==3.1.2
```

We are now ready to initialize the Django project:

```bash
# run in backend dir
django-admin startproject server
```

You can run initialized server with command:

```bash
# navigate to server dir
cd server 
# run development server
python manage.py runserver
```

(Note: You can also run development server with `./manage.py runserver` command.)

Please enter [`127.0.0.1:8000`](http://127.0.0.1:8000) in your favourite browser and you should see a default Django welcome site.

[![Default Django welcome site](django_start_project.png){:.image-border}](django_start_project.png)

Congratulations you have successfully initialized the backend! We are in the half-way of this post, let's initialize the frontend project.

To stop the Django development server use keyboard: `CONTROL+C`.

## Create React App

Please navigate to the project root directory (where the `backend` directory is) and run commands:

```bash
npx create-react-app frontend
cd frontend
npm start
```

The above commands should initialize the React frontend code and run the development server at address: [`http://localhost:3000/`](http://localhost:3000/). You should see in the browser a spinning atom:

[![Default React welcome site](react_start_project.png){:.image-border}](react_start_project.png)

Congratulations you have successfully initialized the frontend! :tada:

#### Ports

Please notice that default Django port for development is `8000` and for React application it is `3000`.

### Commit code to the repository

Your project structure should look like this:

```bash
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

```bash
git add frontend
git add backend
```

To see what files will be added run `git status`:
```bash
> git status

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

We need to commit and push changes to the repository server:

```bash
git commit -am "initalize backend and frontend"
git push
```

You can check the repository in the browser (in my case it will be [https://github.com/saasitive/django-react-boilerplate](https://github.com/saasitive/django-react-boilerplate)), all changes should be there.

---

## Summary

- The hardest part is to start and we already started!  
- We have backend and frontend applications initialized. We are able to run development servers for both apps.
- The code is manages in the code repository.

## What's next

We will write authentication for backend in the next post. We will use Token-based authentication from Django Rest Framework and Djoser package for ready views. Our authentication backend will be able to:
- create (signup) a new user,
- login and logout the user,
- return user information.

Next article: [Token Based Authenitcation with Django Rest Framework and Djoser](/blog/token-based-authentication-django-rest-framework-djoser)