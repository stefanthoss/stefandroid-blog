---
layout: post
title: "Use Nextcloud All-in-One with a datadir on a Samba share"
description: "You can use the Docker-based Nextcloud All-in-One deployment while having the datadir located on an external Samba share."
tags: server linux docker
---

[Nextcloud All-in-One](https://github.com/nextcloud/all-in-one) is an easy-to-deploy Nextcloud environment. I recently migrated from a custom bare-metal installation of Nextcloud to the AIO method. I like to have my Nextcloud data directory located on my NAS via a Samba share and not on the Nextcloud host itself. It was not immediately obvious to me how to do that with Nextcloud AIO so I'm documenting it here.

The AIO setup is inherently a bit tricky -- you have little control over it since it's not entirely tracked in Docker Compose. You just create the mastercontainer which then creates all the other containers through the Docker API. I find it a little bit difficult to debug but once it's running, it's running fast and stable. I hope that it pays off long-term compared to a bare-metal installation.

Here is the Docker Compose file I'm using:

```yaml
services:
  nextcloud-aio-mastercontainer:
    image: ghcr.io/nextcloud-releases/all-in-one:latest
    init: true
    restart: unless-stopped
    container_name: nextcloud-aio-mastercontainer
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config
      - nextcloud_aio_nextcloud_datadir:/mnt/ncdata
      - /var/run/docker.sock:/var/run/docker.sock:ro
    network_mode: bridge
    ports:
      - 8080:8080
    environment:
      APACHE_PORT: 11000
      APACHE_IP_BINDING: 0.0.0.0
      NEXTCLOUD_DATADIR: nextcloud_aio_nextcloud_datadir
      SKIP_DOMAIN_VALIDATION: false

volumes:
  nextcloud_aio_mastercontainer:
    name: "nextcloud_aio_mastercontainer"

  nextcloud_aio_nextcloud_datadir:
    name: "nextcloud_aio_nextcloud_datadir"
    driver_opts:
      type: "cifs"
      device: "//192.168.1.4/nextcloud-aio/nextcloud"
      o: "rw,mfsymlinks,seal,username=nextcloud,password=PASSWORD,uid=33,gid=0,file_mode=0770,dir_mode=0770"
```

Change the Samba server IP, the share name/path, the Samba user and Samba password but leave the other [mount options](https://github.com/nextcloud/all-in-one?tab=readme-ov-file#can-i-use-a-cifssmb-share-as-nextclouds-datadir) as above.

The important part is to not change any names or mount paths. The datadir volume name and the config volume name need to be exactly as above, otherwise it won't work. It took me some time to discover that the environment variable `NEXTCLOUD_DATADIR` needs to set to the volume name and only this exact volume name is accepted by Nextcloud. It also needs to be mounted as `/mnt/ncdata` in the mastercontainer and there doesn't seem to be any way to change that path.

If you see an error message like

```
Type: Slim\Exception\HttpNotFoundException
Code: 404
Message: Not found.
File: /var/www/docker-aio/php/vendor/slim/slim/Slim/Middleware/RoutingMiddleware.php
```

in the mastercontainer's logs, it's likely that your installation failed because of an incorrect datadir setup or incorrect permissions on the Samba share. In that case you need to fix the permissions, [reset the instance](https://github.com/nextcloud/all-in-one#how-to-properly-reset-the-instance), and retry. A bit of a cumbersome process.

I'm using it behind a Traefik reverse proxy and that's [well-documented in the GitHub repo](https://github.com/nextcloud/all-in-one/blob/main/reverse-proxy.md#traefik-2).
