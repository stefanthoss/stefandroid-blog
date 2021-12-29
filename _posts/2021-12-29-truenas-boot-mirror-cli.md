---
layout: post
title: "Adding TrueNAS Boot Mirror Disk via CLI"
description: "You can configure a mirrored boot pool for your TrueNAS system using the CLI."
tags: truenas server
---

Adding a second disk to the TrueNAS boot pool will increase resilience of the TrueNAS installation in case the original
boot device fails - by creating a ZFS mirror (RAID1). This can be easily configured via the web UI or via the CLI. The
way of using the command line interface (CLI) is not well documented, so I documented it here.

*Note*: Make a backup of your TrueNAS configuration before you do any of this - if things go wrong, it will break your
TrueNAS installation.

As described in the [TrueNAS documentation about mirroring the boot pool](https://www.truenas.com/docs/core/system/boot/bootpoolmirror/),
you can configure a boot pool mirror in the web UI by selecting a second disk at **System** → **Boot** → **Actions** →
**Boot Pool Status** → **Attach**. But I've frequently gotten the error message `Error: [EFAULT] None` when attempting
this, and I couldn't figure out why or how to fix it. It worked however with the CLI.

All the following commands should be executed with root privileges, e.g. via the web UI **Shell**. First check the
current boot pool with the command `zpool status boot-pool` which should output something like this:

```text
  pool: boot-pool
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        boot-pool   ONLINE       0     0     0
          ada0p2    ONLINE       0     0     0

errors: No known data
```

Adding a mirror disk is a two-step process. Since TrueNAS ZFS mirrors are based on partitions and not entire disks, we
first have to create the correct partition table on the new disk. In the following, I want to add the `ada1` disk as a
mirror to the existing `ada0` disk. Clone the partition table from `ada0` to `ada1` with `gpart` (effectively backing up
the `ada0` partition table and restoring it on `ada1`):

```shell
gpart backup ada0 | gpart restore -F ada1
```

You can then double-check the partition table using `gpart show ada1`. A TrueNAS 12 system has a bootloader partition
(partition 1), a swap partition (partition 3), and the main file system partition (partition 2). Output for a 64 GB boot
disk should look like this:

```text
=>       40  125045344  ada0  GPT  (60G)
         40       1024     1  freebsd-boot  (512K)
       1064   33554432     3  freebsd-swap  (16G)
   33555496   91488256     2  freebsd-zfs  (44G)
  125043752       1632        - free -  (816K)
```

Now we can add the second partition (`ada1p2`) of the newly formatted disk to the boot pool using `zpool attach` (the
usage is `zpool attach pool device new_device`). Execute the following command:

```shell
zpool attach boot-pool /dev/ada0p2 /dev/ada1p2
```

Now check the boot pool with `zpool status boot-pool`. You should see that the pool is currently being resilvered with
the status looking like this:

```text
  pool: boot-pool
 state: ONLINE
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
config:

        NAME        STATE     READ WRITE CKSUM
        boot-pool   ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p2  ONLINE       0     0     0
            ada1p2  ONLINE       0     0     0  (resilvering)

errors: No known data errors
```

Depending on the size of your boot pool, it might take a few minutes until the resilvering is complete. After that, all
data will be copied from your original boot disk to the new boot disk. You now have a mirrored boot disk (RAID1)!
