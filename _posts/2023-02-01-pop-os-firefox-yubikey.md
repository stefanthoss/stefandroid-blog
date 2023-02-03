---
layout: post
title: "Use a YubiKey with Firefox on Pop_OS!"
description: "After a fresh Pop_OS! installation, you need to install U2F libraries to use a YubiKey with Firefox."
tags: linux
---

I recommend everyone to use a [YubiKey](https://www.yubico.com/products) as a second factor authentication method for
increased security. After a fresh installation of [Pop_OS!](https://pop.system76.com), the YubiKey authentication in
Firefox doesn't seem to work (I assume the same holds true for Ubuntu and its other derivatives like Linux Mint).
The solution is easy: With a simple `sudo apt update && sudo apt install libu2f-udev`, the necessary U2F libraries
are installed and the YubiKey will start to work. Stay safe on the Internet, everyone!
