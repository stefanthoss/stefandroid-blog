---
layout: post
title: "Install UniFi Network Controller in a FreeNAS Jail"
tags: freenas server
---

You can use the UniFi Network Controller to administrate Ubiquiti network equipment from a macOS/Linux/Windows client
if you have a Dream Machine or Cloud Key. But there's another way: Using the FreshPort
[unifi5](https://www.freshports.org/net-mgmt/unifi5) it's possible to install the controller in a FreeNAS jail so you
don't have to run a server 24/7 for this specific purpose. This FreshPort is not an official Ubiquiti package but as of
July 2020 it's well maintained.

## Installation

Setup a new FreeNAS jail in the FreeNAS web interface with **Jails** → **Add**. You can use all the default settings or
change them if you know what you're doing. Make sure you activate auto-start. Once the new jail is running, click on the
**Shell** button to enter the Jail's shell. Then install the `unifi5` package:

```shell
pkg upgrade
pkg install unifi5
```

To enable autostart for the UniFi Network Controller add the line `unifi_enable="YES"` in `/etc/rc.conf`.

Then start the service:

```shell
service unifi start
```

In the Jail overview of the FreeNAS web interface you can see the IPv4 address of the jail, e.g. `192.168.1.16`. Go to
<https://192.168.1.16:8443> to visit the Network Controller's interface which will guide you through the rest of the
setup.
