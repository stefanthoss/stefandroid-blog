---
layout: post
title: "Encrypt a USB Drive with LUKS on Linux"
description: "Linux Unified Key Setup (LUKS) makes it easy to encrypt a removable USB drive on Linux and protect your data."
tags: linux encryption
---

It's important to back up your data. The backup is ideally stored on an encrypted drive so that nobody except for
yourself can access your data. This is especially important for removable USB drives because they can more easily get
lost or stolen. In this guide I use Linux Unified Key Setup (LUKS) for encrypting a hard drive (which can be an external
USB drive but also an internal drive). This is supported by pretty much any modern Linux system, so it's easy to take
your drive to a different computer and access the encrypted data.

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

In this example we will be using a 6 TB disk at `/dev/sdx` (make sure to replace `/dev/sdx` with your disk going
forward):

```text
Disk /dev/sdx: 5.46 TiB, 6001175126016 bytes, 11721045168 sectors
Disk model: 001-2BB186
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

## Setup

First we initialize a new LUKS partition on the disk. You will be asked twice for the encryption passphrase. **This will
delete all data on the disk.** With `--type luks2` we specify to use LUKS2, the newer implementation of LUKS. By
default, the `aes-xts-plain64` cipher with a 512 bit key is used.

```shell
sudo cryptsetup luksFormat --type luks2 /dev/sdx
```

After that we will open the newly created LUKS partition using the mapped device `backupDrive` (opening the LUKS
partition will ask you for the passphrase you just set). You can change the mapping name `backupDrive` to anything you
want for this disk. Use something unique in case you want to mount multiple LUKS encrypted partitions at the same time.

```shell
sudo cryptsetup luksOpen /dev/sdx backupDrive
```

The mapped block device will then be available at `/dev/mapper/backupDrive`. You can check the status of the mapped
device with

```shell
sudo cryptsetup -v status backupDrive
```

which will list information about the block device and the encryption cipher. Example output:

```text
/dev/mapper/backupDrive is active.
  type:    LUKS2
  cipher:  aes-xts-plain64
  keysize: 512 bits
  key location: keyring
  device:  /dev/sdx
  sector size:  512
  offset:  32768 sectors
  size:    11721012400 sectors
  mode:    read/write
Command successful.
```

The next step is to create a filesystem in the mapped block device `/dev/mapper/backupDrive`:

```shell
sudo mkfs -t ext4 -V /dev/mapper/backupDrive
```

The output will look something like this:

```text
mkfs from util-linux 2.36
mkfs.ext4 /dev/mapper/backupDrive
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 1465126550 4k blocks and 183144448 inodes
Filesystem UUID: 84326f68-6842-415e-a04b-7a3ec7e81893
Superblock backups stored on blocks:
        ...

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```

The device can now be mounted at `/mnt/backupDrive` and the disk usage can be listed with `df`:

```shell
sudo mount /dev/mapper/backupDrive /mnt/backupDrive
df -h
```

```text
Filesystem               Size  Used Avail Use% Mounted on
...
/dev/mapper/backupDrive  5.5T   89M  5.2T   1% /mnt/backupDrive
...
```

You can now copy data to `/mnt/backupDrive`. Don't forget to unmount before removing the drive. It's a good idea to
practice mounting and unmounting before using the drive with valuable data - if decrypting the LUKS partition doesn't
work the data is lost forever!

## Usage

Use the following commands to use the encrypted drive after the above setup is completed.

### Mounting

Open/decrypt the LUKS partition `/dev/sdx` and mount the block device `/dev/mapper/backupDrive`:

```shell
sudo cryptsetup luksOpen /dev/sdx backupDrive
sudo mount /dev/mapper/backupDrive /mnt/backupDrive
```

### Unmounting

Unmount the block device and close the LUKS partition:

```shell
sudo umount /mnt/backupDrive
sudo cryptsetup luksClose backupDrive
```
