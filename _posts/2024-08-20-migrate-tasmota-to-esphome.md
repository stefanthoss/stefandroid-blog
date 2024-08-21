---
layout: post
title: "Migrate Smart Plugs from Tasmota to ESPHome"
description: "You can replace the firmware of a Tasmota smart plug with ESPHome through the web interface."
tags: homeassistant
---

I've been using Tasmota smart plugs for a couple of years, but I've recently started migrating them to ESPHome. The
migration can be done entirely over-the-air and does not require physical access to the smart plug.

I prefer ESPHome for a couple of reasons:

* The integration in Home Assistant is better.
* You don't have to manually configure an MQTT broker on the device.
* Updates can be done on multiple devices directly through the Home Assistant add-on (there is the
[TasmoAdmin](https://github.com/hassio-addons/addon-tasmoadmin) plug-in, but I didn't have a good experience with that).
* I already have a couple of ESPHome devices, so they'll integrate nicely.
* The configuration of ESPHome devices is driven through a config file instead of a web UI.
* ESPHome is much more powerful in terms of configuration and customization. Example: I have a plug where I disabled
the physical switch, using it only for power monitoring.

These instructions should work with all Aoycocr-X10S-based smart plugs. I'm using the awesome [CloudFree Smart Plug 2](https://cloudfree.shop/product/cloudfree-smart-plug-runs-tasmota/)
which I strongly recommend -- that plug is solidly built, costs only $13 as of March 2023, includes power monitoring,
and is rated for up to 15 A / 1800 W. For other Tasmota devices the steps will be roughly the same, but the
configuration file has to be different. Check out [ESPHome Device](https://www.esphome-devices.com/type/plug) for a
collection of ESPHome configurations for various devices.

## Installation

The first step is to flash the minimal Tasmota firmware instead of the regular build. If you try to flash ESPHome with
the regular Tasmota firmware, you'll get an "Upload Failed. Not enough space." message in Tasmota. Go to your Tasmota
plug's web UI, select *Firmware Upgrade* and upgrade by web server using the
`http://ota.tasmota.com/tasmota/tasmota-minimal.bin.gz` OTA URL.

Next, go to the Home Assistant ESPHome add-on, click *New Device*, give the configuration a name, and select the device
type (ESP8266 for the CloudFree P2 plugs). Don't install it yet.

Now you have to edit the configuration. I'm doing it based on the config that's outlined in the
[ESPHome Device page about Aoycocr-X10S](https://www.esphome-devices.com/devices/Aoycocr-X10S-Plug) but with slight
modifications to the latest ESPHome firmware (2023.3.1 as of this writing). We're starting off with the default
configuration that ESPHome created for you, which will already include a lot of the best practices, secrets, and WiFi
configuration.

1. Add `restore_from_flash: true` to the `esp8266` section. This writes each state change to flash for switch or light
with restore_mode, check out the [documentation](https://esphome.io/components/esphome.html#esp8266-restore-from-flash)
for more info.
2. Leave the `logger`, `api`, `ota`, `wifi`, and `captive_portal` section as-is.
3. (Optional) Add a comment to the `esphome` section to describe the device
(e.g. `comment: CloudFree Smart Plug 2, based on Aoycocr-X10S Plug`)
4. Apend the following config:

    ```yaml
    # Enable time component for use by daily power sensor
    time:
      - platform: homeassistant
        id: homeassistant_time

    binary_sensor:
      # Reports when the button is pressed
      - platform: gpio
        device_class: power
        pin:
          number: GPIO13
          inverted: True
        name: Button
        on_press:
          - switch.toggle: relay

      # Reports if this device is Connected or not
      - platform: status
        name: Status

    sensor:
      # Reports the WiFi signal strength
      - platform: wifi_signal
        name: WiFi Signal

      # Reports how long the device has been powered (in minutes)
      - platform: uptime
        name: Uptime
        filters:
          - lambda: return x / 60.0;
        device_class: duration
        unit_of_measurement: min
        state_class: total_increasing

      # Reports the Current, Voltage, and Power used by the plugged-in device (not counting this plug's own usage of about 0.7W/0.02A, so subtract those when calibrating with this plugged into a Kill-A-Watt type meter)
      - platform: hlw8012
        sel_pin:
          number: GPIO12
          inverted: True
        cf_pin: GPIO5
        cf1_pin: GPIO14
        current_resistor: 0.001 #The value of the shunt resistor for current measurement. Defaults to the Sonoff POW’s value 0.001 ohm. Verified on https://fccid.io/2AKBP-X10S/Internal-Photos/X10S-Int-photo-4308983 that we use "R001" = 0.001 ohm
        voltage_divider: 2401 #The value of the voltage divider on the board as (R_upstream + R_downstream) / R_downstream. Defaults to the Sonoff POW’s value 2351. From the pic we use 2x "125" = 2x 1.2Mohm for R_upstream and "102" = 1kohm for R_downstream, so (1,200,000+1,200,000+1,000)/1,000 = 2401
        # but those don't fix the measurement values, probably because we actually have a BL0937 chip instead of a HLW8012, (and part variance aswell) so we have to manually calibrate with a known load or a load and a Kill-A-Watt type meter. My values used below will only be +/-10% of yours I think.
        # The comments about the voltage divider were taken from the AWP04L template. I was unable to verify the voltage divider in the Aoycocr X10S plug.
        power:
          name: Power
          unit_of_measurement: W
          id: wattage
          filters:
            - calibrate_linear:
                # Map 0.0 (from sensor) to 0.0 (true value)
                - 0.0 -> 0.0 #Need to keep 0 mapped to 0 for when connected device is not drawing any power
                - 67.6 -> 11.0 #Tested using a Kill-A-Watt meter and LED bulb minus 0.7W from just this plug with LED bulb off
            - lambda: if (id(relay).state) return x; else return 0;
        current:
          name: Current
          unit_of_measurement: A
          filters:
            - calibrate_linear:
                # Map 0.0 (from sensor) to 0.0 (true value)
                - 0.0 -> 0.0 #Need to keep 0 mapped to 0 for when connected device is not drawing any power
                - 0.12 -> 0.08 #Tested using a Kill-A-Watt meter and LED bulb minus 0.02A from just this plug with LED bulb off
        voltage:
          name: Voltage
          unit_of_measurement: V
          filters:
            - calibrate_linear:
                # Map 0.0 (from sensor) to 0.0 (true value)
                - 0.0 -> 0.0
                - 322.4 -> 118.3 #Tested using a Kill-A-Watt meter, value while connected LED bulb was on
        change_mode_every: 1 #Skips first reading after each change, so this will double the update interval. Default 8
        update_interval: 10s #20 second effective update rate for Power, 40 second for Current and Voltage. Default 60s

      # Reports the total Power so-far each day, resets at midnight, see https://esphome.io/components/sensor/total_daily_energy.html
      - platform: total_daily_energy
        name: Total Daily Energy
        power_id: wattage
        filters:
          - multiply: 0.001 ## convert Wh to kWh
        unit_of_measurement: kWh

    text_sensor:
      # Reports the ESPHome Version with compile date
      - platform: version
        name: ESPHome Version
      - platform: wifi_info
        ip_address:
          name: IP Address

    switch:
      - platform: gpio
        name: Switch
        pin: GPIO4
        id: relay
        restore_mode: ALWAYS_ON
        #RESTORE_DEFAULT_OFF (Default) - Attempt to restore state and default to OFF if not possible to restore. Uses flash write cycles.
        #RESTORE_DEFAULT_ON - Attempt to restore state and default to ON. Uses flash write cycles.
        #ALWAYS_OFF - Always initialize the pin as OFF on bootup. Does not use flash write cycles.
        #ALWAYS_ON - Always initialize the pin as ON on bootup. Does not use flash write cycles.
        on_turn_on:
          - light.turn_on:
              id: blue_led
              brightness: 100%
        on_turn_off:
          - light.turn_off: blue_led

    output:
      - platform: esp8266_pwm
        id: blue_output
        pin: GPIO2
        inverted: True

    light:
      - platform: monochromatic
        name: Blue LED
        output: blue_output
        id: blue_led
        restore_mode: ALWAYS_OFF #Start with light off after reboot/power-loss event.
        disabled_by_default: true

    status_led:
      pin:
        number: GPIO0
        inverted: true
    ```

5. Adapt the `restore_mode` of the `switch` section to your preferred switch behavior. See comments for descriptions
of the different options.

Next, remove the old Tasmota device from Home Assistant, otherwise it mix with the new ESPHome device. Since we already
downgraded to the "minimal" Tasmota firmware, it will no longer publish MQTT messages to the MQTT broker. There are a
couple of ways to do this, the easiest being purging the respective topics in the MQTT broker. I use
[MQTT Explorer](https://mqtt-explorer.com/) to delete the topics `/tasmota/discovery/XXXXXXXXXXXX` and
`/tele/tasmota_XXXXXX` for the device.

Now install ESPHome. Click *Install*, *Manual download*, *Legacy format*, and download the `bin` file. Go to your
Tasmota plug's web UI, select *Firmware Upgrade* and upgrade by uploading the `bin` file. The plug should reboot and
run ESPHome! The final step is to adopt the device in Home Assistant which is easy since Home Assistant auto-detects
the new device.

All future updates can be done directly through the ESPHome add-on.

Helpful links:

* <https://esphome.io/guides/migrate_sonoff_tasmota.html>

## Full Configuration Example

Here's a full config example I'm using for my espresso machine's smart plug:

```yaml
esphome:
  name: cloudfree-p2-espresso-machine
  friendly_name: CloudFree P2 Espresso Machine
  comment: CloudFree Smart Plug 2, based on Aoycocr-X10S Plug

esp8266:
  board: esp01_1m
  restore_from_flash: true

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

ota:
  platform: esphome
  password: "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Cloudfree-P2-Espresso-Machine"
    password: "XXXXXXXXXXXX"

captive_portal:

# Enable web server
web_server:
  port: 80

# Enable time component for use by daily power sensor
time:
  - platform: homeassistant
    id: homeassistant_time

binary_sensor:
  # Reports when the button is pressed
  - platform: gpio
    device_class: power
    pin:
      number: GPIO13
      inverted: True
    name: Button
    on_press:
      - switch.toggle: relay

  # Reports if this device is Connected or not
  - platform: status
    name: Status

sensor:
  # Reports the WiFi signal strength
  - platform: wifi_signal
    name: WiFi Signal

  # Reports how long the device has been powered (in minutes)
  - platform: uptime
    name: Uptime
    filters:
      - lambda: return x / 60.0;
    device_class: duration
    unit_of_measurement: min
    state_class: total_increasing

  # Reports the Current, Voltage, and Power used by the plugged-in device (not counting this plug's own usage of about 0.7W/0.02A, so subtract those when calibrating with this plugged into a Kill-A-Watt type meter)
  - platform: hlw8012
    sel_pin:
      number: GPIO12
      inverted: True
    cf_pin: GPIO5
    cf1_pin: GPIO14
    current_resistor: 0.001 #The value of the shunt resistor for current measurement. Defaults to the Sonoff POW’s value 0.001 ohm. Verified on https://fccid.io/2AKBP-X10S/Internal-Photos/X10S-Int-photo-4308983 that we use "R001" = 0.001 ohm
    voltage_divider: 2401 #The value of the voltage divider on the board as (R_upstream + R_downstream) / R_downstream. Defaults to the Sonoff POW’s value 2351. From the pic we use 2x "125" = 2x 1.2Mohm for R_upstream and "102" = 1kohm for R_downstream, so (1,200,000+1,200,000+1,000)/1,000 = 2401
    # but those don't fix the measurement values, probably because we actually have a BL0937 chip instead of a HLW8012, (and part variance aswell) so we have to manually calibrate with a known load or a load and a Kill-A-Watt type meter. My values used below will only be +/-10% of yours I think.
    # The comments about the voltage divider were taken from the AWP04L template. I was unable to verify the voltage divider in the Aoycocr X10S plug.
    power:
      name: Power
      unit_of_measurement: W
      id: wattage
      filters:
        - calibrate_linear:
            - 0.0 -> 0.0
            - 196.48288 -> 35.0
            - 3642.32764 -> 622.0
            - 6980.42383 -> 1208.0
            - 7041.69287 -> 1215.0
        - lambda: if (id(relay).state) return x; else return 0;
    current:
      name: Current
      unit_of_measurement: A
      filters:
        - calibrate_linear:
            - 0.0 -> 0.0
            - 0.51565 -> 0.46
            - 7.02498 -> 7.91
            - 12.91727 -> 10.96
        - lambda: if (id(relay).state) return x; else return 0;
    voltage:
      name: Voltage
      unit_of_measurement: V
      filters:
        - calibrate_linear:
            - 0.0 -> 0.0
            - 280.19351 -> 110.7
            - 302.31186 -> 119.7
            - 306.06781 -> 123.3
    change_mode_every: 1 #Skips first reading after each change, so this will double the update interval. Default 8
    update_interval: 10s #20 second effective update rate for Power, 40 second for Current and Voltage. Default 60s

  # Reports the total Power so-far each day, resets at midnight, see https://esphome.io/components/sensor/total_daily_energy.html
  - platform: total_daily_energy
    name: Total Daily Energy
    power_id: wattage
    filters:
      - multiply: 0.001 ## convert Wh to kWh
    unit_of_measurement: kWh

text_sensor:
  # Reports the ESPHome Version with compile date
  - platform: version
    name: ESPHome Version
  - platform: wifi_info
    ip_address:
      name: IP Address

switch:
  - platform: gpio
    name: Switch
    pin: GPIO4
    id: relay
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - light.turn_on:
          id: blue_led
          brightness: 100%
    on_turn_off:
      - light.turn_off: blue_led

output:
  - platform: esp8266_pwm
    id: blue_output
    pin: GPIO2
    inverted: True

light:
  - platform: monochromatic
    name: Blue LED
    output: blue_output
    id: blue_led
    restore_mode: ALWAYS_OFF
    disabled_by_default: true

status_led:
  pin:
    number: GPIO0
    inverted: true
```
