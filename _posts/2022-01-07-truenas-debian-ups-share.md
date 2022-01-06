---
layout: post
title: "Share UPS Information from TrueNAS with a Debian Server"
description: "TrueNAS can share information from a USB-connected UPS with a Debian server that is powered by the same UPS."
tags: linux ups server truenas
---

I have one UPS which powers two servers, one TrueNAS storage server and one Debian server. While the UPS is connected via USB to the TrueNAS server, I also want to shut down the Debian server when the UPS reaches low battery. In order to achieve this, the TrueNAS server has to communicate the UPS status to the Debian server. [Network UPS Tools](https://networkupstools.org) can do that and is supported by both TrueNAS/FreeBSD and Debian. This guide also applies to a Proxmox hypervisor which doesn't have any special built-in support for UPS devices and should be treated like a regular Debian server.

## Expose UPS on the Host (TrueNAS)

Enable the UPS service and configure your UPS (check the [TrueNAS UPS documentation](https://www.truenas.com/docs/core/services/ups/) for details). Enable **Remote Monitor** and configure an extra UPS user (change the password `changeme`) that can be used by the Debian server:

```text
[upsmon]
  password = changeme
  upsmon slave
```

![UPS Configuration in TrueNAS](/assets/images/truenas-ups-service.png)

Choose **UPS reaches low battery** as shutdown mode. Your TrueNAS server will now expose the UPS status using [Network UPS Tools](https://networkupstools.org).

## Configure UPS on the Client (Debian)

On the client, we only need the [Network UPS Tools](https://networkupstools.org) client which can be installed via the [nut-client Debian package](https://packages.debian.org/en/bullseye/nut-client) using root privileges:

```shell
apt update
apt install nut-client
```

After the installation, we have to configure NUT to start the network client. Change the `MODE` in the file `/etc/nut/nut.conf` to the following line:

```text
MODE=netclient
```

In the file `nano /etc/nut/upsmon.conf` we have to add the monitoring configuration. Assuming that the TrueNAS server has the IP 192.168.1.2, the UPS is called `rack-ups`, and the UPS is configured as shown in the screenshot shown earlier, add the following line to the file:

```text
MONITOR rack-ups@192.168.1.2 1 upsmon changeme slave
```

The rest of the `upsmon.conf` can stay with the default configuration. Now you can enable system start of the NUT service, start it, and check the service status:

```shell
systemctl enable nut-monitor.service
systemctl start nut-monitor.service
systemctl status nut-monitor.service
```

If the service is running without error messages, you can check with `upsc rack-ups@192.168.1.2` whether the UPS data can be retrieved. The NUT monitor service logs and dmesg should show the following log line:

```text
upsmon[1141]: Communications with UPS rack-ups@192.168.1.2 established
```

The UPS is now configured correctly, and both the host (TrueNAS) and the client (Debian) should shut down when the UPS battery is low.

## Test

To test the setup, we can simulate a low battery event. Use the command `upsmon -c fsd` on the TrueNAS host to trigger a shutdown of all connected clients and itself. After executing the command, the TrueNAS host will notify the Debian client to shut down and then shut down itself. On the Debian client, the NUT monitor service logs and dmesg will show the following log lines:

```text
upsmon[1141]: UPS rack-ups@192.168.1.2: forced shutdown in progress
upsmon[1141]: Executing automatic power-fail shutdown
upsmon[1141]: Auto logout and shutdown proceeding
```

If both servers shut down gracefully, the test was successful.
