---
layout: post
title: "Install Plex Media Server on Debian Buster"
description: "Installing Plex Media Server on Debian 10 Buster using the official Apt repository is quick and easy."
tags: linux plex server
---

I run my [Plex Media Server](https://www.plex.tv) on a Debian VM. I think this is the easiest way to run a
Plex Media Server and I prefer it over the Docker container version. I'm using a
[Debian Buster Proxmox template]({% post_url 2021-07-02-proxmox-debian-cloud-init-template %}) to set up the base VM quickly.

[Plex's installation article](https://support.plex.tv/articles/200288586-installation) provides detailed
instructions for the Plex installation on various operating systems. For Debian it's straight-forward, just execute
the following steps to add the Plex repository and install the Plex Media Server:

```shell
sudo apt update
sudo apt install apt-transport-https ca-certificates gnupg
curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -
echo "deb https://downloads.plex.tv/repo/deb public main" | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
sudo apt update
sudo apt install plexmediaserver
```

Check with `sudo systemctl status plexmediaserver.service` whether the Plex service is running. You can access the Plex
UI at `http://<PLEX_SERVER_IP>:32400/web` and follow the instructions for the initial setup.

The logs are written to `/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Logs/`. In case that the
Plex Media Server is not working, check the main log file `Plex Media Server.log` for error messages.

## The "Not authorized" error

During the initial setup, you might see the error message

> Not authorized - You do not have access to this server

on the Plex UI. The problem seems to be that the initial setup cannot be done from a different subnet. So if
your Plex Media Server runs in a different VLAN, you'll face the above error message. The setup has to be completed from
localhost or using an SSH tunnel. Since I'm running Plex on a server without GUI, I'm using an SSH tunnel to accomplish
this. Execute

```shell
ssh -L 32400:localhost:32400 user@<PLEX_SERVER_IP>
```

to establish the tunnel and expose the Plex UI on port 32400 of your local machine. Now you can access the remote Plex
Media Server in the local browser at <http://localhost:32400/web> and finish the initial setup. Once that's done, you can
exit the SSH tunnel and access the Plex Media Server at `http://<PLEX_SERVER_IP>:32400/web`.
