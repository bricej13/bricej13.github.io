---
layout: post
title: Roll your own HTTP proxy with Squid, Alpine, and LXC
date: '2016-04-08 20:59:00'
tags:
- alpine-linux
- squid
- lxc
- http-proxy
- lxc-homelab
---

By setting up your own [Squid](http://www.squid-cache.org/) HTTP proxy you can bypass corporate firewalls. [Alpine](http://alpinelinux.org/) running in a [Linux Container](https://linuxcontainers.org/) is a good choice for security and resource usage. 

### Prerequisites
- A server with [LXC installed](https://wiki.debian.org/LXC)
- A basic Alpine LXC container setup - [Guide](https://www.flockport.com/new-micro-containers-based-on-alpine-linux/)

### Create your base Alpine container
This is not meant to be an exhaustive guide to setting up a container.
```bash
lxc-create --template alpine -n squid
lxc-start -n squid
lxc-attach -n squid
setup-alpine
```
Follow the prompts of the `setup-alpine` script to finish setting up your system.

### Setup Squid
#### Install Squid package
```bash
apk add squid
service squid start
```

#### If using Proxmox
Proxmox doesn't add `/dev/shm` by default into the new container. To fix this, let's make a new file `/usr/share/lxc/config/alpine.common.conf` with the following lines:

```
lxc.include /usr/share/lxc/config/common.conf

lxc.mount.entry = none dev/shm tmpfs rw,nosuid,nodev,create=dir
```

#### Configure Squid
Squid works right out of the box once it has been started. To connect to Squid, you will need to point your computer's proxy to `http://[your-ip-address]` on port `3128` without a username and password.

#### Secure your Squid server
By default Squid allows anonymous connections. You definitely need to fix this, as malicious actors are scanning all parts of the internet at all time. It is only a matter of time before your brand new proxy has been discovered. Luckily this is easy to fix.
###### Create a password file
We can create a basic password file using the utility `htpasswd`
```bash
apk add apache2-utils
htpasswd -c /etc/squid/passwords [username]

# Follow the prompts to finish creating the file.

# Ensure password file is readable by squid user
chmod 666 /etc/squid/passwords
```
 _Note: additional users can be added by running the command `htpasswd /etc/squid/passwords [username2]`._

###### Update Squid to use password

Now we need to tell squid to use that file for authentication by editing the Squid config file `/etc/squid/squid.conf`. Add the following lines before the part that says `http_access allow localnet`.
```Bash
/usr/lib/squid/basic_ncsa_auth /etc/squid/passwords
auth_param basic realm proxy
acl authenticated proxy_auth REQUIRED
http_access allow authenticated
```

Time to restart Squid
```Bash
service squid restart
```

You should now test connecting to your Squid server this time. Don't forget to add the credentials you just created.

### Start Squid at container boot
By default, Squid won't start at boot. This is easy to fix.
```bash
rc-update add squid
```

#### Boot up test
Exit the container with the `exit` command. Then you need to restart the container and then test if Squid is running once again.
```bash
# from the lxc host
lxc-shutdown -n squid
lxc-start -n squid
```

### Start Squid container at system boot
We've setup our Squid service to start up when the container boots, but we also need to set up our Squid container to start when the host boots.

From the host machine, edit `/var/lib/lxc/squid/config` to add the following line:
```bash
lxc.start.auto = 1
```

To test to make sure your container will boot on startup, *shut down the container* and then run the following command:
```bash
lxc-autostart
```
> This command starts all containers that are marked to boot on startup. After running this command you should be able to hit your proxy.

### Create clean backup
Not that you would ever screw up your container, but it's a good idea to create a backup once everything is setup properly, and before any real changes are made.
```bash
lxc-stop -n squid
cd /var/lib/lxc/squid
tar --numeric-owner -czvf squid_vanilla.tar.gz ./*
lxc-start -n squid
```
Copy your backup to a safe space!

### Cleanup
If you want, you can get rid of stuff you'll no longer need.
```Bash
apk del apache2-utils sshd
```
### Now what?
I use [Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif) on Chrome to manage my proxy connection. I like to use auto-switch rules to just route blocked domains (giphy!) through the proxy.