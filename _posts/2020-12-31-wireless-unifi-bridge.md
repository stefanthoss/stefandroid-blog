---
layout: post
title: "Bridge UniFi Switches with Wireless Uplink"
tags: networking server linux
---

You can use UniFi networking gear to bridge two areas of wired networking with a wireless connection. This is a good
solution if you live in a home where you can't run wires between rooms. The setup is relatively straight-forward and I
was able to achieve throughputs of 360 to 460 Mbit/s for both upload and download. This is far from wired throughput
but still pretty good for a lot of use cases.

## Setup

I have two UniFi switches that each have an access point connected:

![Wireless Uplink Bridge with UniFi](/assets/images/wireless-unifi-bridge.png)

I'm using

* a UniFi Switch 16 PoE and a UniFi Switch Lite 16 PoE,
* a UniFi FlexHD and a UniFi nanoHD, and
* a UniFi Controller 6.0.

This should work with any other currently supported UniFi networking gear. Ubiquiti has a good guide that explains how
to [configure a wireless uplink](https://help.ui.com/hc/en-us/articles/115002262328-UniFi-Configuring-a-Wireless-Uplink).
The guide explains this with the example of building a wireless mesh network. It does not explicitly mention that you
can connect additional switches and devices to the Ethernet port of the uplinked access point but this definitely works.

*Note*: For this to work, make sure that the "Enable wireless uplink" setting under "Services" in the "Site" section of
the UniFi controller is enabled. Additionally, the "Enable Meshing" option in the "Radios" section of each access point
should be enabled as well.

To start the setup, move the switch and access point that is not connected to the UniFi Controller to the location of
the other switch and connect the two switches with a wired Ethernet connection. Make sure that all devices are adopted
in the UniFi Controller. Then disconnect the cable between the switches. After some time, the access point with the
wireless uplink should be listed as *Connected|Wireless* in the UniFi Controller:

![Wireless Uplink in UniFi Controller](/assets/images/unifi-devices-wireless-connected.png)

## Speed Tests

I'm using [iPerf3](https://iperf.fr/) to measure network throughput. For the tests I chose 30 seconds as the test
duration and 4 parallel client streams since my server has 4 cores and I want to maximize throughput. I tested the
throughput in two scenarios, once with a single wall between the two access points and once with more obstruction.

I'm testing with the following two devices:

* Server: pfSense 2.4.5 router with iPerf 3.7 (IP 192.168.10.1)
* Client: Debian 10 server with iPerf 3.6

## One Wall

In the first scenario the two access points are roughly 40 ft / 12 m apart, on the same floor, and separated by one
interior (wood frame) wall. The throughput is roughly 460 Mbit/s for both upload and download. That is roughly half of
what a wired Gigabit connection provides but still enough for most home networking use cases and the average residential
Internet connection. You probably don't want to use this to connect your homelab servers to each other but as a
connection to your modem it should be sufficient.

### One Wall - Client Upload Speed

```bash
iperf3 -c 192.168.10.1 -f m -P 4 -t 30
```

Result:

```text
[ ID] Interval           Transfer     Bitrate         Retr
[SUM]   0.00-30.00  sec  1.62 GBytes   463 Mbits/sec    3             sender
[SUM]   0.00-30.02  sec  1.61 GBytes   460 Mbits/sec                  receiver
```

### One Wall - Client Download Speed

```bash
iperf3 -c 192.168.10.1 -f m -P 4 -t 30 -R
```

Result:

```text
[ ID] Interval           Transfer     Bitrate         Retr
[SUM]   0.00-30.01  sec  1.60 GBytes   457 Mbits/sec  754             sender
[SUM]   0.00-30.00  sec  1.59 GBytes   456 Mbits/sec                  receiver
```

## More Obstruction

In the second scenario the two access points are roughly 30 ft / 9 m apart, on the same floor, and separated by 3
interior (wood frame) walls and a staircase. The throughput drops by roughly 20% to 365 Mbit/s. As expected,
obstructions like walls have major impact on the throughput. If you can arrange the access points in a way to minimize
the number of walls in between, your throughput will likely benefit from it. I assume that multiple floor or concrete
walls have an even greater impact.

### More Obstruction - Client Upload Speed

```bash
iperf3 -c 192.168.10.1 -f m -P 4 -t 30
```

Result:

```text
[ ID] Interval           Transfer     Bitrate         Retr
[SUM]   0.00-30.00  sec  1.29 GBytes   369 Mbits/sec    0             sender
[SUM]   0.00-30.02  sec  1.28 GBytes   367 Mbits/sec                  receiver
```

### More Obstruction - Client Download Speed

```bash
iperf3 -c 192.168.10.1 -f m -P 4 -t 30 -R
```

Result:

```text
[ ID] Interval           Transfer     Bitrate         Retr
[SUM]   0.00-30.01  sec  1.26 GBytes   361 Mbits/sec  4026             sender
[SUM]   0.00-30.00  sec  1.26 GBytes   360 Mbits/sec                  receiver
```
