---
layout: post
title: How to install Ghost on Alpine Linux on LXC
date: '2016-04-08 04:43:52'
tags:
- alpine-linux
- ghost-tag
- lxc
- lxc-homelab
---

This guide will get you [Ghost](https://ghost.org/) running inside a super tiny and secure [Alpine Linux](http://alpinelinux.org/) container.

### Prerequisites
- A server with [LXC installed](https://wiki.debian.org/LXC)
- A basic Alpine LXC container setup - [Guide](https://www.flockport.com/new-micro-containers-based-on-alpine-linux/)

### Setup Ghost
#### Install dependencies
```bash
apk add curl unzip nodejs
```

#### Download Ghost files
```bash
mkdir -p /var/www/ghost
cd /var/www/ghost
curl -L https://ghost.org/zip/ghost-latest.zip -o ghost.zip
unzip -uo ghost.zip
```

#### Install Ghost
From the `/var/www/ghost` directory
```bash
npm install --production 
```
#### Edit Config
```Bash
cp /var/www/ghost/config.example.js /var/www/ghost/config.js
```
At minimum you will need to chage the `production.server.host` value to `0.0.0.0`
#### Start your new site
Again, from the `/var/www/ghost` directory.
```bash
npm start --production
``` 
You can now visit your site at `http://[ip address]:2368`

Additional setup information is available in the [Ghost Documentation](http://support.ghost.org/installing-ghost-linux/)

### Start Ghost at container boot
We will use the local service to run our start up script when the system boots
#### Enable local service
```bash
rc-update add local
```
#### Create startup script
Create a file called `/etc/local.d/ghost.start` and add the following lines:
```bash
cd /var/www/ghost
/usr/bin/npm start --production
```
#### Make the script executable
```bash
chmod +x /etc/local.d/ghost.start
```

#### Boot up test
Exit the container with the `exit` command. Then you need to restart the container and then test if Ghost is running once again.
```bash
# from the lxc host
lxc-shutdown -n ghost
lxc-start -n ghost
```
If Ghost does not come back up, look at the log stored in `/npm-debug.log`

### Start Ghost container at system boot
We've setup our Ghost service to start up when the container boots, but we also need to set up our Ghost container to start when the host boots.

From the host machine, edit `/var/lib/lxc/ghost/config` to add the following line:
```bash
lxc.start.auto = 1
```

To test to make sure your container will boot on startup, shut down the container and then run the following command:
```bash
lxc-autostart
```
This command starts all containers that are marked to boot on startup. After running this command you should be able to navigate to your Ghost blog from your web browser.

### Create clean backup
Not that you would ever screw up your container, but it's a good idea to create a backup once everything is setup properly, and before any real changes are made.
```bash
lxc-stop -n ghost
cd /var/lib/lxc/ghost
tar --numeric-owner -czvf ghost_vanilla.tar.gz ./*
lxc-start -n ghost
```

### Now what?
I don't know. Maybe checkout the [Getting Started Guide](http://support.ghost.org/getting-started/).