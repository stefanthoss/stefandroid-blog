
[iPerf3](https://iperf.fr/)

Client: Debian 10 with iPerf 3.6
Server (192.168.10.1): pfSense 2.4.5 with iPerf 3.7

## Client to Server (Client Upload Speed)

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

iperf Done.
```

## Server to Client (Client Download Speed)

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

iperf Done.
```
