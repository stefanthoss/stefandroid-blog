---
layout: post
title: "Disable Annoying UPS Beeps with pfSense"
description: "You can disable the beeps that UPS units emit on battery power using pfSense."
tags: networking pfsense
---

I have my modem and pfSense router connected to an uninterruptible power supply (UPS) - great if you live in California with plenty of power outages. Most UPS units beep when the input power is disconnected. Assuming that you have enabled email notifications for the UPS (**Services** → **UPS** → **UPS Settings** → **General Settings** → **Enable E-Mail notifications**), it's not really necessary since you'll get an email and the beeping is just annoying. In this guide, I show how to disable the beeps using pfSense.

I assume that the UPS is connected via USB and is recognized by pfSense. In **Services** → **UPS** → **UPS Status** you should see a bunch of information about your UPS.

You need your UPS name which is defined in **Services** → **UPS** → **UPS Settings** in the field **UPS Name**. Then we need to get the admin password for the UPS. pfSense will automatically generate one, but you can't see it in the UPS section of the web UI. Go to **Diagnostics** → **Command Prompt** and use **Execute Shell Command** to execute the command `cat /usr/local/etc/nut/upsd.users`. It will print pfSense's UPS user configuration. Take note of the `password` in the `[admin]` section of the file. You'll need that to authenticate the `upscmd` command below.

Let's assume the UPS name is `router-ups` and the admin password is `ncf3c2op`. Execute the following three commands:

```shell
upsc router-ups ups.beeper.status
# Returns: enabled

upscmd -u admin -p ncf3c2op router-ups beeper.disable
# Returns: OK

upsc router-ups ups.beeper.status
# Returns: disabled
```

The beeps should now be disabled. If the above doesn't work, the command `upscmd -l router-ups` will list all available commands for this UPS. Check whether disabling beeps is a supported command for your UPS.
