---
layout: post
title: Install a Headless Selenium server in an Alpine Linux container
date: '2016-06-11 14:42:36'
tags:
- alpine-linux
- lxc
- selenium
---

[Selenium](http://www.seleniumhq.org/) is a web automation tool. It can be used to crawl and scrape websites. It requires a web browser installed to use as a web driver: we're going to use Firefox. We'll also need to trick Firefox into thinking there is a display attached: we'll user Xvfb for that. 

One setup, you'll be able to run Selenium scripts at your pleasure. 

## Prerequisites
- A virgin Alpine Linux container
- A medium knowledge of Linux

## Add Firefox repository to APK
As of this blog post, Firefox is not available in the main alpine repository. You can [search the Alpine repository](https://pkgs.alpinelinux.org/packages?name=firefox&branch=&repo=&arch=&maintainer=#) to find out where you can install Firefox from. In my case, it is available in the `community` repository branch `v3.3`.

With that information, I can add the following line to `/etc/apk/repositories`
```bash
http://dl-2.alpinelinux.org/alpine//v3.3/community
```

Update your cache to pull from the new repository
```bash
apk upgrade --update-cache --available
```

## Install Selenium dependencies
Now you should be install Firefox and all other Selenium dependencies without issue.
```bash
apk add xvfb firefox dbus py-pip ttf-dejavu
```
Here is what we just installed:
<table>
<thead>
<td><b>Repository</b></td>
<td><b>Purpose</b></td>
</thead>
<tr>
<td>xvfb</td>
<td>A virtual display driver. We're building a headless system, so we need a virtual display for our web browser to run in.</td>
</tr>
<tr>
<td>firefox</td>
<td>We will use the Firefox driver for Selenium. This is what we will use for scraping web sites.</td>
</tr>
<tr>
<td>py-pip</td>
<td>Used to install Selenium drivers for Python</td>
</tr>
<tr>
<td>ttf-dejavu</td>
<td>Fonts! These are required for Firefox to render pages</td>
</tr>
</table>

## Setup Xvfb virtual display server
#### Test Xvfb is setup properly
We should be able to run the Xvfb virtual display server by running
```bash
Xvfb :99 -ac &
```
Check to make sure your display server is running using the `top` command.

_**Note:** If you get an error complaining about a machine-id, install the `dbus` package and run `dbus-uuidgen > /etc/machine-id`. You can then uninstall dbus._


#### Setup Xvfb to start on system boot
We will use the `local` service to create a basic script that will start our service at boot time. We'll start by enabling the `local` service to run at boot.
```bash
rc-update add local default
```

Now we need to create our start script. When `local` service is enabled, it will run all executable scripts in the `/etc/local.d/` that end with `.start` at boot time. It will run all `.stop` scripts when the `local` service is stopped

In our case, we just need a single script `/etc/local.d/Xvfb.start` with the following lines:
```bash
#!/bin/sh
/usr/bin/Xvfb :99 -ac &
```
Now make it executable:
```bash
chmod +x /etc/local.d/Xvfb.start
```

_**Note:** Now is a good time to test that everything is working smoothly. Stop and start your container and then check to see if the Xvfb service is running_

_**Note:** Creating a script to shutdown Xvfb is left as an exercise for the reader_

## Install Selenium with Python
We're going to use Python Selenium bindings because it's really easy to get started.
```bash
pip install --upgrade pip # ensure pip is upgraded to latest version
pip install selenium requests
```

That's it. You can checkout the Selenium Python [Getting Started Guide](http://selenium-python.readthedocs.io/getting-started.html).

## Conclusion
