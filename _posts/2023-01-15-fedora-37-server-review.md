---
layout: post
title: "Linux Server Review: Fedora 37 Server Edition"
categories: linux
date: 2023-01-15
---

## Introduction
In the [previous post]({% post_url 2023-01-05-linux-server-review-debian-11 %}), we've reviewed Debian 11 from the perspective of using it as a server. Today, let's check out the other major distribution, Fedora 37 Server Edition.

## Installation
The installation is done using net install image. Fedora installation process really encourages you to use its graphical interface, and its TUI has very limited capability. Compare to Debian, it is a much heavier weight experience. The installation process needs at least 2GB of RAM to succeed. Even in TUI, the process will crash if has only 1GB of memory.

On the other hand, the Fedora's installer (the older but lesser known [Anaconda](https://anaconda-installer.readthedocs.io/en/latest/)) allows much more flexibility and customization than Debian's installation. 

![Fedora Install Root Account](/assets/fedora_install_custom_root_setting.png)

In particular, you can fully control the enable / disable of root account, whether allowing SSH as root using password, and regular user account's group setting all independently.

![Fedora Install User Account](/assets/fedora_install_user_creation.png)

## Default Service and Security
Fedora 37 Server Edition by defaults installs [Cockpit](https://cockpit-project.org/) that runs on port 9090. This is a fully featured server management web service (web console, in Red Hat's documentation) that make lots of system administration work straightforward. It also contain lots of additional capabilities like managing virtual machines or containers. But those features are not limited to Fedora, and beyond the scope of this review.

From security point of view, the default settings for Fedora Server is really tight. It has only port 22 (SSH) and 9090 (aforementioned Cockpit) open out of the box. More importantly, the majority ports are configured to drop the packet outright. That makes port scanning takes more than 10x longer than using reject.
```
# Nmap 7.93 scan initiated Mon Jan  9 22:35:45 2023 as: nmap -p- -T4 -oN - 192.168.1.55
Nmap scan report for fedora-server.lan (192.168.1.55)
Host is up (0.00059s latency).
Not shown: 65383 filtered tcp ports (no-response), 150 filtered tcp ports (admin-prohibited)
PORT     STATE SERVICE
22/tcp   open  ssh
9090/tcp open  zeus-admin
MAC Address: 0A:E5:16:D0:BC:B5 (Unknown)

# Nmap done at Mon Jan  9 22:38:09 2023 -- 1 IP address (1 host up) scanned in 144.60 seconds
```

In addition, the (in)famous SELinux is set to `enforcing` by default. This adds an additional layer of protection, and will potentially cause a ton of headaches down the line if you need to do anything unconventional by Red Hat's book.

## Network Services
Similar to the Debian review, I am going to document my experience in setting up Nginx and PostgreSQL in Fedora.

### Serving Web Pages with Nginx
Fedora 37's package repository have Nginx version 1.22.1, which is the current stable version. The installation includes a `nginx.service` unit file, but by default it is not enabled. To start the Nginx server, you will need to enable and start the service using `systemctl`, and also add HTTP service in `firewall-cmd`. The configuration files come with the package are very close to the upstream.
```
# tree /etc/nginx
/etc/nginx
├── conf.d
├── default.d
├── fastcgi.conf
├── fastcgi.conf.default
├── fastcgi_params
├── fastcgi_params.default
├── koi-utf
├── koi-win
├── mime.types
├── mime.types.default
├── nginx.conf
├── nginx.conf.default
├── scgi_params
├── scgi_params.default
├── uwsgi_params
├── uwsgi_params.default
└── win-utf
```

#### Serving Static Files
For static sites, you will simply need to change the `root` location in `/etc/nginx/nginx.conf` and make sure the user `nginx` has access to the files. This is very close to the upstream behavior. 

#### Serving Dynamic Web Applications
For a dynamic web app using nginx -> gunicorn -> flask, Fedora Server turns out to be much more difficult to configure than expected, mostly due to the harsh SELinux security settings.

Fedora Server's gunicorn installation doesn't include a systemd unit file. You can manually create one in `/etc/systemd/system/`, similar to many other Linux distributions. However, due to SELinux, the unit file has to be actually in that directory. A symlink to an unexpected location will cause SELinux to deny access due to lack of Type Enforcement context.

Similarly, a simple `proxy_pass` in nginx will also fail. After scrambling around SELinux audit log and `audit2why` command, we get the following message.
```
type=AVC msg=audit(1673673416.968:788): avc:  denied  { name_connect } for  pid=7058 comm="nginx" dest=8000 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:soundd_port_t:s0 tclass=tcp_socket permissive=0

	Was caused by:
	The boolean httpd_can_network_connect was set incorrectly.
	Description:
	Allow httpd to can network connect

	Allow access by executing:
	# setsebool -P httpd_can_network_connect 1
```
That means SELinux does not allow nginx to connect to upstream server by default. To allow network connect, the "correct" way is to run the following command
```
# setsebool -P httpd_can_network_connect 1
```

 There are lots of online discussion and debate already on whether to turn off SELinux. It is beyond the scope of this review to have deeper discussion about this topic.

### Database with PostgreSQL
Fedora 37 comes with PostgreSQL version 14.3, which is one major version behind as of this writing. The installation includes a few Fedora customized tools such as `postgresql-setup` and `postgresql-new-systemd-unit`. Once it is installed, it will put `postgresql.service` and `postgresql@.service` under `/usr/lib/systemd/system`, but no actual database will be created.

To create the default database with data directory at `/var/lib/pgsql/data`, simply run the included script and then enable the `postgresql` service:
```
# postgresql-setup --initdb
# systemctl enable postgresql
# systemctl start postgresql
```
To allow connection over TCP port (5432 by default), the port needs to be explicitly allowed in `firewalld`
```
# firewall-cmd --permanent --add-port=5432/tcp
```

For a database with custom data directory, things is a bit more complicated under Fedora 37 Server Edition. There are directions on how to achieve that under `/usr/share/doc/postgresql/README.rpm-dist`. 

You first need to create a new systemd unit file, and initialize the database in the special directory 
```
# postgresql-new-systemd-unit --unit postgresql@pgdata --datadir /data/pgdata
# postgresql-setup --initdb --unit postgresql@pgdata --port 5433
```
Once the data directory is created, to make SELinux happy, you will need to set context and enable NIS.
```
# semanage fcontext -a -t postgresql_db_t "/data/pgdata(/.*)?"
# setsebool -P nis_enabled 1
```
Finally, for allowing network connection, in addition to changes in typical `postgresql.conf` and `pg_hba.conf`, you will also need to open the TCP port.

## Summary
Fedora 37 Server Edition is a very solid and secure distribution, to the point it is making system administration annoying with SELinux enabled. Most packages are quite up-to-date, but the additional scripts included are not as useful as Debian ones. This is a great Linux distribution for those who are already experienced in Linux. 