---
layout: post
title: "Install Nvidia Drivers on Debian 12 Using Ansible"
description: "It's easy to install Nvidia drivers for a Debian 12 or general Linux machine using a simple Ansible playbook that also works for ARM64 hosts."
tags: server linux nvidia gpu
---

I want to install the Nvidia driver and the Nvidia container toolkit on my new [Ampere homelab server]({% post_url 2025-02-06-ampere-server %}). I'm using Ansible for the setup so I wanted to use an Ansible role or playbook. There is the official https://github.com/NVIDIA/ansible-role-nvidia-driver Ansible playbook but that does support neither Debian nor ARM64 hosts.

This following playbook shows how Nvidia drivers can be installed via Ansible on an ARM64 or x86 machine running Debian by using the Nvidia drivers from the Debian repository and the container toolkit from the Nvidia repositories:

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

    - name: Install Nvidia and Google drivers
      ansible.builtin.apt:
        update_cache: yes
        name:
          - firmware-misc-nonfree
          - nvidia-container-toolkit
          - nvidia-driver
          - nvtop
        state: latest
```

Since this using the Debian instead of the Nvidia repositories for the driver, this does not install the latest version (as of Feb 2025, the latest version for Linux was 550 but the Debian-provided version which gets installed here is 535). From my experience, this should not matter for most homelab use cases.

You can test with 

`nvidia-smi`

or

`docker run -it --rm --gpus all debian nvidia-smi`

that the GPU is recognized correctly. For monitoring of GPU performance I recommend using `nvtop`.

## Links

* https://wiki.debian.org/NvidiaGraphicsDrivers#Installation
