---
layout: post
title: "Supermicro Fans Are Repeatedly Ramping Up and Down"
description: "Fan thresholds for low RPM Noctua fans on a Supermicro server motherboards have to be adjusted so that the fans are not repeatedly ramping up and down."
tags: linux server
---

When connecting low RPM Noctua fans to a Supermicro motherboard, I noticed that the fans were repeatedly ramping up and
down every few seconds - essentially cycling between "normal" RPM and maximum RPM. This is because the normal operating
RPM of these fans is below the default threshold of 500 RPM, and the motherboard will go into a critical state and ramp
up the fans to maximum RPM. It can be fixed by setting new thresholds on the Supermicro motherboard that fit the low RPM
fans. By default, these server motherboards expect high RPM fans, as used in server racks.

You will see the following log lines in the Health Event Log if you encounter this issue:

```text
Lower Critical - going low - Assertion
Lower Non-recoverable - going low - Assertion
Lower Non-recoverable - going low - Deassertion
Lower Critical - going low - Deassertion
```

![Critical Supermicro Fan Speed](/assets/images/supermicro-fan-speed-critical.png)

I use multiple Noctua NF-R8 redux-1800 PWM fans on X11 generation server motherboards. The fans are connected to the
FAN1 - FAN6 headers, which are controlled by the CPU temperature (the FANA and FANB headers are not). Below commands are
tested on Debian Bullseye.

First install `ipmitool` which can be used to read and modify the fan controller:

```bash
apt update
apt install ipmitool
```

First check the current sensor's properties:

```text
# ipmitool sensor get FAN2
Locating sensor record...
Sensor ID              : FAN2 (0x42)
 Entity ID             : 29.2
 Sensor Type (Threshold)  : Fan
 Sensor Reading        : 500 (+/- 0) RPM
 Status                : Lower Critical
 Lower Non-Recoverable : 300.000
 Lower Critical        : 500.000
 Lower Non-Critical    : 700.000
 Upper Non-Critical    : 25300.000
 Upper Critical        : 25400.000
 Upper Non-Recoverable : 25500.000
 Positive Hysteresis   : 100.000
 Negative Hysteresis   : 100.000
 Assertion Events      : lcr- 
 Assertions Enabled    : lcr- lnr- ucr+ unr+ 
 Deassertions Enabled  : lcr- lnr- ucr+ unr+
```

The lower critical threshold is set at 500. The Noctua fans I'm using have a maximum RPM of 1800, so they will fall
below this threshold when the motherboard sets the fan speed to 27% or less.

The fix is to create a small Bash script which sets the critical threshold to 100 RPM and the non-recoverable
threshold to 200 RPM. You can set the thresholds for multiple fans at once - in the below example I'm doing it for
FAN2 and FAN3. Create the file `/opt/fancontrol.sh` with the following content (adjust the fan sensor names according
to your setup):

```bash
#!/bin/sh

/usr/bin/ipmitool sensor thresh FAN2 lower 0 100 200
/usr/bin/ipmitool sensor thresh FAN3 lower 0 100 200
```

Make the script executable:

```bash
chmod 755 /opt/fancontrol.sh
```

You could just execute this file to set the new limits. To make sure the new fan thresholds are set even after a
motherboard reset, I created a systemd service that re-applies them after every boot.

Create the service file `/etc/systemd/system/fancontrol.service` with the following content:

```ini
[Unit]
Description=Set fan thresholds for low RPM fans.

[Service]
Type=simple
ExecStart=/bin/bash /opt/fancontrol.sh

[Install]
WantedBy=multi-user.target
```

Set the permissions of the service file and enable the service:

```bash
chmod 644 /etc/systemd/system/fancontrol.service
systemctl enable fancontrol.service
```

After a reboot or manual execution of the script, you can check the new threshold limits:

```text
# ipmitool sensor get FAN2
Locating sensor record...
Sensor ID              : FAN2 (0x42)
 Entity ID             : 29.2
 Sensor Type (Threshold)  : Fan
 Sensor Reading        : 500 (+/- 0) RPM
 Status                : ok
 Lower Non-Recoverable : 0.000
 Lower Critical        : 100.000
 Lower Non-Critical    : 200.000
 Upper Non-Critical    : 25300.000
 Upper Critical        : 25400.000
 Upper Non-Recoverable : 25500.000
 Positive Hysteresis   : 100.000
 Negative Hysteresis   : 100.000
 Assertion Events      : 
 Assertions Enabled    : lcr- lnr- ucr+ unr+ 
 Deassertions Enabled  : lcr- lnr- ucr+ unr+
```

The fans should now run smoothly while being temperature-controlled by the Supermicro motherboard.
