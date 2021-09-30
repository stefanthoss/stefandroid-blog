---
layout: post
title: "Upgrade Debian from Buster to Bullseye"
description: "With the release of Debian 11 Bullseye, it's time to upgrade existing Debian 10 Buster servers using a simple upgrade process with a few Apt repository changes."
tags: linux server
---

I run Debian 10 Buster on all of my [Proxmox VMs]({% post_url 2021-07-02-proxmox-debian-cloud-init-template %}). This
week, Debian 11 Bullseye was released, so it's time to upgrade. You can check with `cat /etc/os-release` what OS version
you're running at the moment. For further details on the upgrade process, check out the
[official Debian upgrade guide](https://wiki.debian.org/DebianUpgrade).

I strongly recommend executing all of these steps in a [`tmux` session](https://github.com/tmux/tmux). The SSH
connection might get interrupted during the upgrade and with `tmux` you can easily reconnect to the terminal.
Furthermore, I also strongly recommend making a backup of your system before attempting the upgrade.

First, upgrade all packages to the lastest version available in Buster:

```shell
sudo apt update
sudo apt full-upgrade
```

Now the repository has to be changed from Buster to Bullseye. In the repository configuration file `/etc/apt/sources.list`,
replace `buster` with `bullseye`. The security suite is now named `bullseye-security` instead of `buster/updates` and
has to be renamed as well. I also noticed that all the sources were defined with HTTP connections, so let's convert
those to HTTPS while we're at it. It's a good idea to either keep a backup of the old `/etc/apt/sources.list` file or
comment out the existing lines instead of modifying them. That way you can revert in case anything goes wrong.

In summary,

```text
deb http://deb.debian.org/debian/ buster main
deb http://deb.debian.org/debian/ buster-updates main
deb http://security.debian.org/debian-security buster/updates main
```

becomes

```text
deb https://deb.debian.org/debian/ bullseye main
deb https://deb.debian.org/debian/ bullseye-updates main
deb https://deb.debian.org/debian-security bullseye-security main contrib
```

Check with `ls -l /etc/apt/sources.list.d/` whether there are any custom source configuration files that need to change
from `buster` to `bullseye`. If so, change them as well.

Now it's time to perform the upgrade and remove unused dependencies:

```shell
sudo apt clean
sudo apt update
sudo apt full-upgrade
sudo apt autoremove
```

Finally, reboot the machine with `sudo shutdown -r now`. You'll be running Debian 11 Bullseye.
