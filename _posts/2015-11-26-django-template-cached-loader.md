---
layout: post
title: "How to Use Cached Template Loader in Django"
categories: django python deployment
date: 2015-11-26 13:53
---

Django's [deployment checklist][django-deployment-checklist] mentioned that using a cached template loader will improve the performance. However, without giving a solid example, the documentation only direct me to its [template loaders docs][django-template-loader]. So it's left to me to do some digging and experimenting.

The Django documentation describes the `cached.Loader` as a wrapper for other "real" loaders. Unlike the more common `filesystem.Loader` and `app_directories.Loader`, which one only needs to set `DIRS` and `APP_DIRS` options inside of `TEMPLATES` settings, `cached.Loader` requires a full `loaders` option complete with arguments. The example given by the documentation looks like this

{% highlight python %}
TEMPLATES = [{
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'DIRS': [os.path.join(BASE_DIR, 'templates')],
    'OPTIONS': {
        'loaders': [
            ('django.template.loaders.cached.Loader', [
                'django.template.loaders.filesystem.Loader',
                'django.template.loaders.app_directories.Loader',
            ]),
        ],
    },
}]
{% endhighlight %}

Glancing at this example, which inconspicuously missing the usual `'APP_DIRS': True` setting, one attempts to merge that with existing settings which usually ends up look like this

{% highlight python %}
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
            'loaders': [
                ('django.template.loaders.cached.Loader', [
                    'django.template.loaders.filesystem.Loader',
                    'django.template.loaders.app_directories.Loader',
                ]),
            ],
        },
    },
]
{% endhighlight %}

Unfortunately, this is not the case. This setting will hit you with an `ImproerlyConfigured` exception complains about

    app_dirs must not be set when loaders is defined.

when rendering. After some Googling and experimenting, it turns out the `'APP_DIRS'` setting went missing in the example for a reason. Once you specified `django.template.loaders.app_directories.Loader` as one of the loaders wrapped in the cached loader, you don't need (and should not) set `'APP_DIR': True` anymore. Removing that line gives you a happily configured Django. Now the correct `TEMPLATE` setting look like this

{% highlight python %}
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
            'loaders': [
                ('django.template.loaders.cached.Loader', [
                    'django.template.loaders.filesystem.Loader',
                    'django.template.loaders.app_directories.Loader',
                ]),
            ],
        },
    },
]
{% endhighlight %}

[django-deployment-checklist]: https://docs.djangoproject.com/en/1.8/howto/deployment/checklist/
[django-template-loader]: https://docs.djangoproject.com/en/1.8/ref/templates/api/#template-loaders
