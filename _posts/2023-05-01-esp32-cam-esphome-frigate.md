---
layout: post
title: "Use a ESP32-CAM with ESPHome in Frigate"
description: "t.b.d."
tags: homeassistant esp32
---


Roughly $6 (e.g. <https://www.aliexpress.us/item/3256803148159834.html>)

This is actually this cam: https://docs.ai-thinker.com/en/esp32-cam which has good documentation including [this schematic diagram](https://docs.ai-thinker.com/_media/esp32/docs/esp32_cam_sch.pdf). It's a ESP32-S module with an OV2640 camera module and a USB serial interface. It also supports an external WiFi antenna.


https://esphome.io/components/esp32_camera.html
https://esphome.io/components/esp32_camera_web_server.html

## Installation

If you can't see the device but instead see the following lines in `dmesg` that's because it's claimed by the BRLTTY service.

```text
[  277.336833] usb 1-2: new full-speed USB device number 9 using xhci_hcd
[  277.484640] usb 1-2: New USB device found, idVendor=1a86, idProduct=7523, bcdDevice= 2.54
[  277.484657] usb 1-2: New USB device strings: Mfr=0, Product=2, SerialNumber=0
[  277.484664] usb 1-2: Product: USB2.0-Ser!
[  277.487262] ch341 1-2:1.0: ch341-uart converter detected
[  277.487945] ch341-uart ttyUSB0: break control not supported, using simulated break
[  277.488187] usb 1-2: ch341-uart converter now attached to ttyUSB0
[  278.075267] input: BRLTTY 6.4 Linux Screen Driver Keyboard as /devices/virtual/input/input20
[  278.192497] usb 1-2: usbfs: interface 0 claimed by ch341 while 'brltty' sets config #1
[  278.192924] ch341-uart ttyUSB0: ch341-uart converter now disconnected from ttyUSB0
[  278.192962] ch341 1-2:1.0: device disconnected
```

In case you don't need the BRLTTY service, you can disable the udev rules with

```shell
for f in /usr/lib/udev/rules.d/*brltty*.rules; do
    sudo ln -s /dev/null "/etc/udev/rules.d/$(basename "$f")"
done
sudo udevadm control --reload-rules
```

as suggested by [this StackExchange answer](https://unix.stackexchange.com/a/670637).

