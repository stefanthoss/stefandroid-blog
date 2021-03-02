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
