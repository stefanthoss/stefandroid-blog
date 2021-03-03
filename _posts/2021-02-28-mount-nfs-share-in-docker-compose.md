---
layout: post
title: "Mount NFS Share directly in docker-compose file"
tags: networking server linux docker
---

An NFS share can be directly mounted in a Docker container. This is a much cleaner way than mounting the NFS share on
the Docker host first and then mounting the host directory in the Docker container. With docker-compose, it is very easy
to configure an NFS mount.

To mount the `/path/to/video-dir` NFS share from the NFS server `192.168.1.4` as the named volume `videos`, add the
following to the volumes section of your docker-compose file:

```yaml
volumes:
  videos:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.1.4,nfsvers=4"
      device: ":/path/to/video-dir"
```

The `device` flag should contain the path to the share on the NFS server, note the `:` at the beginning. In the above
example the share can be mounted as the named volume `videos` in your docker-compose file.

The driver-specific options you can use after the address in the `o` flag can be found in the
[nfs manual page](https://man7.org/linux/man-pages/man5/nfs.5.html). Here are some examples of commonly used options:

* `nfsvers=3` or `nfsvers=4` to specify the NFS version
* `nolock` (optional): remote applications on the NFS server are not affected by lock files within the Docker container
(only other processes within the container are affected by locks)
* `timeo=n` (optional, default 600): the NFS client waits `n` tenths of a second before retrying an NFS request
* `soft,retrans=n`: the NFS client fails an NFS request after `n` unsuccessful retries, otherwise it will try
indefinitely

In addition, you can use `ro` to mount the share read-only or `rw` to mount the share explicitly as read-write.

An example for a complete docker-compose file would be to mount an NFSv4 share with video files read-only without
locking and limited retries in Jellyfin:

```yaml
version: "3"

services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    ports:
      - 8096:8096
    volumes:
      - ./config:/config
      - ./cache:/cache
      - videos:/mnt/videos

volumes:
  videos:
    driver_opts:
      type: "nfs"
      o: "addr=192.168.1.4,nolock,ro,soft,nfsvers=4"
      device: ":/path/to/video-dir"
```

Start the Docker service with `docker-compose up` and the Docker daemon will mount the NFS share
`192.168.1.4:/path/to/video-dir` at `/mnt/videos`.

If Docker can't mount the share, you'll see an error message like this:

```
ERROR: for jellyfin  Cannot start service jellyfin: error while mounting volume
'/var/lib/docker/volumes/jellyfin_videos/_data': failed to mount local volume:
mount :/path/to/video-dir:/var/lib/docker/volumes/jellyfin_videos/_data, flags: 0x1,
data: addr=192.168.1.4,nolock,soft,nfsvers=4: no route to host
```

Once successful, you can see the mount on the Docker host with `mount`:

```
:/path/to/video-dir on /var/lib/docker/volumes/jellyfin_videos/_data type nfs4
(ro,relatime,vers=4.0,rsize=131072,wsize=131072,namlen=255,hard,proto=tcp,timeo=600,
retrans=2,sec=sys,clientaddr=192.168.1.5,local_lock=none,addr=192.168.1.4)
```
