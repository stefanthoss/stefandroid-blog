---
layout: post
title: "Eaton UPS Fan Swap"
description: "Replace the factory fan in an Eaton 5P UPS with a quieter Noctua fan."
tags: ups linux hardware
---

I have an Eaton 5P1500RC UPS which is a great 2U UPS for a homelab. It has however a relatively loud fan that is running at all times so I wanted to replace the fan with a quieter fan. Do not do this unless you know your way around electricity.

The factory-installed fan is a DWPH EFS-08E12D-ER09. It's hard to find exact specs on this model online besides what's printed on the fan itself. I decided to replace it with a Noctua NF-R8 redux-1800. That's a 12V 3-pin fan using up to 0.11 A and providing 31 CFM of airflow which should be sufficient for this UPS.

![Eaton UPS Factory Fan Label](/assets/images/eaton-ups-standard-fan-label.jpg)

## Fan Swap

First turn off the UPS and disconnect it from the grid. You have to remove only one screw in the back to slide off the top cover:


![Back of the Eaton UPS](/assets/images/eaton-ups-outside.jpg)

In the back you can see the factory-installed fan:

![Factory-installed fan in the Eaton UPS](/assets/images/eaton-ups-standard-fan-installed.jpg)

You have to remove the fan's 4 screws from the outside, you can then remove the fan. Next, we have to swap the connectors. You can see in this picture that the fan connector that Eaton uses (white) and the Noctua fan connector that's common in PCs (black) are different:

![Fan connectors of the factory fan and Noctua](/assets/images/eaton-ups-connector.jpg)

There are a couple of options here. You could cut them off and solder the factory connector to the Noctua fan. I just pulled out the little metal contact pins from both connectors and swapped them, i.e. pushing the contact pins from the Noctua fan cable into the white plastic housing of the factory fan. This has worked pretty well. In any case, pay attention to the wire colors as they are not in the same order.

Now plug the new fan back in and install it in the back of the UPS:

![Noctua installed inside the Eaton UPS](/assets/images/eaton-ups-noctua-fan.jpg)

Close the UPS and power it back on. You might get an error message about a fan fault. You can reset it on the UPS menu under *Control* -> *Reset Fault State*. The message will then disappear.

## Testing

Overall, I feel like the UPS has gotten significantly quieter. Some basic testing using a sound meter app on my phone gives me the following reading:

* Factory fan: ~54 dB behind and above the UPS
* Noctua fan: ~45 dB behind and above the UPS

I also simulated a Noctua fan failure and the UPS is alerting accordingly. So the Eaton UPS is recognizing the fan correctly and tracks its RPM.
