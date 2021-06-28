---
layout: post
title: "Create a Debian Cloud-Init template on Proxmox"
tags: linux server
---

See https://pve.proxmox.com/wiki/Cloud-Init_Support

## Preparation

Download `debian-10-openstack-amd64.qcow2` from https://cloud.debian.org/images/cloud/OpenStack/current-10

```shell
qm create 900 --name debian-10-openstack-amd64 --net0 virtio,bridge=vmbr0
qm importdisk 900 debian-10-openstack-amd64.qcow2 local-lvm
qm set 900 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-900-disk-0
qm set 900 --ide2 local-lvm:cloudinit
qm set 900 --boot c --bootdisk scsi0
qm set 900 --serial0 socket --vga serial0
qm template 900
```

## Usage

https://pve.proxmox.com/wiki/VM_Templates_and_Clones

To create a new VM with the ID 104:

```shell
qm clone 900 104 --name <vm-name>
```

In the Cloud-Init tab of the VM, add your username and public SSH key.

In the Options tab of the VM, enable the QEMU Guest Agent.

```shell
sudo apt update
sudo apt full-upgrade
sudo apt install qemu-guest-agent
```

Now resize the disk as https://pve.proxmox.com/wiki/Resize_disks. Through the Proxmox UI, go to the *Hardware* tab of
the VM, select the hard disk (by default 2 GiB), and click the *Resize disk*. Select the size increment and resize the disk.
