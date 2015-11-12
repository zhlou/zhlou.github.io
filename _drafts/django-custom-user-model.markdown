---
layout: post
title: "Custom User Model in Django"
categories: django python
---
Django comes with a complete authentication system [`django.contrib.auth`][django-auth] that includes a ['User'][django-user] class with build in permissions support, `admin` support, and views and forms for the most basic tasks like user sign-up, login/logout, password reset, etc. Basically, it provides all the functionalities you would want to have for a user module. That is, if you are fine without wanting to customize the `User` class yourself.

[django-auth]: https://docs.djangoproject.com/en/1.8/topics/auth/
[django-user]: https://docs.djangoproject.com/en/1.8/ref/contrib/auth/#user
