---
layout: post
title: "Install Plex Media Server on Debian Buster"
tags: linux server
---

Execute the following steps to add the Plex repository and install Plex Media Server:

```shell
sudo apt update
sudo apt install apt-transport-https ca-certificates gnupg
curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -
echo "deb https://downloads.plex.tv/repo/deb public main" | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
sudo apt update
sudo apt install plexmediaserver
```

Check with `sudo systemctl status plexmediaserver.service` whether the Plex service is running.

## The "Not authorized" error

Assuming that your Plex Media Server's IP is `192.168.1.3` and you try to access the Plex UI at
`http://192.168.1.3:32400/web`, you might run into the error

> Not authorized - You do not have access to this server

The problems seems to be that the inital setup has to be completed from localhost. Since I'm running Plex on a
server without GUI, I'm using an SSH tunnel to accomplish this. Execute

```shell
ssh -L 32400:localhost:32400 user@192.168.1.3
```

and access the remote Plex server in the local browser at http://localhost:32400/web. Now you can finish the inital
setup. Once that's done, you can exit the SSH tunnel and access the Plex Media Server at http://192.168.1.3:32400/web.
