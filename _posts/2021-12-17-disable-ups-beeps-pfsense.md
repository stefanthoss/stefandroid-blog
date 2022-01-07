---
layout: post
title: "Disable Annoying UPS Beeps with pfSense"
description: "You can disable the beeps that UPS units emit on battery power using pfSense."
tags: pfsense ups
---

I have my modem and pfSense router connected to an uninterruptible power supply (UPS) - useful if you live in California
with plenty of power outages. Most UPS units beep when they are on battery power during power outages. Assuming that you
have enabled email notifications for the UPS (**Services** → **UPS** → **UPS Settings** → **General Settings** →
**Enable E-Mail notifications**), it is not really necessary since you will receive an email and the beeping is just
annoying. In this guide, I show how to disable the beeps using pfSense.

I assume that the UPS is connected via USB and is recognized by pfSense. In **Services** → **UPS** → **UPS Status** you
should see a bunch of information about the UPS if that is the case.

You need the UPS name which is defined in **Services** → **UPS** → **UPS Settings** → **UPS Name**. Then you need to get
the admin password for the UPS. pfSense will automatically generate one, but you can't see it in the UPS section of the
web UI. Go to **Diagnostics** → **Command Prompt** and use **Execute Shell Command** to execute the command
`cat /usr/local/etc/nut/upsd.users`. It will print the UPS user configuration. Take note of the password in the
`[admin]` section of the file. You will need that to authenticate the `upscmd` command below.

Let's assume the UPS name is `router-ups` and the admin password is `ncf3c2op`. Execute the following three commands
using the pfSense **Execute Shell Command** functionality:

```shell
upsc router-ups ups.beeper.status
# Returns: enabled

upscmd -u admin -p ncf3c2op router-ups beeper.disable
# Returns: OK

upsc router-ups ups.beeper.status
# Returns: disabled
```

The beeps should now be disabled. If the above doesn't work, the command `upscmd -l router-ups` will list all available
commands for this UPS. Check whether disabling beeps is a supported command for your UPS.
