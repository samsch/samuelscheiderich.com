---
title: First Time Setting Up Caddy in a Container
date: 2016-11-15 17:33:10
tags:
- developer experience
- deployment
- node
- Caddy
- Node.js
- PHP
---
Primary Goal: Setup Caddy as a static web server with a self-signed cert, in a LXD container.
Secondary Goal: Setup proxy server to Node.js for dynamic content.
Tertiary Goal: Setup FastCGI proxy for PHP.

# Article OS assumption

My base OS is Xubuntu 16.10. While technically LXD is not a Ubuntu-specific technology, I'm not as familiar with other distros, and won't be providing instructions for other platforms.

# Do basic startup of container image

For starters, if you don't have LXD/LXC (pronounced Lex-D and Lex-C), check out the [installation instructions.](https://linuxcontainers.org/lxd/getting-started-cli/) Don't run `newgrp lxd` with sudo. I'm not sure why I thought that made sense, but no, that just adds the group to root, and drops you to the root terminal.

Create a new container with `lxc launch ubuntu:16.10 my-ubuntu`. I'm going to hazard a guess that everything will work on 16.04 as well, which is an LTS release.

If you aren't familiar with LXD, we just created a running container based on a generic Ubuntu image. You can stop the container with `lxc stop my-ubuntu`. (The container is named 'my-ubuntu'. You can name yours whatever you want.) To start your container back up, use `lxc start my-ubuntu`. To get a bash console in the container, run `lxc exec my-ubuntu bash`.

# Downloading Caddy

The preferred method of downloading Caddy seems to be as a configurable [tar.gz download](https://caddyserver.com/download). So I'll do that and push it into the container manually. I chose the default HTTP core, 'expires' and 'git' directives/middleware, and no DNS providers (since this is a local-only instance). Make sure you know where you download the server to.

Push the file into the container with `lxc file push -p Downloads/caddy_linux_amd64_custom.tar.gz my-ubuntu/home/ubuntu/`. The `-p` flag tells `lxc file push` to create any folders which don't already exist. If you get an error that the flag does exist, you are probably using lxd on 16.04 or older, which don't have the flag. You can manually create any folder needed.

Enter bash into the container (`lxc exec my-ubuntu bash`), cd to where you pushed the file (`cd /home/ubuntu`), and run `tar -xzf caddy_linux_amd64_custom.tar.gz` to unpack.

Move caddy to `/usr/local/bin` with `mv caddy /usr/local/bin/`, and set ownership and permissions `chown root:root /usr/local/bin/caddy`, `chmod 755 /usr/local/bin/caddy`.

# Setup daemon

We want Caddy to start up with the container. Fortunately, Caddy comes with some unofficial, unsupported scripts for setting this up. Recent Ubuntu uses systemd, so that's the instruction set we'll follow. The scripts are contained in the `init/` folder unpacked from the tar.gz, but you can also read the [readme on github](https://github.com/mholt/caddy/tree/master/dist/init/linux-systemd).

We can skip the groupadd and useradd commands, as the Ubuntu container already has the appropriate 'www-data' user setup.

We need some folders setup:
```bash
# - From the readme
sudo mkdir /etc/caddy
sudo chown -R root:www-data /etc/caddy
sudo mkdir /etc/ssl/caddy
sudo chown -R www-data:root /etc/ssl/caddy
sudo chmod 0770 /etc/ssl/caddy
```

The next step requires that we have a Caddyfile setup.
