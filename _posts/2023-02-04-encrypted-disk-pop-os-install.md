---
layout: post
title: "Use Disk Encryption with Pop_OS! and Custom Partitioning"
description: "With LUKS and LVM, you can set up disk encryption and custom partitioning for a new Pop_OS! installation."
tags: linux encryption
---

Pop_OS! does not offer the option to setup disk encryption when using custom partitioning during the installation process. You have to choose between using the entire disk with encryption or using custom partitioning without encryption. Custom partitioning allows for advanced setups like dual booting, separate `/home` partitions, or installations that span multiple disks. With a few command line tools, it's however possible to install Pop_OS! with disk encryption and custom partitioning. You should defintely use disk encryption so make sure to follow these steps.

This applies to Pop!_OS 22.04 LTS as of February 2023, I hope that they add custom encryption capabilities to their installation wizard in the future.

The goal is to setup a LUKS-encrypted partition with an LVM volume. You can install Pop_OS! into an existing LVM volume within a LUKS partition. All of this can be done within Pop_OS! when booted from the installation medium. Check out my post about [encrypting a USB drive with LUKS]({% post_url 2021-02-08-encrypt-usb-drive-with-luks %}) for more details about how to use LUKS encryption. Please make sure you have a backup of your files before doing any of these, there is no recovering lost data.

First, create an empty partition where you want to setup encryption and install Pop_OS! In the following examples, that will be `/dev/sdx`. You perform the partitioning with GParted and check the partitions with `sudo fdisk -l`.

Next, format the partition with LUKS, open it (I use `crypt_sdx` as the mapping name), and initialize it for use by LVM. During these steps, you will be asked to set the encryption password -- use a strong one and memorize it well!

```shell
sudo cryptsetup luksFormat --type luks2 /dev/sdx
sudo cryptsetup luksOpen /dev/sdx crypt_sdx
sudo pvcreate /dev/mapper/crypt_sdx
```

List all LVM physical volumes with `pvs` and check that it got created correctly:

```text
$ sudo pvs
  PV                    VG Fmt  Attr PSize   PFree
  /dev/mapper/crypt_sdx    lvm2 ---  240.00g 240.00g
```

Create a new LVM colum group which I call `vge01`:

```shell
sudo vgcreate vge01 /dev/mapper/crypt_sdx
```

List all LVM volume groups with `vgdisplay` (you can also use `vgs` for a shorter form) and check that it got created correctly:

```text
$ sudo vgdisplay
  --- Volume group ---
  VG Name               vge01
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               240.00 GiB
  PE Size               4.00 MiB
  Total PE              61488
  Alloc PE / Size       0 / 0
  Free  PE / Size       61488 / 240.00 GiB
  VG UUID               XXXXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXXXX
```

Last step is to create the logical volume which I call `lv00`. The size has to be specified with the `-L` option which is 240 GB for my case (see "VG Size" above).

```shell
sudo lvcreate -n lv00 -L 240G vge01
```

List all logical volumes with `lvs` and check that it got created correctly:

```text
$ sudo lvs
  LV   VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv00 vge01 -wi-a----- 240.00g
```

Now you're ready for the installation of Pop_OS! (see [official installation guide](https://support.system76.com/articles/install-pop)). In the installation wizard, select `Custom (Advanced)`. Now select the `/dev/sdx` partition we created and it will ask you for the encryption password. Select the logical volume within the encrypted partition as the destination for the OS installation. Finish the installation as usual.

You're done! Pop_OS! will ask you for your disk password on every boot.

Once booted, you can see with `mount` that the mapped encrypted volume `/dev/mapper/vge01-lv00` is used, rather than the `/dev/sdx` partition directly:

```text
/dev/mapper/vge01-lv00 on / type ext4 (rw,noatime,errors=remount-ro)
```
