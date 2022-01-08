---
layout: post
title: "Monitor the Health of Multiple SMART Disks with Scrutiny"
description: "Scrutiny can monitor the SMART health of multiple disks in multiple servers and expose it through a web interface."
tags: linux server truenas
---

[Scrutiny](https://github.com/AnalogJ/scrutiny) is a tool that collects SMART hard drive data and exposes them
through a web UI. You can run a central Docker container that contains the database/UI and install the data collection
agent on multiple clients. This will provide a single interface to monitor all hard drives in all servers. I'm using it
to monitor my TrueNAS SSDs and the NVME drives in my Debian servers.

## Web UI

The web UI can be deployed using the `analogj/scrutiny:web` Docker image (the `analogj/scrutiny:latest` image contains
both the web UI and the data collection agent). I use docker-compose files to deploy the application. Create a new
directory and the following Dockerfile:

```yaml
version: "3"

services:
  scrutiny:
    image: analogj/scrutiny:web
    container_name: scrutiny-web
    restart: unless-stopped
    ports:
      - "8080:8080"
    healthcheck:
      test: "curl -f -s http://localhost:8080/web/dashboard"
    volumes:
      - ./config:/scrutiny/config
    environment:
      - GIN_MODE=release
```

Or download the file from [my collection of docker-compose files](https://github.com/stefanthoss/container-fest) and
start it:

```shell
mkdir scrutiny
cd scrutiny
wget https://raw.githubusercontent.com/stefanthoss/container-fest/main/scrutiny/docker-compose.yml
docker-compose up -d
```

The web UI will be available at port 8080 of the Docker host. Here's a screenshot of the web ui:

![Scrutiny Webapp Dashboard](/assets/images/scrutiny-webapp-dashboard.png)

## Collector on TrueNAS

Scrutiny needs Smartmontools version 7. Check on the TrueNAS terminal that version 7 is installed. TrueNAS Core 12.0
ships with Smartmontools 7.2 as checked with the `smartctl -V` command:

```text
smartctl 7.2 2020-12-30 r5155 [FreeBSD 12.2-RELEASE-p6 amd64] (local build)
```

Download the Collector agent binary (below link is for version 0.3.13 - the latest as of January 2022). Then copy it to
`/usr/local` and make it executable. Execute the following as `root`:

```shell
mkdir -p /usr/local/tools/scrutiny/bin
wget https://github.com/AnalogJ/scrutiny/releases/download/0.3.13/scrutiny-collector-metrics-freebsd-amd64 -P /usr/local/tools/scrutiny/bin
chmod +x /usr/local/tools/scrutiny/bin/scrutiny-collector-metrics-freebsd-amd64
```

Create the configuration file by downloading the sample `collector.yaml` from the GitHub repo. Edit it (e.g. with `nano`)
and change the following parameters:

* `host.id` should be an identifier for your TrueNAS server
* `api.endpoint` should be the HTTP endpoint of the Scrutiny Web server

Everything else can be left as is. Here are the commands which have  to be executed as `root`:

```shell
mkdir -p /usr/local/tools/scrutiny/config
wget https://raw.githubusercontent.com/AnalogJ/scrutiny/master/example.collector.yaml -O /usr/local/tools/scrutiny/config/collector.yaml
nano /usr/local/tools/scrutiny/config/collector.yaml
```

![TrueNAS Scrutiny Cronjob](/assets/images/truenas-scrutiny-cronjob.png)

Now go to **Tasks** → **Cron Jobs** and add a new cron job:

| Description | Scrutiny |
| Command | `. /etc/profile; /usr/local/tools/scrutiny/bin/scrutiny-collector-metrics-freebsd-amd64 run --config /usr/local/tools/scrutiny/config/collector.yaml` |
| Run As User | root |
| Schedule | Hourly |

Enable both `Hide Standard Output` and `Hide Standard Error` once you confirmed the collector to be working. Otherwise
TrueNAS will send an email with the full log for every single log.

At the start of the next hour, the SMART data should appear in the Scrutiny Web UI.

## Collector on Debian Bullseye

Follow the [Scrutiny installation instructions](https://github.com/AnalogJ/scrutiny/blob/master/docs/INSTALL_MANUAL.md#collector)
to install the collector agent. You don't have to install the Webapp/API.

Edit the `/opt/scrutiny/config/collector.yaml` file and change the following parameters:

* `host.id` should be an identifier for your TrueNAS server
* `api.endpoint` should be the HTTP endpoint of the Scrutiny Web server

Add the following tab using `crontab -e`:

```text
0 * * * * . /etc/profile; /opt/scrutiny/bin/scrutiny-collector-metrics-linux-amd64 run --config /opt/scrutiny/config/collector.yaml
```
