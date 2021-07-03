---
layout: post
title: "Create a Debian Cloud-Init template on Proxmox"
tags: linux server
---

Proxmox templates together with Cloud-Init can be used to quickly deploy new VMs. A template quickly creates a new VM
and Cloud-Init will initialize the new VM so that you only have to set the host name and the initial user account.
No more installing the operating system from scratch for every new VM. In this guide, I'm describing how to do this with
Debian Buster to spin up headless Debian servers.

Debian doesn't provide a special image for this use case, but the Debian images designed for OpenStack come with
Cloud-Init support. Check out the [Proxmox's documentation](https://pve.proxmox.com/wiki/Cloud-Init_Support) for details
on how Proxmox's Cloud-Init support works.

Download `debian-10-openstack-amd64.qcow2` from <https://cloud.debian.org/images/cloud/OpenStack/current-10> to the
Proxmox host. Then execute the following steps:

```shell
qm create 900 --name debian-10-openstack-amd64 --net0 virtio,bridge=vmbr0
qm importdisk 900 debian-10-openstack-amd64.qcow2 local-lvm
qm set 900 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-900-disk-0
qm set 900 --ide2 local-lvm:cloudinit
qm set 900 --boot c --bootdisk scsi0
qm set 900 --serial0 socket --vga serial0
qm template 900
```

Here's the explanation for above steps:

1. Create a new VM with ID 900 using VirtIO networking drivers.
2. Import the qcow Debian image as a disk to the new VM. The disk will be called `local-lvm:vm-900-disk-0`.
3. Attach the imported disk as a VirtIO SCSI device to the VM.
4. Attach a drive for the Cloud-Init config to the VM.
5. Set the VM to boot from the imported disk image.
6. Add a serial console to the VM, which is needed by OpenStack.
7. Convert the VM into a template.

## Usage

To deploy a new server VM based on the template using the ID 101 and the name `VM-NAME`, execute the following command
on the Proxmox host:

```shell
qm clone 900 101 --name VM-NAME
```

More details about templates can be found in the [Proxmox wiki](https://pve.proxmox.com/wiki/VM_Templates_and_Clones).

After the new VM is created, you can finish the setup in the Proxmox web interface:

1. In the *Cloud-Init* tab of the VM, configure the name of the default user and the public SSH key you want to use for
authentication.
2. In the *Options* tab, enable the QEMU Guest Agent.
3. In the *Hardware* tab, select the `scsi0` hard disk and click *Resize disk*. The default size of the Debian image is
2 GiB. Specify the amount you want the disk to be increased by (e.g. 30 GiB for a total size of 32 GiB).

Everything is ready to go! Start the VM, run a system upgrade, and install the
[Qemu guest agent](https://pve.proxmox.com/wiki/Qemu-guest-agent):

```shell
sudo apt update
sudo apt full-upgrade
sudo apt install qemu-guest-agent
```
