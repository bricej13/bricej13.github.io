---
layout: post
title: Step-by-step guide to creating an Alpine container in Proxmox
date: '2016-06-12 19:02:58'
tags:
- getting-started
- alpine-linux
- proxmox
---

## Download the Alpine template
Templates can be downloaded directly from the Proxmox web interface, though it's a bit hidden. You have to go to _Storage View --> local --> Content --> Templates_. 

![](/content/images/2016/06/Screen-Shot-2016-06-12-at-1-29-55-PM.png)

_**Note:** `local` is the name of my local storage, but yours may be different. Find out where your container templates are stored by going to Datacenter --> Storage and see where 'Container Templates are stored._

From here you can just click on the 'Templates' button, locate the Alpine Linux template and download. 

![](/content/images/2016/06/Screen-Shot-2016-06-12-at-1-34-51-PM.png)

This will download the file to `/var/lib/vz/template/cache` by default. You can also upload templates manually to that location if you wish.


## Creating the container
Let's start by creating a new container from the Proxmox web interface.
![](/content/images/2016/06/Screen-Shot-2016-06-12-at-7-46-01-AM.png)

#### General Information
Next we'll fill out the general information about the container
![](/content/images/2016/06/Screen-Shot-2016-06-12-at-7-49-28-AM-1.png)
<table>
<tr><th>Field</th><th>Purpose</th></tr>
<tr>
<td>Node</td>
<td>Physical machine that the container will live on. I only have one server in my 'cluster' so that is my only choice.</td>
</tr>
<tr>
<td>VM ID</td>
<td>Proxmox's internal vmid for this container. We will be using this number a lot, especially when using the command line tools</td>
</tr>
<tr>
<td>Hostname</td>
<td>Self-explanitory</td>
</tr>
<tr>
<td>Resource pool</td>
<td>Only important if you're using resource pools. Can be ignored if you have a simple setup.</td>
</tr>
<tr>
<td>Password</td>
<td>Password for root user</td>
</tr>
</table>

#### Template

Here you will want to select the Alpine template that we downloaded previously
![](/content/images/2016/06/Screen-Shot-2016-06-12-at-7-58-47-AM.png)

#### Root Disk
The only thing I usually change on this tab is the root disk size. That being said, if you're creating a container that you plan on using a lot of storage, don't add it here. You can mount host directories in the container directly.

![](/content/images/2016/06/Screen-Shot-2016-06-12-at-8-00-57-AM.png)

#### CPU
Just set your CPU core limit and be on your way. Don't worry about oversubscribing to CPU cores, this is just the limit that a single container can use. Also see [this thread](https://forum.proxmox.com/threads/q-lxc-configuration-wizard-what-is-cpu-limit-cpu-unit.22661/) if you want to know about CPU units.

![](/content/images/2016/06/Screen-Shot-2016-06-12-at-8-07-25-AM.png)

#### Memory

Set the memory and swap space that you want allocated to the container. Remember that containers are much more lightweight than VMs, so you don't need to give them as much RAM. You can guess low on this page, and then easily increase it in the future as needed.

![](/content/images/2016/06/Screen-Shot-2016-06-12-at-1-39-08-PM.png)

#### Network

Here is a basic network setup that I use. I like to leave a static IP address and set the last octet to the vmid assigned by Proxmox (112 in this case). This makes life easier if you're running a relatively simple network.

![](/content/images/2016/06/Screen-Shot-2016-06-12-at-1-41-58-PM.png)

#### DNS

Here you can leave all the defaults. Only change this section if you know why you're changing this section.

![](/content/images/2016/06/Screen-Shot-2016-06-12-at-1-45-47-PM.png)

#### Confirm and Finish

You're ready to rip. You should see a successful creation as shown below.

![](/content/images/2016/06/Screen-Shot-2016-06-12-at-1-47-02-PM.png)

## Start and access your new container

From here on out I prefer to use the command line tools to interact with my containers. This is done from an SSH session on the host machine.

Proxmox uses the `pct` tool to manage containers. We'll also need to know the vmid that Proxmox assigned to the container upon creation. 

```bash
# Start the container
pct start <vmid>

# List all containers w/ status
pct list

# Access the container
pct enter <vmid>
```

Once you `pct enter` your container, you should be able to `apk update && apk upgrade`.
```
# pct enter 112                                                                                    
/ # apk update && apk upgrade
fetch http://dl-2.alpinelinux.org/alpine//v3.3/main/x86_64/APKINDEX.tar.gz
v3.3.3-57-gb0ca6a4 [http://dl-2.alpinelinux.org/alpine//v3.3/main]
OK: 5317 distinct packages available
(1/2) Upgrading libcrypto1.0 (1.0.2g-r0 -> 1.0.2h-r0)
(2/2) Upgrading libssl1.0 (1.0.2g-r0 -> 1.0.2h-r0)
OK: 6 MiB in 16 packages
/ # 
```

That's all there is to it! You can now download any package from the [APK Repository](https://pkgs.alpinelinux.org/packages), or [follow one of my guides](/tag/lxc-homelab/).