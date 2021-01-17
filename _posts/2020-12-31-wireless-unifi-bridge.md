---
layout: post
title: "Bridge UniFi Switches with Wireless Uplink"
tags: networking server linux
---

If you have two areas of wired networking that need to be bridged with a wireless connection, this can easily be done with Unifi. This
is a good solution if you live in a place where you can't run wires between rooms. The setup is quite straight-forward and I
was able to achieve speeds around 460 Mbit/s in both upload and download. This is far from wired speeds but still
pretty good for most use cases.

## Setup

I have two UniFi switches that each have an access point connected:

![Wireless Uplink Bridge with UniFi](/assets/images/wireless-unifi-bridge.png)

I'm using a UniFi Switch 16 PoE and a UniFi Switch Lite 16 PoE and FlexHD / nanoHD access points.
This should work with any UniFi products though. Ubiquiti has a good article that explains how to
[configure a wireless uplink](https://help.ui.com/hc/en-us/articles/115002262328-UniFi-Configuring-a-Wireless-Uplink).
The Ubiquity guide explains this with the example of building a wireless mesh network.
What they don't explicitly mention is that you can connect additional switches and devices to the Ethernet port of the
uplinked access point.

*Note*: For this to work, first make sure that the "Enable wireless uplink" setting under "Services" in the "Site" section of the UniFi controller is enabled. Additionally, the "Enable Meshing" option in the "Radios" section of each access point should be
enabled as well.

Once the setup is completed, the access point with the wireless uplink should be listed as *Connected|Wireless* in the
UniFi Controller:

![Wireless Uplink in UniFi Controller](/assets/images/unifi-devices-wireless-connected.png)

## Speed Tests

I'm using [iPerf3](https://iperf.fr/) to measure connection speeds. For the tests I chose 30 seconds as the test duration and 4 parallel client streams since the pfSense hardware has 4 cores and I want to maximize throughput. I tested the throughput in two scenarios:
Only one wall between the two access points and multiple walls between the two access points.

In my setup I'm testing with the following two devices:

* Server: pfSense 2.4.5 router with iPerf 3.7 (IP 192.168.10.1)
* Client: Debian 10 server with iPerf 3.6

## One Wall

In the first test setup the two access points are roughly ??? feet apart and separated by one indoor wall. The connection
reaches 460 Mbit/s throughput for both upload and download. That is roughly half of what a wired Gigabit connection provides
but still enough for most home networking use cases and the average residential Internet connection. You probably don't
want to use this to connect your homelab servers with each other but as a connection to your modem it should be sufficient.

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

In the second test setup the two access points are roughly ??? feet apart and separated by multiple walls and a stair
case. The connection speeds drop to 370 Mbit/s for the client upload which is roughly 20% less. I expect the wireless
uplink connection to drop even more if you try to cover multiple floors.

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
