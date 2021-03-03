---
layout: post
title: "Mount NFS Share directly in docker-compose Service"
tags: networking server linux docker
---

An NFS share can be directly mounted in a Docker container. This is a much cleaner way than mounting the NFS share on
the host first and then mounting the host directory in the Docker container.

To mount the `/path/to/video-dir` NFSv4 share from the `192.168.1.4` host as a `videos` volume:

```yaml
volumes:
  videos:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.1.4,nfsvers=4"
      device: ":/path/to/video-dir"
```

The driver-specific options you can use after the address in the `o` flag can be found in the [nfs manual page](https://man7.org/linux/man-pages/man5/nfs.5.html). Here are some examples of commonly used options:

* `nfsvers=3` or `nfsvers=4` to specify the NFS version
* `nolock` so that remote applications on the NFS host are not affected by lock files within the Docker container (only other processes within the container are affected by locks)

In addition, you can use `ro` to mount the share read-only or `rw` to mount the share explicitly as read-write.
