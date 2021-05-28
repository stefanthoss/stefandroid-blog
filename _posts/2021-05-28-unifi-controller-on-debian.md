---
layout: post
title: "Install UniFi Network Controller on Debian Buster"
tags: networking server linux
---

To administrate Ubiquiti network equipment you can either buy a UniFi Dream Machine/Cloud Key or host the controller
software yourself. This can be done on almost any Linux host, e.g. in a
[FreeNAS Jail]({% post_url 2020-08-15-unifi-controller-on-freenas %}) or on a Debian server - both physical and VM.
Unfortunately the current UniFi Network Controller (version 6.1) requires some older versions of MongoDB and JVM which
makes the installation procedure on a modern Debian 10 "Buster" a bit more complicated.

This guide is based on [Ubiquiti's guide to install the controller on Debian via APT](https://help.ui.com/hc/en-us/articles/220066768-UniFi-How-to-Install-and-Update-via-APT-on-Debian-or-Ubuntu).
I assume that you have a fresh Debian Buster installation to perform the following steps.

First install some helper packages and `haveged` which ensure that the Linux pseudo
random number generator has sufficient entropy. [This DigitalOcean tutorial](https://www.digitalocean.com/community/tutorials/how-to-setup-additional-entropy-for-cloud-servers-using-haveged)
explains the technical background of that.

```shell
sudo apt update
sudo apt install apt-transport-https ca-certificates gnupg haveged
```

Now we can install the software packages. In addition to the UniFi Network Controller itself, we have to install a
compatible version of MongoDB and the JVM. The UniFi Network Controller version 6.1 (the latest as of May 2021) does not
support the current MongoDB 4.x and Java 11. You will need MongoDB 3.x and Java 8 for the controller software to work.
The following steps will guide you through the installation of [MongoDB 3.6](https://docs.mongodb.com/v3.6/tutorial/install-mongodb-on-debian/) and the [AdoptOpenJDK 8](https://adoptopenjdk.net/) HotSpot VM from the official repositories.

First, download the trusted key for the UniFi, AdoptOpenJDK, and MongoDB 3.6 repositories and add them to the Debian
source lists. For MongoDB 3.6 we use the Debian Stretch repo since there is no Debian Buster repo for MongoDB 3.x.

```shell
wget -qO - https://dl.ui.com/unifi/unifi-repo.gpg | sudo apt-key add -
wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | sudo apt-key add -
wget -qO - https://www.mongodb.org/static/pgp/server-3.6.asc | sudo apt-key add -

echo "deb https://www.ui.com/downloads/unifi/debian stable ubiquiti" | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list
echo "deb https://adoptopenjdk.jfrog.io/adoptopenjdk/deb buster main" | sudo tee /etc/apt/sources.list.d/adoptopenjdk.list
echo "deb http://repo.mongodb.org/apt/debian stretch/mongodb-org/3.6 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.6.list
```

Now we can update the package list, remove the existing OpenJDK 11 installation (which comes pre-installed with
Debian Buster), and install the new packages.

```shell
sudo apt update
sudo apt remove openjdk-11-*
sudo apt install unifi adoptopenjdk-8-hotspot mongodb-org
```

Finally, make sure that the UniFi Network Controller, MongoDB, and `haveged` are automatically started at boot.

```shell
sudo systemctl enable unifi.service
sudo systemctl enable mongod.service
sudo systemctl enable haveged.service
```

You're all done! Go to `https://<YOUR_HOST_IP>:8443/` to access the UniFi Network Controller UI. If the UI is not accessible,
check with `sudo systemctl status unifi.service` whether the UniFi service started successfully or there are any error messages.
