---
layout: post
title: "Bridge UniFi Switches with Wireless Uplink"
tags: networking server linux
---

If you have two areas of wired networking that need to be bridged with a wireless uplink, this can easily be done with Unifi. This
is a good solution if you live in an apartment where you can't run wires between rooms. The setup is quite straight-forward and I
was able to achieve speeds around 460 Mbit/s in both directions which is far from wired speeds but still pretty good for most use cases.

## Setup

Server (IP 192.168.10.1): pfSense 2.4.5 router with iPerf 3.7
Client: Debian 10 server with iPerf 3.6

I'm using [iPerf3](https://iperf.fr/) to measure connection speeds. For the tests I chose 30 seconds as the test doration and 4 parallel client streams since the pfSense hardware has 4 cores and I want to maximize throughput. 

## One Wall

### Client to Server (Client Upload Speed)

```bash
iperf3 -c 192.168.10.1 -f m -P 4 -t 30
```

Results:

```text
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-30.00  sec   413 MBytes   116 Mbits/sec    0             sender
[  5]   0.00-30.02  sec   412 MBytes   115 Mbits/sec                  receiver
[  7]   0.00-30.00  sec   344 MBytes  96.1 Mbits/sec    0             sender
[  7]   0.00-30.02  sec   342 MBytes  95.5 Mbits/sec                  receiver
[  9]   0.00-30.00  sec   504 MBytes   141 Mbits/sec    0             sender
[  9]   0.00-30.02  sec   502 MBytes   140 Mbits/sec                  receiver
[ 11]   0.00-30.00  sec   393 MBytes   110 Mbits/sec    3             sender
[ 11]   0.00-30.02  sec   390 MBytes   109 Mbits/sec                  receiver
[SUM]   0.00-30.00  sec  1.62 GBytes   463 Mbits/sec    3             sender
[SUM]   0.00-30.02  sec  1.61 GBytes   460 Mbits/sec                  receiver
```

### Server to Client (Client Download Speed)

```bash
iperf3 -c 192.168.10.1 -f m -P 4 -t 30 -R
```

Results:

```text
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-30.01  sec   355 MBytes  99.2 Mbits/sec  209             sender
[  5]   0.00-30.00  sec   354 MBytes  99.1 Mbits/sec                  receiver
[  7]   0.00-30.01  sec   501 MBytes   140 Mbits/sec  118             sender
[  7]   0.00-30.00  sec   500 MBytes   140 Mbits/sec                  receiver
[  9]   0.00-30.01  sec   438 MBytes   122 Mbits/sec  203             sender
[  9]   0.00-30.00  sec   437 MBytes   122 Mbits/sec                  receiver
[ 11]   0.00-30.01  sec   340 MBytes  95.2 Mbits/sec  224             sender
[ 11]   0.00-30.00  sec   340 MBytes  95.0 Mbits/sec                  receiver
[SUM]   0.00-30.01  sec  1.60 GBytes   457 Mbits/sec  754             sender
[SUM]   0.00-30.00  sec  1.59 GBytes   456 Mbits/sec                  receiver
```

## Many Walls

### Client to Server (Client Upload Speed)

```bash
iperf3 -c 192.168.10.1 -f m -P 4 -t 30
```

Results:

```text
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-30.00  sec   345 MBytes  96.6 Mbits/sec    0             sender
[  5]   0.00-30.02  sec   344 MBytes  96.1 Mbits/sec                  receiver
[  7]   0.00-30.00  sec   433 MBytes   121 Mbits/sec    0             sender
[  7]   0.00-30.02  sec   430 MBytes   120 Mbits/sec                  receiver
[  9]   0.00-30.00  sec   234 MBytes  65.6 Mbits/sec    0             sender
[  9]   0.00-30.02  sec   233 MBytes  65.2 Mbits/sec                  receiver
[ 11]   0.00-30.00  sec   309 MBytes  86.3 Mbits/sec    0             sender
[ 11]   0.00-30.02  sec   307 MBytes  85.9 Mbits/sec                  receiver
[SUM]   0.00-30.00  sec  1.29 GBytes   369 Mbits/sec    0             sender
[SUM]   0.00-30.02  sec  1.28 GBytes   367 Mbits/sec                  receiver
```

### Server to Client (Client Download Speed)

```bash
iperf3 -c 192.168.10.1 -f m -P 4 -t 30 -R
```

Results:

```text
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-30.01  sec   431 MBytes   121 Mbits/sec  375             sender
[  5]   0.00-30.00  sec   430 MBytes   120 Mbits/sec                  receiver
[  7]   0.00-30.01  sec   295 MBytes  82.5 Mbits/sec  455             sender
[  7]   0.00-30.00  sec   295 MBytes  82.3 Mbits/sec                  receiver
[  9]   0.00-30.01  sec   444 MBytes   124 Mbits/sec  348             sender
[  9]   0.00-30.00  sec   443 MBytes   124 Mbits/sec                  receiver
[ 11]   0.00-30.01  sec   294 MBytes  82.1 Mbits/sec  561             sender
[ 11]   0.00-30.00  sec   293 MBytes  81.9 Mbits/sec                  receiver
[SUM]   0.00-30.01  sec  1.43 GBytes   409 Mbits/sec  1739             sender
[SUM]   0.00-30.00  sec  1.43 GBytes   409 Mbits/sec                  receiver
```
