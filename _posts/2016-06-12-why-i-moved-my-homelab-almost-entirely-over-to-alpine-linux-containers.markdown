---
layout: post
title: Why I moved my homelab over to Alpine Linux containers
date: '2016-06-12 03:13:54'
tags:
- alpine-linux
- lxc
- lxc-homelab
- proxmox
---

Ever since I purchased my home server (a Dell T20) I have been running [Proxmox](https://www.proxmox.com/en/) for all of my home server needs. I had never used containers before and I assumed that I would be using it to manage virtual servers. I tried using OpenVZ containers (because it's so easy in Proxmox) running Debian and found that they worked great for all of my use cases. I ended up not creating a single real VM.

[Proxmox 4.2](http://pve.proxmox.com/wiki/Roadmap#Proxmox_VE_4.2) introduced support for Alpine Linux and I decided to move as much as I could over.

## Containers vs VMs
The biggest benefit of using containers over virtual machines is the reduction of overhead. Each virtual machine you run has virtual hardware and it's own kernel. If you have 5 virtual machines running on your server, you have 5 sets of virtualized hardware and 5 separate operating system kernels: even if they're exactly the same!

Containers solve this problem by sharing the same kernel. If you have 5 containers on your system you still just have one instance of the kernel. This makes it so that you can run many more containers on a server than you would virtual machines.

Wait! Not all operating systems use the same kernel! You're right. This is a limitation of containers. You can't run a BSD-based OS on a Linux container host, and Windows is out of the question. Fortunately for us, you can do pretty much everything with Linux.

Another nice benefit of Proxmox is that it can also manage virtual machines, so you could create one in a pinch.

## So Docker is like the best container thingie, right?
Docker is so hot right now. It has tons of support, but it's not a good choice for a homelab. Docker was designed for the enterprise environment. It was designed so that each container would really only house a single process. This is an extremely powerful idea, but it comes with a complexity doesn't add much value to a home server setup. 

Here's an example. Let's say that you want to run a Wordpress website. Docker would recommend that you install Wordpress in one container and MySQL in a separate container, and have a third data volume for static content. You would then have to use Docker tools to wire the three containers together so that they can talk to each other. This extra overhead can be a pain in a homelab situation, because we're not going to be using the scaling benefits that it provides.

[OpenVZ](https://openvz.org/Main_Page) and [LXC](https://linuxcontainers.org/) containers are designed with the purpose of replicating the native OS environment. They feel like regular full-fledged operating systems. They are made to run multiple processes. You can run you Wordpress server, database server in the same container and you can have persistent data. Yes please!

## Alpine Linux
I first built my homelab on Debian containers. Everything worked great, but offsite backups were a bit heavy for my abysmal upload rate. I came across Alpine as a possible alternative, and I had to give it a go. 

From the [Alpine](http://www.alpinelinux.org/about/) web site:
> Alpine Linux is built around musl libc and busybox. This makes it smaller and more resource efficient than traditional GNU/Linux distributions. A container requires no more than 8 MB and a minimal installation to disk requires around 130 MB of storage. Not only do you get a fully-fledged Linux environment but a large selection of packages from the repository.

They're not kidding about how small it is, checkout `top` running on a brand new installation:
![](/content/images/2016/06/Screen-Shot-2016-06-11-at-9-39-09-PM.png)

In my experience, Alpine has packages for almost everything I want (Plex is the big exception, but they do have Emby). It may take a little more wrenching than a more popular distribution, but it's worth it for me.

_**Summary:** It's tiny and has tons of easily-installed packages_

## Real life size differences
I have been most impressed by how small the containers are. My container backups have gone from 1Gb+ to 8-100MiB. This makes a huge difference for my off-site backup bandwidth.
<table>
<tr>
<th>Application</th>
<th>Backup Size (gzip)</th>
</tr>
<tr>
<td>Ghost Blog</td>
<td>59MiB</td>
</tr>
<tr>
<td>Laverna</td>
<td>69MiB</td>
</tr>
<tr>
<td>Transmission w/ OpenVPN client</td>
<td>5MiB</td>
</tr>
<tr>
<td>Squid</td>
<td>9MiB</td>
</tr>
<tr>
<td>SickRage</td>
<td>69MiB</td>
</tr>
<tr>
<td>Selenium w/ Firefox</td>
<td>105MiB</td>
</tr>
<tr>
<td></td>
<td></td>
</tr>
</table>

## Guides
I've documented my process for others to follow:

- [Create an Alpine Container in Proxmox](/step-by-step-guide-to-creating-an-alpine-container-in-proxmox/)
- [Squid](/roll-your-own-http-proxy-with-squid-alpine-and-lxc/)
- [Transmission](/how-to-install-transmission-with-vpn-in-an-alpine-lxc-container/)
- [SickRage](/install-sickrage-in-an-alpine-linux-container/)
- [Ghost](/how-to-install-ghost-on-alpine-linux-on-lxc/)
- [Laverna](/how-to-install-laverna-in-an-alpine-lxc-container/)
- [Selenium](/install-a-headless-selenium-server-in-an-alpine-linux-container/)

Have fun!
