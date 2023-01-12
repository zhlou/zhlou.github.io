---
layout: post
title: "Linux Server Review: Debian 11 Bullseye"
categories: linux
date: 2023-01-05
---

## Forward
Recently, I needed to spin up a Linux server instance. Since I have been out of practice for a few years now, I went out to look for a few comprehensive reviews for various Linux distributions. Unfortunately, while that are abundance of Linux reviews, from the great [Distrowatch](https://distrowatch.com) to various blogs and Youtube channels, rarely do they focus on server related topics. Considering Linux as a server is what over 90% of people working on Linux actually do, and _Linux on desktop_&trade; is perpetually a year away, the lack of server oriented Linux review is really a sad really. Unsatisfied with this, I decided to write my own.

This will be the first out of hopefully a few posts to review distributions with a focus on server administration. This post will review Debian 11 Bullseye.

## Installation 
The installation is done using `netinst.iso` image via TUI with only SSH server and standard system utilities installed.

![Debian Install Software Selection](/assets/debian_install_software_selection.png)

### Root Account
During installation, the Debian installer will give you the option to set a password for `root` user or disable it entirely. 

![Debian Install Set Root Password](/assets/debian_install_user_passwords.png)

Setting a non-empty password will enable the `root` user login via password. This is a more traditional setup that many people are familiar with. If one choose to enter an empty password in the process, the `root` user will be disabled. Instead, the regular user to be created in the next step will be part of `sudo` group and can carry out sysadmin work via `sudo` commands. Worth noting that, as a quirk from the Debian installation, if the `root` user is enabled, `sudo` the program will _not_ be installed by default, and as a result, the regular user created will _not_ be in `sudo` group.

## Default Security 
Once installed, Debian 11 has reasonably good out-of-box security. Nmap scan show that only TCP port 22 (SSH) is open. One thing to note is that all other ports are rejected with TCP reset. Depending the school of thought you subscribe to, this might be either the exact right thing to do, or reducing the security by not using drop / no-response, and thus allowing port scanning to move quickly. I was able to scan all 65,535 ports in about 11 seconds. 
```
# Nmap 7.93 scan initiated Sat Dec 31 00:11:30 2022 as: nmap -p- -T4 -oN - 192.168.1.53
Nmap scan report for debian.lan (192.168.1.53)
Host is up (0.00054s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:D2:04:B6 (Oracle VirtualBox virtual NIC)

# Nmap done at Sat Dec 31 00:11:41 2022 -- 1 IP address (1 host up) scanned in 10.85 seconds
```
In addition, the default `sshd_config` is set to disallow password login for `root` account. This adds some more security or annoyance depending on your view.

## Network Services
For most developers, you want to hit the ground run in the shortest time possible. This usually means needing to spin up a web server and a database of sorts. Here I review how to install and configure Nginx and PostgreSQL in Debian 11.

### Serving Web Pages with Nginx
Debian 11's package repository currently contains Nginx version 1.18.0. This is a few version behind, but completely expected. When installed, it will automatically add `nginx.service` as part of `multi-user.target.wants`. It also come with a very modularized configuration directory.
```
# tree /etc/nginx/
/etc/nginx/
├── conf.d
├── fastcgi.conf
├── fastcgi_params
├── koi-utf
├── koi-win
├── mime.types
├── modules-available
├── modules-enabled
│   ├── 50-mod-http-geoip.conf -> /usr/share/nginx/modules-available/mod-http-geoip.conf
│   ├── 50-mod-http-image-filter.conf -> /usr/share/nginx/modules-available/mod-http-image-filter.conf
│   ├── 50-mod-http-xslt-filter.conf -> /usr/share/nginx/modules-available/mod-http-xslt-filter.conf
│   ├── 50-mod-mail.conf -> /usr/share/nginx/modules-available/mod-mail.conf
│   ├── 50-mod-stream.conf -> /usr/share/nginx/modules-available/mod-stream.conf
│   └── 70-mod-stream-geoip.conf -> /usr/share/nginx/modules-available/mod-stream-geoip.conf
├── nginx.conf
├── proxy_params
├── scgi_params
├── sites-available
│   └── default
├── sites-enabled
│   └── default -> /etc/nginx/sites-available/default
├── snippets
│   ├── fastcgi-php.conf
│   └── snakeoil.conf
├── uwsgi_params
└── win-utf
```
#### Serving Static Files
For static pages, one can put the files anywhere. Just make sure the user `www-data` have access to the directory and the files within, and update the `root` line in `/etc/nginx/sites-available/default` and it is good to go.

#### Serving Dynamic Web Applications
Unlike static pages, Debian doesn't include builtin niceties for serving a web app. Use an nginx -> gunicorn -> flask application as an example. To achieve that, you will need to manually create a systemd unit file for gunicorn in `/etc/systemd/system/`, and in nginx site configure file, point `proxy_pass` to the bind address or socket of gunicorn. 

### Database with PostgreSQL
Another common use for a Linux server is to run a relational database. Here I use PostgreSQL as an example for the review.

Debian 11 comes with PostgreSQL version 13, which is not up to date, but completely serviceable. In addition to the database and client tools, it also included a few Debian customized management tools such as `pg_lsclusters`, `pg_ctlcluster`, and `pg_createcluster` that help with cluster management. Upon installation, somewhat surprisingly, it automatically creates a database cluster called `main` with data directory at `/var/lib/postgresql/13/main`. This cluster is configured to start automatically, and allow localhost connection only by default. If this is what you need, you are good to go. Or you can further customize the configuration files under `/etc/postgresql/13/main/`.

Sometimes, a data directory under `/var` may not be the best option. It is straightforward to create new cluster with custom data directory, presumably in a separate mount point, using either the included Debian specific `pg_createcluster` script or PostgreSQL's builtin `initdb` command. In addition, the default cluster can be turned off by editing the configure file `/etc/postgresql/13/main/start.conf`.

## Summary
The Debian 11 offers a solid albeit slightly out-of-date Linux server distribution. Its packages includes many small quality-of-life improvements over their corresponding upstream releases. From the perspective of server applications, these choices are certainly the right tradeoff to make. This is definitely a great Linux distribution for servers.

