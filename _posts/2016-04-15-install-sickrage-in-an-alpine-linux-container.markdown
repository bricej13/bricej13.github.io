---
layout: post
title: Install SickRage in an Alpine Linux container
date: '2016-04-15 01:40:38'
tags:
- sickrage
- alpine-linux
- lxc
- lxc-homelab
---

This guide will help you install SickRage from source in an Alpine Linux container. 

### Prerequisites
- A server with [LXC installed](https://wiki.debian.org/LXC)
- A basic Alpine LXC container setup - [Guide](https://www.flockport.com/new-micro-containers-based-on-alpine-linux/)

### Setup SickRage
To get started we need to enter our container and install some software packages
```bash
apk update
apk add unrar git openssl libssl1.0 python sudo
```
Next we need to set up a user and group to run SickRage so we aren't running SickRage as root.
```bash
addgroup -S sickrage
adduser -S -h /var/lib/sickrage -g "SickRage" -G sickrage sickrage
```

With that done, let's download our SickRage files
```Bash
mkdir -p /opt/sickrage
git clone --single-branch --depth 1 https://github.com/SickRage/SickRage.git /opt/sickrage
chown -R sickrage:sickrage /opt/sickrage
```

### Setup SickRage service
Sickrage comes with several init scripts. We're going to use the one for Gentoo as it uses the same init system as Alpine
```bash
cp /opt/sickrage/runscripts/init.gentoo /etc/init.d/sickrage
```

Our init script needs a configuration file to run properly. Create a file at `/etc/conf.d/sickrage` with the following lines:
```bash
# If you've been following these lines, you shouldn't need to change any of these settings
SICKRAGE_USER=sickrage
SICKRAGE_GROUP=sickrage
SICKRAGE_DIR=/opt/sickrage
PATH_TO_PYTHON_2=/usr/bin/python2.7
SICKRAGE_DATADIR=/opt/sickrage
SICKRAGE_CONFDIR=/opt/sickrage
```

Now we can try starting up our service for the first time
```bash
service sickrage start
```
_You may see a few errors about `config.ini` not existing. This file is created automatically on the first time the service is started_

Congratulations, you can now visit your SickRage installation at `http://[ipaddress]:8081`

### Add SickRage to startup scripts
It just takes one command to setup SickRage to start at boot
```bash
rc-update add sickrage default
```

#### Boot up test
Exit the container with the `exit` command. Then you need to restart the container and then test if SickRage is running once again.
```bash
# from the lxc host
lxc-shutdown -n [containername]
lxc-start -n [containername]
```


### Start SickRage container at system boot
We've setup our SickRage service to start up when the container boots, but we also need to set up our SickRage container to start when the host boots.

_**Proxmox note:** If using Proxmox, you can set the container to start at system boot in the web interface. This section can be skipped._


From the host machine, edit `/var/lib/lxc/[containername]/config` to add the following line:
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
lxc-stop -n [containername]
cd /var/lib/lxc/[containername]
tar --numeric-owner -czvf [containername]_vanilla.tar.gz ./*
lxc-start -n [containername]
```

### Now what?
You may want to mount some drives in your container. You can do this any way you regularly mount drives in Alpine, or use LXC [bind mounts](https://linuxcontainers.org/lxc/manpages/man5/lxc.container.conf.5.html) to mount host folders in the container.

Setup a [Transmission container with OpenVPN](/how-to-install-transmission-with-vpn-in-an-alpine-lxc-container/) to do the dirty work.