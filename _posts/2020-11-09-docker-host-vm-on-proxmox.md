---
layout: post
title: "Docker Host VM on Proxmox"
tags: linux vm proxmox
---

## Install Debian

This post assume that you have installed a fresh Debian Buster as a VM with `sudo` activated.

## Add guest agent for Proxmox

This is helpful to see the IP and networking information in the Proxmox UI. Make sure to enable the QEMU Guest Agent within Proxmox before you do this. Enabling it in Proxmox has to be done either during VM creation in the system section or in the options section after creation.

```shell
sudo apt update
sudo apt install qemu-guest-agent
```

Restart the VM for the QEMU Guest Agent to register with Proxmox.

## Install Docker

See <https://docs.docker.com/engine/install/debian/> and <https://docs.docker.com/compose/install/>.

```shell
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```

Finally, add your non-root user to the Docker group (replace `USERNAME` with yout username):

```shell
sudo usermod -aG docker USERNAME
```

You have to end your current SSH session and reconnect in order for the group modification to be effective.

## Launch Applications

Launch applications with the docker-compose files from <https://github.com/stefanthoss/container-fest>.
