---
layout: post
title: "Use Plex Media Server with a Let's Encrypt certificate"
tags: linux security server
---

You can use a free [Let's Encrypt](https://letsencrypt.org) certificate for your 
[self-hosted Plex Media Server VM]({% post_url 2021-07-17-install-plex-on-debian %}). With
[Certbot](https://certbot.eff.org) and a simple Bash script this will provide a secure connection without certificate
warnings. It will also auto renew certificates. I'm using Debian Buster but this will work on any Linux distribution.

## Motivation

There are two possible options how to secure the connection to your Plex server when exposing it to the public Internet:

1. Use a reverse proxy like HAProxy or nginx that forwards the traffic and performs SSL offloading.
2. Use Plex's remote access feature and forward the port on your firewall directly to your Plex server.

Generally I would prefer the reverse proxy since I can use my existing reverse proxy which already has a valid Let's Encrypt
certificate. Also, I have more control on how the server is exposed to the public Internet. Unfortunately a few Plex
features like the Sonos integration and the mobile Plex apps are not working with this setup since they need direct
access. So I chose the remote access (reverse proxy will work fine if those features are not important for you). The
following guide will explain how to use a valid Let's Encrypt certificate with Plex remote access.

## Setup

You need to forward the public port on your firewall to your Plex server. The detailed instructions for this depend on
the type of firewall you're using (in case you use pfSense you have to
[add a firewall rule](https://docs.netgate.com/pfsense/en/latest/firewall/rule-list-intro.html) on the WAN interface).

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

Verification of the domain can either be done via an HTTP challenge or a DNS challenge. I choose a DNS challenge because
it doesn't require opening port 80 from the outside. In the following I use a
[DNS challenge](https://certbot.eff.org/docs/using.html#dns-plugins) using Cloudflare. But the rest of this guide works
the same even when you choose to use a different type of challenge or a different DNS provider.

Install the [DNS Cloudflare plugin](https://certbot-dns-cloudflare.readthedocs.io/en/stable/):

```shell
sudo snap set certbot trust-plugin-with-root=ok
sudo snap install certbot-dns-cloudflare
```

Create the file `~/.secrets/certbot/cloudflare.ini` with the API token you got from Cloudflare:

```ini
dns_cloudflare_api_token = SECRET_TOKEN
```

## PKCS #12 Certificate

Certbot generates a private key file, a certificate file, and the CA file. Plex however expects a PKCS #12 certificate
file that bundles all of these together. I created a script that is triggered by Certbot's renewal hook which will convert
the Certbot output into a PKCS #12 certificate file that is compatible with Plex.

In the following, I'm using `plex.example.com` as a domain, replace this with your own domain name. Also replace
`PASSWORD` with a random password of your choice.

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

Go to the `Network` tab of the Plex settings. Set the following settings (replace `PASSWORD` and `plex.example.com`):

* Secure connections: Required
* Custom certificate location: `/var/lib/plexmediaserver/plex_certificate.p12`
* Custom certificate encryption key: `PASSWORD`
* Custom certificate domain: `plex.example.com`
* Enable Strict TLS configuration
* Custom server access URLs: `https://plex.example.com:32400`

![Plex Network PKCS #12 Certificate](/assets/images/plex-network-p12-certificate.png)

## Forward the port
