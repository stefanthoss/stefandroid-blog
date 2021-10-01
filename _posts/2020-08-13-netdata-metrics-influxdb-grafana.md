---
layout: post
title: "Collect Metrics with Netdata/InfluxDB from Multiple Raspberry Pis"
description: "With Netdata, InfluxDB, and Grafana, you can collect and visualize system metrics from multiple Raspberry Pis in a central location."
tags: raspberrypi server linux
---

To collect metrics from multiple Raspberry Pis or other Linux servers in a central location, the
Netdata/InfluxDB/Grafana stack is a good solution. Netdata will collect the actual metrics and send them to a central
InfluxDB instance. A Grafana dashboard will display the metrics.

A central InfluxDB and Grafana instance should be installed and available at `192.168.1.18`. Activate the
[OpenTSDB protocol support in InfluxDB](https://docs.influxdata.com/influxdb/v1.8/administration/config/#opentsdb-settings)
on port 4242. This guide assumes a private network and does not consider encryption and authentication.

## Install Netdata

First make sure that the hostname is set to something useful as it is used to identify different servers in InfluxDB. On
Raspbian the hostname can be easily changed under Network Options in `sudo raspi-config`.

Now install the Netdata client. On most Linux systems (including Raspbian) it's as easy as

```shell
wget https://my-netdata.io/kickstart.sh
bash kickstart.sh --disable-telemetry
```

Use `--disable-telemetry` to prevent Netdata from sending usage statistics. The `kickstart.sh` script takes
approximately 30 minutes to finish on a Raspberry Pi 2. For other systems check the
[Netdata installation guide](https://learn.netdata.cloud/docs/agent/packaging/installer).

## Configuration

Tip: If you want to use `nano` for editing the configuration files, execute `export EDITOR=nano` first.

Use

```shell
cd /etc/netdata
sudo ./edit-config exporting.conf
```

to add the lines

```ini
[opentsdb:my_instance]
    enabled = yes
    destination = 192.168.1.18:4242
```

to the `exporting.conf` file (change the IP address to point to your InfluxDB). This will forward all Netdata metrics
via the OpenTSDB protocol to your InfluxDB instance. Check the
[Netdata documentation](https://learn.netdata.cloud/docs/agent/exporting/opentsdb) for more details about the OpenTSDB
exporter.

Optionally, you can also disable the integrated [Netdata web server](https://learn.netdata.cloud/docs/agent/web/server)
that usually serves the Netdata UI via HTTP on port 19999. You will check the metrics through Grafana and this will save
some system resources. Disable the web server by editing the `netdata.conf` file. Use

```shell
cd /etc/netdata
sudo ./edit-config netdata.conf
```

to set `mode = none` in the `[web]` section.

Restart the Netdata client with `sudo service netdata restart` and start collecting metrics!

## Grafana Dashboard

The Netdata metrics will now be stored in InfluxDB. Use a dashboard like the
[Templated Netdata Dashboard](https://grafana.com/grafana/dashboards/2701) to display the metrics.
