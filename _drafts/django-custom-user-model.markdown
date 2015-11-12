---
layout: post
title: "Custom User Model in Django"
categories: django python
---
Django comes with a complete authentication system [`django.contrib.auth`][django-auth] that includes a [`User`][django-user] class with build in permissions support, `admin` support, and views and forms for most basic tasks like user sign-up, login/logout, password reset, etc. Basically, it provides all the functionalities you would want to have for a user module. That is, if you are fine without wanting to customize the `User` class yourself. The usual way of customizing the User class is to use a custom model that has a [one to one][django-one-to-one] relationship to the `User` class. However, most recent development in the internet demands us to drop the concept of `username` altogether. Instead, simply use the email address, or in the case of most Chinese website, use the cell phone number as their unique identifier.

[django-auth]: https://docs.djangoproject.com/en/1.8/topics/auth/
[django-user]: https://docs.djangoproject.com/en/1.8/ref/contrib/auth/#user
[django-one-to-one]: https://docs.djangoproject.com/en/1.8/ref/models/fields/#ref-onetoone
