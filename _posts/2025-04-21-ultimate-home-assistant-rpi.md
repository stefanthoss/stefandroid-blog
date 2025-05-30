---
layout: post
title: "Building the Ultimate Home Assistant Raspberry Pi"
description: "Building the Ultimate Home Assistant setup with a Raspberry Pi 5 and a Waveshare PoE M.2 HAT+."
tags: homeassistant rpi
---

I was replacing the machine that my Home Assistant instance is running on, and I wanted the replacement to be a Raspberry Pi with low power consumption and high reliability. I chose a Raspberry Pi 5 with a PoE/NVMe hat.

## Why?
 
I wanted to use a Raspberry Pi because

* I have good experience with those,
* they use little power for decent performance,
* they have a great ecosystem for accessories and online how-tos, and
* they are a first-class citizen for Home Assistant OS.

My research has shown the following three options:

Hardware | Price | CPU | Memory | Storage
--- | --- | --- | --- | ---
Home Assistant Green | $99 | quad 1.8 GHz | 4 GB | 32 GB eMMC
Home Assistant Yellow | $205 (*) | quad 1.5 GHz | 4 or 8 GB | 128 GB NVMe
RPi 5 | $105 (**) | quad 2.4 Ghz | 4 GB | 128 GB NVMe

(*) $135 for the Home Assistant Yellow with PoE, $50 for the RPi Compute Module 4, $20 for a 128 GB 2230 NVMe SSD

(**) $60 for the RPi 5, $25 for the Waveshare PoE M.2 HAT+, $20 for a 128 GB 2230 NVMe SSD

A Raspberry Pi 5 with a PoE/NVMe hat seemed like the best bang for the buck, so I went with that setup.

## Instructions

Flash "Raspberry Pi OS Lite (64-bit)" using the Raspberry Pi Flasher to an SD card. We need the OS briefly to configure the NVMe boot order and flash HAOS to the SSD.

Login to the RPi via SSH. First, check that the SSD got recognized using `lspci`. You're looking for a NVMe device like

```
0000:01:00.0 Non-Volatile memory controller: SK hynix Gold P31/PC711 NVMe Solid State Drive
```

The Waveshare HAT+ should be enabled by default, otherwise follow the [directions to enable the PCIe interface](https://www.waveshare.com/wiki/PoE_M.2_HAT%2B#Hard_disk_mounting). You can also check that the OS recognizes the disk with `lsblk`.

```shell
sudo apt update
sudo apt install rpi-imager
```

From the [HAOS release page](https://github.com/home-assistant/operating-system/releases), download the latest `haos_rpi5-64-{latest_version}.img.xz` release with `wget` and decompress it with `xz`, and copy it to the NVMe drive:

```shell
wget https://github.com/home-assistant/operating-system/releases/download/13.2/haos_rpi5-64-13.2.img.xz
xz -d haos_rpi5-64-13.2.img.xz
sudo dd bs=4M if=haos_rpi5-64-13.2.img of=/dev/nvme0n1
```

Not strictly required, but you can enable PCIe Gen3 speeds by adding `dtparam=pciex1_gen=3` to `/boot/firmware/config.txt`.

Now change the boot order with `sudo raspi-config`:
1. Choose "6 Advanced Options"
2. Choose "A4 Boot Order"
3. Choose "B2 NVMe/USB Boot Boot from NVMe before trying USB and then SD Card"

Reboot and enjoy your Home Assistant installation! If you migrate from a previous installation, you can restore the backup in the initial setup screens. It's also safe to remove the SD card now since it's no longer required.

## References

* Waveshare Hat wiki: https://www.waveshare.com/wiki/PoE_M.2_HAT%2B
* Jeff Geerling's article: https://www.jeffgeerling.com/blog/2023/nvme-ssd-boot-raspberry-pi-5
* Home Assistant on RPi guide: https://www.home-assistant.io/installation/raspberrypi
