---
layout: post
title: "Fix pfSense high latency during high upload throughput"
tags: networking linux
---

If you max out the upload that your Internet connection provides, you might experience degraded performance on your
pfSense router. You will see high latency/ping (RTT) and high packet loss on the pfSense dashboard and your clients will
experience slow-loading web pages, distorted video/voice calls, and unresponsive behavior. This is known as
[Bufferbloat](https://www.bufferbloat.net/projects/bloat/wiki/Introduction/) and is basically traffic piling up on the
router due to the Internet upload bandwith being limiting to outgoing traffic.

Here's a screenshot of my pfSense dashboard when I max out the 20 Mbit/s upload of my Cable Internet connection, showing
a RTT of over 100ms and 14% packet loss:

![pfSense dashboard with high latency and packetloss](/assets/images/pfsense-high-latency-packetloss.png)

In pfSense's **Status** → **System Logs** → **Gateways** you will see logs like this:

```text
Time             Process   PID     Message
Mar 5 10:54:27   dpinger   38533   WAN_DHCP: Clear latency 210739us stddev 122030us loss 16%
Mar 5 10:54:15   dpinger   38533   WAN_DHCP: Alarm latency 210585us stddev 121638us loss 21%
Mar 5 10:44:04   dpinger   38533   WAN_DHCP: Clear latency 182806us stddev 113934us loss 18%
Mar 5 10:43:27   dpinger   38533   WAN_DHCP: Alarm latency 194447us stddev 107034us loss 21%
```

## Solution

Using [pfSense Traffic Shaper](https://docs.netgate.com/pfsense/en/latest/trafficshaper/index.html) you can setup
Controlled Delay (CoDel) queue management. This will help the traffic to flow smoother and without spikes in latency
and packet loss. The pfSense documentation provides [more details on CoDel](https://docs.netgate.com/pfsense/en/latest/trafficshaper/altq-scheduler-types.html#codel-active-queue-management). The [DSL Reports Speed Test](http://dslreports.com/speedtest)
is an easy test to measure bufferbloat.

Netgate uploaded a slide deck [pfSense Hangout August 2018](https://www.slideshare.net/NetgateUSA/pfsense-244-short-topic-miscellany-pfsense-hangout-august-2018) which describes the setup for CoDel limiters on slide 5
to 11 in more detail.

## Setup

The following description is for a pfSense 2.5.0 firewall using an IPv4 WAN gateway. If you have an IPv6 WAN gateway,
you'll have to take additional steps.

In pfSense, go to **Firewall** → **Traffic Shaper** → **Limiters** → **New Limiter**.

| Enable | [x] Enable |
| Name | WanDownload |
| Bandwith | 400 Mbit/s, Schedule: none |
| Mask | None |
| Queue Management Algorithm | CoDel |
| Scheduler | FQ_CODEL |
| Queue length | 1000 |
| ECN | [x] Enable |

Adapt the bandwith to the download bandwith of your Internet connection. Leave everything else and the Advanced Options
empty. After you saved the limiter, click **Add new Queue**.

| Enable | [x] Enable |
| Name | DownloadQueue |
| Mask | None |
| Queue Management Algorithm | CoDel |
| ECN | [x] Enable |

Leave everything else and the Advanced Options empty. Save the queue.

Repeat the above limiter and queue creations for the upload - creating a **WanUpload** limiter with an **UploadQueue**
queue. Use the upload bandwith of your Internet connection for the upload limiter.

![pfSense Traffic Shaper Limiters](/assets/images/pfsense-limiter-queue.png)

Once you have the limiters set up as shown above, go to **Firewall** → **Rules** → **Floating** → **Add**.

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

Save the rule. You're done!
