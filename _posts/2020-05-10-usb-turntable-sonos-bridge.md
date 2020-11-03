---
layout: post
title: "Play music from a USB turntable on Sonos using a Raspberry Pi"
tags: raspberrypi music
---

The goal is to stream music from my [Audio-Technica AT-LP120-USB](https://www.audio-technica.com/cms/turntables/583f30b3a8662772/index.html)
to my Sonos system without using a Sonos Connect, Sonos Port, or Sonos Amp. The instructions should work for any USB
turntable or USB soundcard.

## Setup

The first step is to setup Raspbian on a headless Raspberry Pi. Download the Raspbian Lite image without a desktop and
install it as described in the [official installation instructions](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).
Then [configure Wi-Fi without display and keyboard](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)
and [enable SSH](https://www.raspberrypi.org/documentation/remote-access/ssh). After that, check the Raspberry Pi's IP
address on our router and connect via SSH. Update the system and don't forget to change the default password.

While connected to the Raspberry Pi, connect the turntable via USB. With `dmesg` you should see that the turntable gets
recognized as a new audio device:

```text
[  109.913070] usb 1-1.5: new full-speed USB device number 5 using dwc_otg
[  110.065147] usb 1-1.5: New USB device found, idVendor=08bb, idProduct=29c0, bcdDevice= 1.00
[  110.065165] usb 1-1.5: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  110.065181] usb 1-1.5: Product: USB AUDIO  CODEC
[  110.065192] usb 1-1.5: Manufacturer: BurrBrown from Texas Instruments
[  110.084469] input: BurrBrown from Texas Instruments USB AUDIO  CODEC as /devices/platform/soc/3f980000.usb/usb1/1-1/1-1.5/1-1.5:1.3/0003:08BB:29C0.0001/input/input0
[  110.153545] hid-generic 0003:08BB:29C0.0001: input,hidraw0: USB HID v1.00 Device [BurrBrown from Texas Instruments USB AUDIO  CODEC] on usb-3f980000.usb-1.5/input3
[  110.309854] usbcore: registered new interface driver snd-usb-audio
```

Identify the device number with `arecord -l`. The output should look like

```text
**** List of CAPTURE Hardware Devices ****
card 1: CODEC [USB AUDIO  CODEC], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

which will tell you the turntable is subdevice #0 of card #1 (referred to as `plughw:1,0` in the configuration below).

## Configure Icecast2 and DarkIce

Now install the necessary software packages:

```shell
sudo apt update
sudo apt install darkice icecast2
```

During the installation, a prompt will ask you to configure Icecast2. Select **yes**, keep `localhost` as the hostname,
and enter relay and administration passwords.

Using superuser privileges, create the DarkIce configuration file `/etc/darkice.cfg` with the following contents:

```ini
[general]
duration        = 0             # 0 = forever
bufferSecs      = 1             # buffer in seconds
reconnect       = yes
realtime        = yes

[input]
device          = plughw:1,0    # Soundcard device for the audio input
sampleRate      = 44100         # 44.1 kHz sample rate
bitsPerSample   = 16            # 16 bits
channel         = 2             # 2 = stereo

[icecast2-0]
bitrateMode     = cbr           # cbr = constant bit rate
format          = mp3
bitrate         = 320           # bitrate
server          = localhost
port            = 8000          # port of the Icecast2 server
password        = hackme        # password for the Icecast2 server
mountPoint      = turntable.mp3 # mount point on the Icecast2 server
name            = Turntable
description     = Turntable Audio Stream via USB
```

More details about the DarkIce configuration file can be found in the [Debian manpages](https://manpages.debian.org/buster/darkice/darkice.cfg.5.en.html).

## Start Icecast2 and DarkIce

Start the Icecast2 service and enable autostart:

```shell
sudo systemctl enable icecast2.service
sudo service icecast2 start
```

Create the file `/etc/systemd/system/darkice1.service` using superuser privileges:

```ini
[Unit]
Description=DarkIce audio streamer
After=network.target

[Service]
ExecStart=/usr/bin/darkice -c /etc/darkice.cfg
StandardOutput=inherit
StandardError=inherit
Restart=always

[Install]
WantedBy=multi-user.target
```

Start the DarkIce service and enable autostart:

```shell
sudo systemctl enable darkice1.service
sudo service darkice1 status
```

In the following I assume that your Raspberry Pi has the IP address `192.168.1.33`, replace the IP with yours. Check the
Icecast2 server status by visiting <http://192.168.1.33:8000> in your browser.

You can listen to the actual audio stream at <http://192.168.1.33:8000/turntable.mp3> in your browser. You should be
able to hear the music of your turntable.

## Configure Sonos

First you need TuneIn added as a music service in your Sonos (TuneIn doesn't require an account). Then add the Icecast2
stream as a radio station by clicking **Radio by TuneIn** → **My Radio Stations** → **Add New Radio Station**. Add the
streaming URL <http://192.168.1.33:8000/turntable.mp3> and pick a station name like "Turntable".

Play the new radio station at **Radio by TuneIn** → **My Radio Stations** → **Turntable**. There is unfortunately no
autoplay as you would have with a Sonos line-in device.

More detailed instructions can be found in [Sonos' support page](https://support.sonos.com/s/article/260?language=en_US).
