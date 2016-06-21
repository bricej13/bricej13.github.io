---
layout: post
title: How to install Laverna in an Alpine LXC container
date: '2016-06-08 15:21:37'
tags:
- alpine-linux
- lxc
- lxc-homelab
- laverna
---

This guide will get you [Laverna](https://laverna.cc/index.html) running inside a super tiny and secure [Alpine Linux](http://alpinelinux.org/) container.

### Prerequisites
- A server with [LXC installed](https://wiki.debian.org/LXC)
- A basic Alpine LXC container setup - [Guide](https://www.flockport.com/new-micro-containers-based-on-alpine-linux/)

### Setup Laverna
#### Install dependencies
```bash
apk add openssl unzip
```

#### Download Laverna files
```bash
mkdir -p /var/www/
cd /var/www/
wget https://github.com/Laverna/static-laverna/archive/gh-pages.zip -O laverna.zip
unzip -uo laverna.zip
```

#### Install Nginx
We need a web server to serve Laverna. We're going to use nginx.
```bash
apk add nginx
```

We need to update nginx to point at the directory that houses Laverna. This only takes one change in `/etc/nginx/nginx.conf`. Change the `root` value under `server` to `/var/www/laverna`

```diff
- root http
+ root /var/www/laverna
```


#### Start your new site
All we have left is to start nginx, and set it to start at boot.

```bash
rc-update add nginx
rc-service start nginx
``` 
You can now visit your site at `http://[ip address]`

### Clean Up
In an effort to keep our container as small as possible, we should clean up after ourselves.
```bash
rm /var/www/laverna.zip
apk del unzip
```

### Start Laverna container at system boot
We've setup our Laverna service to start up when the container boots, but we also need to set up our Laverna container to start when the host boots.

From the host machine, edit `/var/lib/lxc/laverna/config` to add the following line:
```bash
lxc.start.auto = 1
```

To test to make sure your container will boot on startup, shut down the container and then run the following command:
```bash
lxc-autostart
```
This command starts all containers that are marked to boot on startup. After running this command you should be able to navigate to Laverna from your web browser.


### Now what?
Take some actual notes