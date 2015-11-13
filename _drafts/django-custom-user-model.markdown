---
layout: post
title: "Custom User Model in Django"
categories: django python
---
Django comes with a complete authentication system [`django.contrib.auth`][django-auth] that includes a [`User`][django-user] class with build in permissions support, `admin` support, and views and forms for most basic tasks like user sign-up, login/logout, password reset, etc. Basically, it provides all the functionalities you would want to have for a user module. That is, if you are fine without wanting to customize the `User` class yourself. The usual way of customizing the User class is to use a custom model that has a [one to one][django-one-to-one] relationship to the `User` class. However, most recent development in the internet demands us to drop the concept of `username` altogether. Instead, many new website simply use the email address, or in the case of most Chinese website, the cell phone number as their unique identifier. Such changes demands a new `User` class model. The good news is that Django provides a way to use a [custom User model][django-custom-user]. However, its documentation is a little hash for a beginner like me and there ain't many tutorials about this topic. So I decide to write this post (and perhaps a few following) to document my adventures.

[django-auth]: https://docs.djangoproject.com/en/1.8/topics/auth/
[django-user]: https://docs.djangoproject.com/en/1.8/ref/contrib/auth/#user
[django-one-to-one]: https://docs.djangoproject.com/en/1.8/ref/models/fields/#ref-onetoone
[django-custom-user]: https://docs.djangoproject.com/en/1.8/topics/auth/customizing/#substituting-a-custom-user-model
