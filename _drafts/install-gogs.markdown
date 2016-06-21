---
layout: post
title: Install gogs
---

## Add testing repository
Edit `/etc/apk/repositories` and add the following lines:
```
@edge http://dl-2.alpinelinux.org/alpine/edge/main
@testing http://dl-2.alpinelinux.org/alpine/edge/testing
```

Save the file, and then update the repositories
```
apk update
```

## Install Gogs from repository
```
apk add gogs@testing sqlite
```

### Current status
- I added a conf.ini file in the `/usr/bin/custom/conf` directory. This got rid of one warning, but it's not a good solution because we really don't want to be placing config files in the `/usr/bin`.
- I added the missing `.VERSION` to `/usr/bin/templates`. I can't figure out what the correct value for that file is. Still getting the following error:
```
2016/06/10 04:28:37 [web.go:77 checkVersion()] [E] Binary and template file version does not match, did you forget to recompile?
```
- I'm just running `gogs web` directly.