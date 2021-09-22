---
layout: post
title: "Supermicro Fans Are Repeatedly Ramping Up and Down"
tags: linux server
---

When connecting low RPM Noctua fans to a Supermicro motherboard, I noticed that the fans were repeatedly ramping up and
down and cycling between "normal" RPM and maximum RPM. This is because the normal operating RPM of these fans is below
the standard threshold of 500 RPM and the motherboard will go into a critical state and ramp up the fans to maximum RPM.

You will see the following log lines in the ?????????? logs:

{{Picture}}

I use this approach with the Noctua NF-R8 redux-1800 PWM fans and X11 generation server motherboards. The fans are
connected to the FAN1 - FAN6 headers which are controlled by the CPU temperature (the FANA and FANB headers are not).
Below commands are tested on Debian Bullseye.

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

The lower critical threshold is set at 500. The Noctua fans I'm using have a maximum RPM of 1800 so they will fall under
this threshold when the motherboard sets their speed to 27% or less.

The fix is to create a small Bash scripts which sets the critical threshold to 100 which seems appropriate as a
failsafe. Create the file `/opt/fancontrol.sh` with the following content:

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

Set the permissions of the service file:

```bash
chmod 644 /etc/systemd/system/fancontrol.service
systemctl enable fancontrol.service
```

After a reboot (or manual execution of the script), you can check the new threshold limits:

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

The fans should now run smoothly while being temperature controlled by the Supermicro motherboard.
