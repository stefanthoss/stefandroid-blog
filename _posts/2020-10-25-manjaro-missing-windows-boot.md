---
layout: post
title: "Dual Boot Windows entry disappears from GRUB (Manjaro)"
tags: linux
---

After a Manjaro Kernel upgrade the Windows 10 entry in GRUB disappears sometimes so that Windows can't be booted again.

This can easily be fixed with `update-grub`. This command will generate a new GRUB configuration file that will include
the GRUB entry for the Windows boot manager. The command has to be executed as root:

```
$ sudo update-grub
Generating grub configuration file ...
Found theme: /usr/share/grub/themes/manjaro/theme.txt
Found linux image: /boot/vmlinuz-5.8-x86_64
Found initrd image: /boot/intel-ucode.img /boot/initramfs-5.8-x86_64.img
Found initrd fallback image: /boot/initramfs-5.8-x86_64-fallback.img
Found Windows Boot Manager on /dev/nvme0n1p1@/efi/Microsoft/Boot/bootmgfw.efi
Adding boot menu entry for UEFI Firmware Settings ...
Found memtest86+ image: /boot/memtest86+/memtest.bin
done
```

After that you'll see the *Manjaro Linux* (with options for each installed kernel) and the *Windows Boot Manager*
entries in GRUB.
