---
layout: post
title: "Install Home Assistant OS on Proxmox"
description: "Home Assistant OS can be easily installed on Proxmox using the qcow2 virtual machine image."
tags: linux proxmox server
---

The Home Assistant Operating System (HAOS) has a couple of advantages over Home Assistant Container and Core. It
includes the Supervisor, [add-ons](https://www.home-assistant.io/addons), and backup functionality. Using the official
pre-built virtual machine image (built on top of Alpine Linux), you can get Home Assistant OS running on Proxmox in a
few minutes.

This guide assumes you have terminal access to the Proxmox host. This makes it easy to import the VM image.

First identify the release by going to the [Home Assistant OS release page on GitHub](https://github.com/home-assistant/operating-system/releases)
and find the latest release. You want to find the compressed `qcow2` image. As of October 2021, that's
[haos_ova-6.4.qcow2.xz](https://github.com/home-assistant/operating-system/releases/download/6.4/haos_ova-6.4.qcow2.xz).

Login to the Proxmox host, download the image, and decompress it:

```bash
wget https://github.com/home-assistant/operating-system/releases/download/6.4/haos_ova-6.4.qcow2.xz
xz -d haos_ova-6.4.qcow2.xz
```

I'm setting up Home Assistant OS with VM ID `101` and VM name `haos-vm` which you should change that as appropriate.
Execute the following steps to import the VM image:

```bash
qm create 101 --name haos-vm --net0 virtio,bridge=vmbr0 --bios ovmf --cores 2 --memory 4096 --agent enabled=1
qm importdisk 101 haos_ova-6.4.qcow2 local-lvm --format qcow2
qm set 101 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-101-disk-0
qm set 101 --boot c --bootdisk scsi0
pvesm alloc local-lvm 101 vm-101-disk-1 4M
qm set 101 -efidisk0 local-lvm:vm-101-disk-1
```

Here's the explanation for above steps:

1. Create the VM (with 2 CPU cores and 4096 MiB of memory).
2. Import the decompressed qcow2 image as a disk to the `local-lvm` storage.
3. Assign the imported disk to the VM.
4. Set the boot disk.
5. Allocate 4 MiB for the EFI disk.
6. Assign the EFI disk to the VM.

Now you can start the VM. Since we enabled the Use QEMU Guest Agent during VM creation, you can see the VM's IP address
in the Proxmox web UI. Go to `http://VM_IP:8123` in your browser and finish the setup.
