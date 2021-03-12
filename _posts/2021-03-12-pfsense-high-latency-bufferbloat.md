---
layout: post
title: "Fix high latency when maxing out upload bandwidth on pfSense"
tags: networking linux pfsense
---

If you max out the upload bandwidth that your Internet connection provides, you might experience degraded performance on
your pfSense router. You will see high latency/ping (RTT) and high packet loss. The clients accessing the Internet will
experience slow-loading web pages, distorted video/voice calls, and unresponsive behavior. This is known as
[Bufferbloat](https://www.bufferbloat.net/projects/bloat/wiki/Introduction/) and is basically traffic piling up on the
router due to the Internet upload bandwith being limiting to outgoing traffic.

Here's a screenshot of my pfSense dashboard when I max out the 20 Mbit/s upload of my Cable Internet connection, showing
a RTT of over 100ms and 14% packet loss:

![pfSense WAN gateway with high latency and packetloss](/assets/images/pfsense-wan-gateway-packetloss.png)

In pfSense's **Status** → **System Logs** → **Gateways** you will see logs like this:

```text
Time             Process   PID     Message
Mar 5 10:54:27   dpinger   38533   WAN_DHCP: Clear latency 210739us stddev 122030us loss 16%
Mar 5 10:54:15   dpinger   38533   WAN_DHCP: Alarm latency 210585us stddev 121638us loss 21%
Mar 5 10:44:04   dpinger   38533   WAN_DHCP: Clear latency 182806us stddev 113934us loss 18%
Mar 5 10:43:27   dpinger   38533   WAN_DHCP: Alarm latency 194447us stddev 107034us loss 21%
```

The [DSL Reports Speed Test](http://dslreports.com/speedtest) is an easy test to measure bufferbloat.

## Solution

Using the [pfSense Traffic Shaper](https://docs.netgate.com/pfsense/en/latest/trafficshaper/index.html) you can setup
Controlled Delay (CoDel) queue management. This will help the traffic to flow smoother and without spikes in latency
and packet loss. The pfSense documentation provides
[more details on CoDel Active Queue Management](https://docs.netgate.com/pfsense/en/latest/trafficshaper/altq-scheduler-types.html#codel-active-queue-management).

Netgate uploaded the slide deck [pfSense Hangout August 2018](https://www.slideshare.net/NetgateUSA/pfsense-244-short-topic-miscellany-pfsense-hangout-august-2018)
which describes the CoDel limiter setup on slide 5 to 11. I provide a summary of that in the following section.

## CoDel Setup

The following description is for a pfSense 2.5.0 firewall using an IPv4 WAN gateway. If you have an IPv6 WAN gateway,
you'll have to take additional steps that I'm not documenting.

Go to **Firewall** → **Traffic Shaper** → **Limiters** → **New Limiter** and add the following limiter:

| Enable | [x] Enable |
| Name | WanDownload |
| Bandwith | ?? Mbit/s, Schedule: none |
| Mask | None |
| Queue Management Algorithm | CoDel |
| Scheduler | FQ_CODEL |
| Queue length | 1000 |
| ECN | [x] Enable |

Adapt the bandwith to the download bandwith of your Internet connection. Leave everything else and the Advanced Options
empty. After you saved the limiter, click **Add new Queue** and enter the following configuration:

| Enable | [x] Enable |
| Name | DownloadQueue |
| Mask | None |
| Queue Management Algorithm | CoDel |
| ECN | [x] Enable |

Leave everything else and the Advanced Options empty. Save the queue.

Repeat the above limiter and queue creations for the upload - creating a **WanUpload** limiter with an **UploadQueue**
queue. Use the upload bandwith of your Internet connection for the upload limiter. The Traffic Shaper Limiter page
should look like this:

![pfSense Traffic Shaper Limiters](/assets/images/pfsense-limiter-queue.png)

Now go to **Firewall** → **Rules** → **Floating** → **Add** and add a floating rule with the following configuration:

| Action | Pass |
| Quick | [x] Apply the action immediately on match. |
| Interface | WAN |
| Direction | out |
| Address Family | IPv4 |
| Protocol | Any |
| Source | any |
| Destination | any |
| Description | CoDel Limiters |
| Advanced Options | Display Advanced |
| Gateway | WAN_DHCP - Interface WAN_DHCP Gateway |
| In / Out pipe | UploadQueue / DownloadQueue |

Save the rule and apply changes. You're done and shouldn't experience bufferbloat anymore!

## Limiter Bandwidth Testing

My Cable Internet connection is marketed as 400 Mbit/s download and 20 Mbit/s upload. Based on my speed tests it is in
reality closer to 420 Mbit/s download and 22 MBit/s upload (I know - very surprising!). I want to find out how I should
configure the limiter's upload bandwidth - slightly above or slightly below the actual upload throughput? I'll be
focussing on the upload only since I'm not experiencing noticeable bufferbloat with my download connection.

First I run a test with a 25 Mbit/s **WanUpload** limiter (slightly above the actual upload throughput):

![pfSense WAN gateway with high latency but no packetloss](/assets/images/pfsense-wan-gateway-high-latency.png)

I'm still experiencing high latency but no packet loss anymore. It seems that the limiter is preventing the packet loss
but it can't do anything about the latency - the Internet connection is still too slow for the amount of data I'm trying
to push.

Now I run a test with a 20 Mbit/s **WanUpload** limiter (slightly below the actual upload throughput):

![pfSense WAN gateway without latency and packetloss](/assets/images/pfsense-wan-gateway-low-latency.png)

This looks much better. I might leave a bit of upload throughput on the table, but I finally see low latency and no
packet loss even under maximum upload stress. You should definitely set your limiter bandwidth slightly below your
actual upload throughput to avoid both high latency and packet loss.
