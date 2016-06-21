---
layout: post
title: How to setup OpenVPN on an Alpine LXC container
date: '2016-04-09 03:01:51'
tags:
- openvpn
- alpine-linux
- lxc
- lxc-homelab
---

[Alpine](alpinelinux.org) on [LXC](linuxcontainers.org) is an excellent base for setting up an application. This guide will show you how to setup an [OpenVPN](https://openvpn.net/index.php/open-source.html) connection that connects automatically when the container is started.

### Prerequisites
- A server with [LXC installed](https://wiki.debian.org/LXC)
- A basic Alpine LXC container setup - [Guide](https://www.flockport.com/new-micro-containers-based-on-alpine-linux/)

### Setup tun module
In order to connect to an OpenVPN server, we need to have a tun adapter setup in our container. Enabling the tun adapter on the host system is beyond the scope of this guide. You'll have to Google it.

Add the following lines to the container's config file in `/var/lib/lxc/vpn/config` to give your container permission to use the host's tun module, and create the tun node on startup:
```
# Add as last item in devices section
lxc.cgroup.devices.allow = c 10:200 rwm

# Create tun node on bootup
# Add as last item in file
lxc.hook.autodev = sh -c "modprobe tun; cd ${LXC_ROOTFS_MOUNT}/dev; mkdir net; mknod net/tun c 10 200; chmod 0666 net/tun"
```

_**Proxmox Note:** if you're using Proxmox, your container config will be located in `/etc/pve/lxc/<vmid>.conf`_

### Setup OpenVPN client
Enter the container and install OpenVPN
```bash
lxc-attach -n vpn
apk add openvpn
```
Marvel that your container is a mere 12MB! 
##### OpenVPN configuration
You should be able to obtain the OpenVPN configuration from your OpenVPN provider. A [sample configuration](https://openvpn.net/index.php/open-source/documentation/howto.html#client) is available from the OpenVPN website. This file will need to be copied to your container to `/etc/openvpn/openvpn.conf`. This is the default config file that is picked up when the OpenVPN service is started.

Now you can test the VPN connection manually
```bash
openvpn /etc/openvpn/openvpn.conf

# Ensure the tunnel was created by running
ifconfig
```

#### Start OpenVPN at boot
```bash
rc-update add openvpn
```
That's it!
### Test your configuration
Exit your container and reboot it
```bash
lxc-stop -n vpn
lxc-start -n vpn
lxc-attach -n vpn
ifconfig
```
You should be able to see a `tun0` connection in the list of interfaces.

### What Next?
You may want to use OpenVPN on several containers. The easiest way to accomplish this is to stop your new is to clone this container into a new container:
```bash
lxc-clone vpn newvpn
```
That was easy.