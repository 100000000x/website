---
title: Starting SaaS with Django and React (Complete Tutorial)
categories: 
    - Django
    - React
    - Boilerplate
author: Piotr Płoński
date: 2020-10-23
type: blog
---


Software-as-a-Service (SaaS) is a model of software delivery in which users pay a subscription to get access to centrally hosted software. Nowadays, you can easily build your own SaaS, but you need to have programming skills. There are a lot of open-source web frameworks that allow you to quickly create a web service. You can serve SaaS on rented hardware in the cloud. You can start your own SaaS at 10-50$ per month of total costs (but you need to have programming skills!). 

The main costs:
- You will need to buy a domain for your website, which costs about 12$ per year. 
- You will need to have a server machine. You can rent it from cloud providers with prices starting at 5$/month (sometimes even lower). Yes, you can run the whole app on a single machine. 

There is even a community of people building online businesses, called [Indie Hackers](https://www.indiehackers.com). You can get there a lot of help. You can check there for examples of SaaS websites that are self-funded: [link to the list](https://www.indiehackers.com/products?businessModel=subscriptions&funding=self&revenueVerification=stripe&sorting=recently-updated).

In this series of posts, I would like to show you how to start your SaaS with Django and React from scratch. By the end of the tutorial, you will have Django + React boilerplate (a template code that can be reused to create new SaaS). This boilerplate will be much different from existing boilerplates:
- You will write it from scratch. You will be familiar with every part of the boilerplate code.
- It will have only parts that are needed for you. If you don't need a feature you won't implement it, simple.
- The boilerplate code will be open-source on the MIT license. You may reuse it as many times as you want for free.
- A complete guide on how to deploy the SaaS boilerplate to a single server machine (and how to update the code) will be available. 

## Prepare your development machine


