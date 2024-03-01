---
layout: post
title: "How to Reverse Proxy Jupyter Notebook on a Subpath Using Nginx"
categories: linux
date: 2024-02-29
---

Recently, I needed to proxy my Jupyter Notebook web interface from `host.domain:8888` to a subpath directly from the HTTP 80 port `host.domain/jupyter/`. Unfortunately, the fact that Jupyter uses WebSocket for the communication between its kernel and the web application, makes setting up correct proxy in Nginx slightly more complicated. After a few searches and experimentation, I have reached a working solution, which I am documenting below.

#### In `jupyter_notebook_config.py` file
In order for the Jupyter server to serve files correctly, we need to set its base URL in `$HOME/.jupyter/jupyter_notebook_config.py`. Simply set or add the following line
``` python
c.ServerApp.base_url = '/jupyter/'
```

#### In `http` section of `nginx.conf` file
We need to create a variable based on the `$http_upgrade` value
``` apacheconf
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}
```

#### In `server` section of `nginx.conf` file
We need to define a subpath and use the variable we just defined
``` apacheconf
location /jupyter/ {
        proxy_pass http://localhost:8888;
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
	proxy_set_header Connection $connection_upgrade;
	proxy_set_header Upgrade $http_upgrade;
}
```