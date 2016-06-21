---
layout: post
title: How to install Transmission (with VPN!) in an Alpine LXC container
date: '2016-04-11 03:30:38'
tags:
- openvpn
- alpine-linux
- lxc
- transmission
- lxc-homelab
---

[Alpine](http://alpinelinux.org) + [OpenVPN](http://linuxcontainers.org) + [Transmission](https://www.transmissionbt.com/) == a very small very secure seedbox. 

### Prerequisites
- A server with [LXC installed](https://wiki.debian.org/LXC)
- Basic knowledge of the command line

### Setup OpenVPN (Optional)
I have written a guide to getting OpenVPN setup in an Alpine container.

> [How to setup OpenVPN on an Alpine LXC container](/how-to-setup-transmission-with-vpn-on-an-alpine-lxc-container)

### Install Transmission
Alpine has a package for for Transmission, so we'll just add it and enable to to run at boot.
```bash
apk add transmission-daemon
rc-update add transmission-daemon
```

### Configuring Transmission
Transmission is installed, but it will only allow us to connect to the web interface from localhost. We'll need to change that. The transmission config file is `/var/lib/transmission/config/settings.json/`.

_**Note:** if the file doesn't exist start and stop the transmission-daemon service and the file should get created._

To allow external connections to the web interface you will have to change the value of `rpc-whitelist`. You can change it to `*` to whitelist the whole world, `192.168.1.*` to whitelist your subnet, or set `rpc-whitelist-enabled` to `false` to disable it completely.

_**Note:** make sure the transmission-daemon service is stopped while making changes to `settings.json`. If not your new settings will be overwritten and lost when the service stops._

##### Password setup
If you're going to make the web interface visible to the whole world, we'll want to set up a password. To do so, we will need to edit 3 lines in `settings.json`.
```javascript
"rpc-authentication-required": true, #change from false
"rpc-password": "hunter2", #set your password
"rpc-username": "admin", #set your username
```

Don't worry about putting your password in plain-text. The next time Transmission starts it will automatically replace the plain text version with a hashed version.

Now it's time to start Transmission
```bash
service transmission-daemon start
```
You should be able access Transmission by navigating to `http://[ipaddress]:9091`

### Attaching storage
Storage can be attached to an Alpine container in many ways, but the easiest is to map a folder from the host. After the destination folder is created within the container, we can bind the folder from the container's config file. 

```bash
# Create folder in container that will be bound to
mkdir -p /media/data/downloads
# Exit and shut down container
exit
lxc-stop -n [containername]
```
Edit `/var/lib/lxc/[containername]/config` to contain the following line:
```
lxc.mount.entry=/host/data/Downloads media/data/downloads none bind 0 0
```
Start your container and your data should be visible within the container. You can set your download directory from the web interface.

_**Proxmox Note:** if you're using Proxmox, your container config will be located in `/etc/pve/lxc/<vmid>.conf`_

### Create clean backup
Not that you would ever screw up your container, but it's a good idea to create a backup once everything is setup properly, and before any real changes are made.
```bash
lxc-stop -n [containername]  
cd /var/lib/lxc/[containername]  
tar --numeric-owner -czvf [containername]_vanilla.tar.gz ./*  
lxc-start -n [containername]  
```

Copy your backup to a safe space!

### Now what?
Check out the guide on [setting up SickRage](/install-sickrage-in-an-alpine-linux-container/)
