---
layout: post
title: "Use Docker SMBv1 server to share Music with Sonos"
tags: linux server music
---

If you want to share music files from your local file server to Sonos, you can do that with a Samba share. Unfortunately
Sonos only supports the outdated and insecure SMBv1. You should not enable SMBv1 on your main file server (e.g. TrueNAS)
for security reasons. An alternative way is to use a small Docker container to share files via the SMBv1 protocol. I use
this setup to share music files from my TrueNAS server to my Sonos.

![Sonos SMBv1 Diagram](/assets/images/sonos-samba-diagram.png)

That way the main fileserver does not expose any insecure services and only the Docker container does. Mounting the NFS
and SMB shares read-only further reduces the potential impact. I don't activate any username/password protection for
this share since it's shared read-only, and I'm okay with everyone in my home network to see the share.

## Setup

You need

* A file server with some music and NFS
* A host for Docker containers (e.g. a VM, a bare metal machine, or a Raspberry Pi)
* A Sonos system

Login to the host machine. Create a new directory and the following Dockerfile:

```yaml
version: "3"

services:
  sonos-samba:
    image: dperson/samba:latest
    container_name: samba
    restart: unless-stopped
    command: '-g "ntlm auth = yes" -g "server min protocol = NT1" -S -s "Music;/mnt/music;yes;yes"'
    ports:
      - 139:139
      - 445:445
    environment:
      - TZ=PST8PDT
    volumes:
      - music:/mnt/music

volumes:
  music:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.1.4,nolock,ro,soft,nfsvers=4"
      device: ":/path/to/music-dir"
```

Or download the file from [my collection of docker-compose files](https://github.com/stefanthoss/container-fest):

```shell
mkdir sonos-samba
cd sonos-samba
wget https://raw.githubusercontent.com/stefanthoss/container-fest/main/sonos-samba/docker-compose.yml
```

Adapt the `TZ` variable as necessary to your local time zone. Adapt the volumes section to point to your NFS share, see
[my post about mount NFS shares in Docker]({% post_url 2021-03-03-mount-nfs-share-in-docker-compose %}) for details.

Start the docker-compose service with `docker-compose up -d`. Now you can go to your Sonos app and add
`\\{IP_ADDRESS}\Music` (no username, no password) as a shared music folder.
