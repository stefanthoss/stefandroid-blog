---
layout: post
title: "Encrypt a USB drive under Linux"
tags: linux encryption
---

For backups or storage of your personal data on a removable USB drive you should encrypt the drive so that nobody except for yourself can access that data. This is especially important for removable drives because they can more easily get lost or stolen. In this guide I use Linux Unified Key Setup (LUKS) for the disk encryption. This is supported by pretty much any modern Linux system so it's easy to take your drive to a different computer and access the encrypted data.

## Preparation

First, install the required `cryptsetup` software:

```shell
# Arch Linux / Manjaro
sudo pacman -Syu cryptsetup

# Debian
sudo apt install cryptsetup
```

Then identify the disk you want to encrypt using `fdisk`:

```shell
sudo fdisk -l
```

In this example we will be using the 6 TB disk at `/dev/sdb` (make sure to replace `/dev/sdb` with your disk going forward):

```
Disk /dev/sdb: 5.46 TiB, 6001175126016 bytes, 11721045168 sectors
Disk model: 001-2BB186
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

## Setup

First we initialize a new LUKS partition on the disk. This will ask you for the password that you want to set. After that we will open this new LUKS partition using the mapping `backupDrive` (opening the LUKS partition will ask you for the password you just set). This mapping name can be any identifier for this disk which is especially helpful if you want to mount multiple LUKS encrypted partitions at the same time.

```shell
sudo cryptsetup luksFormat --type luks2 /dev/sdb
sudo cryptsetup luksOpen /dev/sdb backupDrive
```
