---
layout: post
title: "Install UniFi Network Controller on Debian Buster"
tags: networking server linux
---

To administrate Ubiquiti network equipment you can either buy a UniFi Dream Machine/Cloud Key or host the controller
software yourself. This can be done e.g. in a [FreeNAS Jail]({% post_url 2020-08-15-unifi-controller-on-freenas %})
or on a Debian host (physical or VM). Unfortunately the UniFi Network Controller requires some older software versions
of MongoDB and JVM which makes the installation procedure on a modern Debian Buster not straight-forward.

This guide is based on [Ubiquiti's guide to install the controller on Debian via APT](https://help.ui.com/hc/en-us/articles/220066768-UniFi-How-to-Install-and-Update-via-APT-on-Debian-or-Ubuntu).
I assume that you have a fresh Debian Buster installation to perform the following steps.

First update the system and install some helper packages. Also install `haveged` which ensure that the Linux pseudo
random number generator has sufficient entropy. [This DigitalOcean tutorial](https://www.digitalocean.com/community/tutorials/how-to-setup-additional-entropy-for-cloud-servers-using-haveged)
explains the technical background of that.

```shell
sudo apt update
sudo apt install apt-transport-https ca-certificates gnupg haveged
```

Now we can install the software packages. In addition to the UniFi Network Controller itself, we have to install a
compatible version of MongoDB and the JVM. The UniFi Network Controller version 6.1 (the latest as of May 2021) only
supports MongoDB before 4.0 and the JVM 8. The following steps will guide how to install [MongoDB 3.6](https://docs.mongodb.com/v3.6/tutorial/install-mongodb-on-debian/)
from the official repositories and the [AdoptOpenJDK 8](https://adoptopenjdk.net/) HotSpot VM.


First download the trusted key for the UniFi, AdoptOpenJDK, and MongoDB 3.6 repositories and add them to the Debian
source lists. For MongoDB 3.6 we use the Debian Stretch repo since there is no Debian buster version of MongoDB 3.x.

```shell
wget -qO - https://dl.ui.com/unifi/unifi-repo.gpg | sudo apt-key add -
wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | sudo apt-key add -
wget -qO - https://www.mongodb.org/static/pgp/server-3.6.asc | sudo apt-key add -

echo "deb https://www.ui.com/downloads/unifi/debian stable ubiquiti" | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list
echo "deb https://adoptopenjdk.jfrog.io/adoptopenjdk/deb buster main" | sudo tee /etc/apt/sources.list.d/adoptopenjdk.list
echo "deb http://repo.mongodb.org/apt/debian stretch/mongodb-org/3.6 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.6.list
```

Now we can update the package list, remove any pre-existing OpenJDK 11 installations (which come pre-installed with
Debian Buster), and install the new packages.

```shell
sudo apt update
sudo apt remove openjdk-11-*
sudo apt install unifi adoptopenjdk-8-hotspot mongodb-org
```

Now we can make sure that the UniFi Network Controller, MongoDB, and `haveged` are automatically started at boot.

```shell
sudo systemctl enable unifi.service
sudo systemctl enable mongod.service
sudo systemctl enable haveged.service
```

You're all done! Go to `https://<YOUR_HOST_IP>:8443/` to access the UniFi Network Controller UI.
