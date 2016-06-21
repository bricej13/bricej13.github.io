---
layout: post
title: Install NGiNX in an Alpine Linux Container
---

## Prerequisites
- An Alpine Linux container
- An internet connection

## Install nginx
```bash
apk add nginx
rc-update add nginx default
rc-service nginx start
```

## Configuration
Config file is located at `/etc/nginx/nginx.conf`.