---
layout: post
title: "Building an Arm-based Homelab Server"
description: "I recently built a powerful Arm-based homelab server using an Ampere Altra motherboard."
tags: server linux hardware
---

Until last year I've been running all of my self-hosted services on an older 2U server powered by an Intel Xeon Silver 4214 CPU. The server was reasonably powerful but very loud and power hungry (averaging 134W). So I wanted to replace it with something less power hungry and quieter fans. I read on [ServerTheHome](https://www.servethehome.com/asrock-rack-altrad8ud-1l2t-review-this-is-the-ampere-arm-motherboard-you-want/) about the new Ampere Altra Arm processors and ASRock motherboards so I wanted to use Arm instead of Intel/AMD x64 for my next build. Arm is usually less power hungry and I had good experiences with the [Ampere-based cloud server on Hetzner](https://www.hetzner.com/press-release/arm64-cloud).

## Building

I settled on the following build:

* Motherboard: [ASRock ALTRAD8UD-1L2T](https://www.asrockrack.com/general/productdetail.asp?Model=ALTRAD8UD-1L2T) ("Deep" Micro-ATX, PCIe 4.0, up to 2TB RAM)
* CPU: [Ampere Altra Q64-22](https://amperecomputing.com/briefs/ampere-altra-family-product-brief) (64 cores at 2.2 GHz, TDP 69W)
* CPU Cooler: [Arctic Freezer 4U-M](https://www.arctic.de/us/Freezer-4U-M/ACFRE00133A)
* RAM: 6x Hynix 32GB (DDR4, 2666MHz)
* PSU: [Corsair RM750e](https://www.corsair.com/us/en/p/psu/cp-9020262-na/rme-series-rm750e-fully-modular-low-noise-atx-power-supply-cp-9020262-na) (ATX, 750W)
* Storage: SK hynix Platinum P41 (PCIe 4.0, 1TB)
* GPU: Nvidia RTX A4000 (16GB VRAM)
* Case: [Sliger CX4150a](https://www.sliger.com/products/rackmount/4u/cx4150a) (4U, 15"/38cm deep)
* Case Fans: 3x [Arctic P12 Max](https://www.arctic.de/us/P12-Max/ACFAN00280A)

The ASRock motherboard is pretty much the only Ampere motherboard available right now and while expensive, it has some cool features: 4 full-size PCIe 4.0 slots, 8 DDR4 RAM slots, and 10GbE networking. Also, [Newegg sells a bundle](https://www.newegg.com/asrock-rack-altrad8ud-1l2t-q64-22-ampere-altra-max-ampere-altra-processors/p/N82E16813140134) with the motherboard and CPU whereas Ampere-based systems are usually not easily available to consumers. For the coolers I went with Arctic instead of Noctua because they were significantly cheaper and also very quiet. The motherboard has support for 3200 MHz DDR4 RAM but I had some 2666 MHz RAM from my last build so I just used that. For the case I went with the Sliger case because I wanted (1) a 4U case for the massive cooler and (2) a short-depth case that fits into my 18"/46cm deep rack. For my AI workloads I'm using an Nvidia RTX A4000 (I'll probably do an LLM comparison of various graphics cards in the future). I was considering an AMD GPU but those have generally no to little Arm support since AMD has a competing CPU business to Arm. The 750W PSU is massively oversized but I had it on hand.

The finished build looks like this:

![Ampere Server Build Top](/assets/images/ampere-server-1.jpg)
![Ampere Server Build Side](/assets/images/ampere-server-2.jpg)

## Installation

The default login that is used by OpenBMC out-of-the-box is:

* Username: `root`
* Password: `0penBmc` (Key there is that the "O" in Open is actually the digit zero `0`)

Now let's install Debian. For my old homelab server I used Proxmox and VMs but since I migrated all my self-hosted services to Docker Compose, I'm just going with bare-metal Debian this time. Also, Proxmox doesn't officially support Arm CPUs although there are community-supported ways.

Debian download:

* https://www.debian.org/distrib/netinst
* ARM64 image: https://cdimage.debian.org/debian-cd/current/arm64/iso-cd/debian-12.9.0-arm64-netinst.iso

In the OpenBMC dashboard, go to *Operations* -> *Virtual Media* and upload the image. Then reboot and boot from the "Linux File-Stor Gadget" disk.

## Power Consumption

And now the big question, how much power does the system consume? I measured the power with a Kill A Watt at the wall outlet. I used the 10GbE port during all measurements which presumably adds to the power consumption.

Condition | GPU | RAM | Power Consumption
--- | --- | --- | ---
Idle | None | 4x 32GB | 54W
100% CPU load (*) | None | 4x 32GB | 91W
Idle | RTX A4000 | 4x 32GB | 68W
100% CPU load (*) | RTX A4000 | 4x 32GB | 100W
Normal operation (**) | RTX A4000 | 6x 32GB | 86W
Normal operation (**) | RTX A4000 | 8x 32GB | 101W

(*) Load created using `stress-ng`.

(**) Normal operation means all of my self-hosted services are deployed and running - a total of ~50 Docker containers.

Take-aways:
* Only 54W at idle with 4 sticks of RAM is pretty good for sucha  system.
* The GPU idles at around 14W.
* I was surprised that each stick of DDR4 RAM uses around 7.5W. I ended up settling at 6 x 32 GB = 192GB RAM which is sufficient for my server and doesn't waste more power.

## Gotchas

Not all Docker containers are available for Arm CPUs, especially those with GPU features often lack an `arm64` variant.

It is hard to find an OS to boot if you need a live linux for debugging, I used the Ubuntu ARM64 for RPi. Most live Linux distros are AMD64 only.
