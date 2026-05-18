---
layout: post
title: "Pairing a Thread Device Without a Phone Directly in Home Assistant"
description: "A guide to pairing Matter-over-Thread devices with Home Assistant, bypassing the common flow using an Android phone or an iPhone."
tags: homeassistant rpi thread matter
---

The standard way of pairing a new Matter-over-Thread device with your Home Assistant installation is via your
Android or iPhone.
Unfortunately this pairing process has often failed for me and got stuck with a "Checking connectivity to Thread network"
message no matter whether I use the Android or iPhone companion app. Luckily there is another way: You can pair new
Thread devices directly through the Home Assistant server.

First the requirements: Your Home Assistant hardware has to have a Bluetooth adapter (or use built-in Bluetooth if
[you're using a device like a Raspberry Pi]({% post_url 2025-04-21-ultimate-home-assistant-rpi %})).
And you have to move the Thread device into Bluetooth range of the hardware since Bluetooth is used for the initial pairing.

To start, make sure that the Matter Server and OpenThread Border Router apps are installed. Then install a Bluetooth
dongle or use the integrated Bluetooth functionality of your Raspberry Pi. Set up the Bluetooth integration in Home
Assistant and in Settings navigate to the Bluetooth integration. You should see the Bluetooth adapter with an
identifier like `hci0` or `hci1`. The number behind the letters is your Bluetooth adapter ID.

Now go to *Settings* -> *Apps* -> *Matter Server* -> *Configuration* -> *Show unused optional configuration
options* -> *Bluetooth Adapter ID*. Enter the number from above. Restart the Matter Server app.

This concludes the initial setup. For every new device you'll have to follow the following steps.

Go to *Settings* -> *Thread* and click on the "i" (Thread network information) in the top right corner of your
preferred Thread network. You'll see a very long alphanumeric string after *Active dataset TLVs*. Copy that string.
It'll never change but I noticed that the Matter Server app of Home Assistant sometimes forgets it so you might
need to copy it again.

Bring the Thread device into Bluetooth range of your Home Assistant server. Factory reset the Thread device or
enter the pairing mode (depends on the specific device).

Now go to the Matter Server app which should be in the sidebar on the left of your Home Assistant dashboard. Click
*Commission node* -> *Commission new Thread device* and enter the active dataset TLV you copied earlier. In the
next step you need to enter your pairing code. The pairing code is a number of three groups of digits that's
usually underneath the Matter pairing QR code (the one you'd scan with the mobile app). Enter this number without
the dashes as your pairing code.

Now wait and the device will be paired! I found this a lot easier than haggling with Google's Android
implementation of the Thread pairing which is used by the Home Assistant Android app. It's also faster, usually it
takes only a few seconds.
