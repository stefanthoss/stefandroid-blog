---
layout: post
title: "Use Plex Media Server with a Let’s Encrypt certificate"
tags: linux security server
---

You can use a free [Let’s Encrypt](https://letsencrypt.org) certificate for your self-hosted Plex Media Server. With
[Certbot](https://certbot.eff.org) and a simple Bash script this is an easy setup which will provide a secure connection
without certificate warnings and auto renew certificates. It will work on any Linux distribution (I'm using Debian Buster).

## Motivation

There are two possible options how to secure the connection to your Plex server when exposing it to the public Internet:

1. Use a reverse proxy like HAProxy or nginx that forwards the traffic and performs SSL offloading.
2. Use Plex's remote access feature and forward the port on your firewall directly to your Plex server.

Generally I would prefer the reverse proxy since I can use my existing reverse proxy which already has a valid Let's Encrypt
certificate. Also, I have more control on how the server is exposed to the public Internet. Unfortunately a few Plex
features like the Sonos integration and the mobile Plex apps are not working with this setup, so I chose the remote
access (reverse proxy will work fine if those features are not important for you). The following guide will explain how
to use a valid Let's Encrypt certificate with Plex remote access.

## Setup

First, install Certbot. The EFF provides [installation guides](https://certbot.eff.org/instructions) for multiple
operating systems. For Debian the official recommendation is using Snap:

```shell
sudo apt update
sudo apt install snapd
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

## HTTP or DNS Let's Encrypt Challenge

```shell
sudo snap set certbot trust-plugin-with-root=ok
sudo snap install certbot-dns-cloudflare
```

I'm using `plex.example.com` as a domain, replace this with your own.

## P12 Certificate

Create the `/etc/letsencrypt/renewal-hooks/post/create_p12_file.sh` file with the following content:

```shell
#!/bin/sh

openssl pkcs12 -export \
  -out /var/lib/plexmediaserver/plex_certificate.p12 \
  -in /etc/letsencrypt/live/plex.example.com/cert.pem \
  -inkey /etc/letsencrypt/live/plex.example.com/privkey.pem \
  -certfile /etc/letsencrypt/live/plex.example.com/chain.pem \
  -passout pass:PASSWORD

chmod 755 /var/lib/plexmediaserver/plex_certificate.p12
```

The `chmod` is necessary because the script will be executed as root and the Plex user has to be able to access the file.

Obtain the initial certificate:

```shell
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \
  -m youremail@example.com \
  -d plex.example.com
```

Install the cronjob (see [Certbot automated renewals](https://certbot.eff.org/docs/using.html#setting-up-automated-renewal)
for details):

```shell
SLEEPTIME=$(awk 'BEGIN{srand(); print int(rand()*(3600+1))}'); echo "0 0,12 * * * root sleep $SLEEPTIME && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

Use `sudo certbot renew --force-renewal` to force a renewal and trigger the post renewal hook.

## Use the Let's Encrypt certificate in Plex
