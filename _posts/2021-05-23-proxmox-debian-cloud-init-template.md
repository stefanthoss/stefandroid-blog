---
layout: post
title: "Create a Debian Cloud-Init template on Proxmox"
tags: linux server
---

Proxmox templates can be used to quickly deploy new VMs. Cloud-Init will initialize the new VM so that you only have to
configure networking, host name, and the initial user account for new VMs instead of installing the operating system
from scratch. In this guide I'm describing how to do this with Debian Buster. Check out the
[Proxmox's documentation](https://pve.proxmox.com/wiki/Cloud-Init_Support) for details on how Proxmox's
Cloud-Init support works.

The Debian images designed for OpenStack come with Cloud-Init support. Download `debian-10-openstack-amd64.qcow2`
from <https://cloud.debian.org/images/cloud/OpenStack/current-10> to the Proxmox host. These are the steps:

```shell
qm create 900 --name debian-10-openstack-amd64 --net0 virtio,bridge=vmbr0
qm importdisk 900 debian-10-openstack-amd64.qcow2 local-lvm
qm set 900 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-900-disk-0
qm set 900 --ide2 local-lvm:cloudinit
qm set 900 --boot c --bootdisk scsi0
qm set 900 --serial0 socket --vga serial0
qm template 900
```

Explanation for above steps:

1. Create a new VM with ID 900 using VirtIO networking drivers.
2. Import the qcow Debian image as a disk to the new VM. The disk will be called `local-lvm:vm-900-disk-0`.
3. Attach the imported disk as a VirtIO SCSI device to the VM.
4. Attach a drive for the Cloud-Init config to the VM.
5. Set the VM to boot from the imported disk image.
6. Add a serial console to the VM which is needed by OpenStack.
7. Convert the VM into a template.

## Usage

To deploy a new VM based on the template with ID 101 and name `VM-NAME`, execute the following command on the Proxmox
host:

```shell
qm clone 900 101 --name VM-NAME
```

For more details on this check out [Proxmox's template documentation](https://pve.proxmox.com/wiki/VM_Templates_and_Clones).

After the new VM is created, you can finish the configuration in the Proxmox web interface. In the *Cloud-Init* tab of
the VM, configure the name of the default user and the public SSH key you want to use for authentication. In the
*Options* tab, enable the QEMU Guest Agent. In the *Hardware* tab, select the `scsi0` hard disk and click *Resize disk*.
The default size is 2 GiB - add the size increment you want the disk to be increased by.

Everything is ready to go! Start the VM, run a system upgrade, and install the
[Qemu guest agent](https://pve.proxmox.com/wiki/Qemu-guest-agent):

```shell
sudo apt update
sudo apt full-upgrade
sudo apt install qemu-guest-agent
```
