---
layout: post
title: "Install ARM64 Nvidia Drivers on Debian 12 Using Ansible"
description: "It's easy to install Nvidia drivers for a Debian 12 (Bookworm) or general Linux machine using a simple Ansible playbook that also works for ARM64 hosts."
tags: server linux nvidia gpu
---

I want to install the Nvidia driver and the Nvidia container toolkit on my new
[Ampere homelab server]({% post_url 2025-04-15-homelab-ampere-arm-server %}) which is running Debian 12 "Bookworm".
I'm using Ansible for the server setup so I wanted to use an Ansible role or playbook. There is the official
[Nvidia Ansible role](https://github.com/NVIDIA/ansible-role-nvidia-driver) but that does support neither Debian nor
ARM64 hosts.

There are generally 3 different ways to install Nvidia drivers:

1. Nvidia Apt repository: Provides the latest version but does not support ARM64
2. Nvidia `.run` driver: Provides the latest version but it's a binary that needs to be downloaded, so it's difficult to
keep up-to-date
3. Debian repository: Doesn't provide the latest version but it's easy to keep up-to-date with `apt`

In addition, I also want to install the [nvidia-container-toolkit](https://github.com/NVIDIA/nvidia-container-toolkit)
which is necessary to build and run GPU accelerated Docker containers. Luckily, the official Nvidia repository for the
container toolkit supports ARM64.

The following playbook shows how Nvidia drivers can be installed via Ansible on an ARM64 or x64 machine running Debian by
using the Nvidia drivers from the Debian repository (option 3) and the container toolkit from the Nvidia repositories.
I chose the Debian repository over the Nvidia `.run` driver because maintainability it more important to me than the
latest driver version. As of February 2025, the latest version for Linux was 550 but the Debian-provided version which
gets installed here is 535. From my experience, this should not matter for most homelab use cases.

```yaml
---
- hosts: all
  become: true

  tasks:
    - name: Add non-free repository to sources list
      ansible.builtin.apt_repository:
        repo: "deb http://deb.debian.org/debian/ {{ ansible_distribution_release }} main contrib non-free non-free-firmware"
        state: present
        filename: non-free

    - name: Add Nvidia container signing key
      ansible.builtin.apt_key:
        url: "https://nvidia.github.io/libnvidia-container/gpgkey"
        state: present

    - name: Add Nvidia container repository to sources list
      ansible.builtin.apt_repository:
        repo: "deb https://nvidia.github.io/libnvidia-container/stable/deb/$(ARCH) /"
        state: present
        filename: nvidia-container-toolkit

    - name: Install Nvidia drivers
      ansible.builtin.apt:
        update_cache: yes
        name:
          - firmware-misc-nonfree
          - nvidia-container-toolkit
          - nvidia-driver
          - nvtop
        state: latest
```

## Testing

You can test with 

`nvidia-smi`

or

`docker run -it --rm --gpus all debian nvidia-smi`

that the GPU is recognized correctly. For monitoring of GPU performance I recommend using [nvtop](https://github.com/Syllo/nvtop).

## Links

* <https://wiki.debian.org/NvidiaGraphicsDrivers#Installation>
