---
layout: post
title: "Unit Conversion in Home Assistant"
description: "Using a template sensor, units (like Celsius and Fahrenheit) can be easily converted in Home Assistant."
tags: homeassistant
---

Unit conversions in Home Assistant are not straight-forward, but easy to accomplish. For my use case, I have a sensor
that exposes a temperature only in Celsius, but I want to show it in Fahrenheit in the Home Assistant UI.

Home Assistant doesn't offer the functionality to convert units in-place. Instead, we have to create a second entity
with the converted value. This can be accomplished with a [template sensor](https://www.home-assistant.io/integrations/template/)
that converts the unit and exposes it as an additional entity.

I have a sensor entity `sensor.living_room_temperature` that exposes the temperature in Celsius. Add the code snippet
below to your `configuration.yml` file to convert the unit. The `state` configuration variable contains the equation for
the Celsius to Fahrenheit conversion (Fahrenheit = 1.8 * Celsius + 32).

This works for any unit conversion. You have to change the `state` equation and adjust the `unit_of_measurement` and
`device_class` to whatever your resulting unit will be.

```yaml
template:
  - sensor:
      - name: "Living Room Temperature (°F)"
        unit_of_measurement: "°F"
        device_class: "temperature"
        state_class: "measurement"
        state: "{ 1.8 * (states('sensor.living_room_temperature') | float) + 32 }}"
```

Now you have a new entity `sensor.living_room_temperature_degf` that tracks the same temperature in °F.

_Note: Don't judge me for this conversion. Celsius is a superior unit of measurement but if you live in the US,
Fahrenheit is what you have to use or nobody will understand your Home Assistant setup._

![Celsius and converted Fahrenheit values in Home Assistant](/assets/images/home-assistant-celsius-fahrenheit.png)
