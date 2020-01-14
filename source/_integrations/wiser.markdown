---
title: Drayton Wiser
description: Instructions on how to integrate Drayton Wiser devices with Home Assistant.
logo: wiser.png
ha_category:
  - Climate
  - Sensor
  - Switch
ha_release: 0.104
ha_iot_class: Local Polling
ha_codeowners:
  - '@asantaga'
---

The `wiser` integration is the main integration to set up and integrate all supported Drayton Wiser devices. Once configured with the minimum required details it will detect and add all Drayton Wiser devices into Home Assistant, including support for multi-zone heating.

There is currently support for the following platforms within Home Assistant:

- [Climate](#climate)
- [Sensor](#sensor)
- [Switch](#switch)

This integration reverse engineers the unofficial API calls used to communicate with the HeatHub, and you will need to find out the unique secret key for your equipment to configure this Hive integration in Home Assistant.

## Obtaining your secret key

See the original reference information here (https://it.knightnet.org.uk/kb/nr-qa/drayton-wiser-heating-control/#controlling-the-system)

### Enter setup mode and connect

 - Briefly press the setup button on your HeatHub.
 - The light will start flashing green
 - Look for the 2.4GHz Wi-Fi network (SSID) called ‘WiserHeatXXX’ where XXX is random
 - Connect to the network from a Windows/Linux/Mac machine

### Fetch the 'secret' url

For Windows use the PowerShell cmdlet:

```
Invoke-RestMethod -Method Get -UseBasicParsing -Uri http://192.168.8.1/secret/
```

For Linux, Mac (or Windows WSL) use:
```
curl http://192.168.8.1/secret
```
This will return a 128-character string which is your system secret. Copy the secret and keep it safe.

### Exit setup mode

 - Press the setup button on the HeatHub again and it will go back to normal operations

To add your Drayton Wiser devices into your Home Assistant installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
wiser:
  host: "YOUR_HEATHUB_IP_ADDRESS"
  password: "YOUR_WISER_SECRET_SEE_ABOVE"
  scan_interval: 300
  minimum: -5
  boost_temp: 2
  boost_time: 30
```

{% configuration %}
host:
  description: The IP address of your Home Assistant installation
  required: true
  type: string
password:
  description: The secret key specific to your HeatHub (see above)
  required: true
  type: string
scan_interval:
  description: The time in seconds between scans of your devices. Don't reduce this too far.
  required: false
  type: integer
  default: 300
minimum:
  description: The lowest temperature in Celcius that will be recorded
  required: false
  type: integer
  default: -5
boost_temp:
  description: The temperature increase in Celcius above the set value when a boost is requested
  required: false
  type: integer
  default: 2
boost_time:
  description: The time in minutes to apply a temperature boost
  required: false
  type: integer
  default: 30
{% endconfiguration %}

## Services

### Service `wiser.boost_heating`

You can use the service `wiser.boost_heating` to set your heating to boost for a period of time at a certain target temperature".

| Service data attribute | Optional | Description                                                            |
| ---------------------- | -------- | ---------------------------------------------------------------------- |
| `entity_id`            | no       | String, Name of entity e.g., `sensor.wiser_itrv_bedroom`                         |
| `time_period`          | no       | Integer, Period of time the boost should last for in minutes e.g. `30` |
| `temperature`          | yes      | String, The required target temperature in Celcius e.g., `20.5`                   |
| `temperature_delta`          | yes      | String, The required boost above the set temperature e.g., `2.5`                   |

Examples:

```yaml
# Example script to boost heating, boost period and target temperature specified.
script:
  boost_heating:
    sequence:
      - service: wiser.boost_heating
        data:
          entity_id: "sensor.wiser_itrv_bedroom"
          time_period: "45"
          temperature: "20.5"
```

## Platforms

<div class='note'>

You must have the [Drayton Wiser integration](/components/wiser/) configured to use the platforms below.

</div>

### Sensor

The `wiser` sensor integration exposes Drayton Wiser data as a sensor.

The platform exposes the following sensors:

- HeatHub
- Smart Room Thermostat
- Smart Radiator Thermostat

### Climate

The `wiser` climate platform integrates your Drayton Wiser HeatHub, room thermostat and ITRV radiator valves into Home Assistant, enabling control of setting the **mode** and setting the **target temperature**.

A short boost for can be set by using the **Boost** option.

The platform supports the following Drayton Wiser products:

- Smart Room Thermostat
- Smart Radiator Thermostat

### Switch

The `wiser` switch platform integrates your HeatHub 'Away' mode into Home Assistant.

The platform supports the following devices:

- HeatHub
