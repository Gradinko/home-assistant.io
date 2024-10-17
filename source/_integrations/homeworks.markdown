---
title: Lutron Homeworks
description: How to use Lutron Homeworks (Interactive, Series 4, and Series 8) with Home Assistant.
ha_category:
  - Binary sensor
  - Button
  - Hub
  - Light
ha_release: 0.85
ha_iot_class: Local Push
ha_domain: homeworks
ha_platforms:
  - binary_sensor
  - button
  - light
ha_integration_type: integration
ha_config_flow: true
---
## Background
[Lutron](https://www.lutron.com/) is an American lighting control company. Lutron Homeworks consists of various older systems, including Homeworks Illumination (late 1990s), Homeworks Series 4 (2000) & Homeworks Series 8 (2003). These systems use RS-232 connections to communicate with home automation systems.  The `homeworks` integration in Home Assistant is responsible for communicating with the main controller for these systems.  Communication must be made through an ethernet to serial (RS-232) converter (like the NPort line manufactured by Moxa). This integration does not support direct connections to Homeworks RS-232, as its functionality is based on the [pyhomeworks package](https://github.com/dubnom/pyhomeworks).

## Other Lutron Integrations
Lutron has created many systems over the years, each with their own unique interfacing protocol.  This platform is only for Homeworks Interactive, Homeworks Series 4 and Homeworks Series 8.  

The separate [Lutron](/integrations/lutron/) integration handles communication Lutron RadioRA 2 and Lutron Homeworks QS systems. And the [Caseta](/integrations/lutron_caseta/) integration supports the Lutron Caséta, RA2 Select (but not RA2), RadioRA 3, and Homeworks QSX (not QS) lines of products. 

## Supported Devices
This integration supports only a subset of the Homeworks system: lights (dimmers) and keypads. All devices need to be manually configured (see below) in order to work with this integration. 

## Lights & Dimmers
Once configured manually (see below), the dimmers and lights will appear in Home Assistant as a typical dimmable light.

## Keypads
Homeworks keypad buttons are momentary switches.  The button is pressed and released, meaning that there is no "state".  Buttons generate `homeworks_button_press` and `homeworks_button_release` events.  These events contain the "id", "name", and "button" of the button that was pressed.  "id" is derived from "name", and "button" is the number of the button on the keypad (starting at 1). It's also possible to add binary sensor entities which indicate if a keypad LED is lit and button entities which can be used to trigger the actions bound to a keypad button.

{% include integrations/config_flow.md %}

## Identifying Device Addresses
Lutron doesn't provide an RS-232 command for automatically detecting device in the Lutron Homeworks ecosystem. Lights and keypads need to be added manually. This is done by configuring the integration after it has been added and specifying the name and Lutron device address for each keypad and dimmer.

To find a device address, you can connect to your Homeworks Processor using RS-232 either directly from your computer or via an ethernet-to-serial convert like a Moxa NPort (which is used by this integration). Use the monitoring commands (`KBMON` and `DLMON`) found in [Lutron's RS-232 Protocol](https://assets.lutron.com/a/documents/hwi%20rs232%20protocol.pdf). The go throughout the house and press all the keypads and/or dimmers. Their addresses will appear on screen.

Alternatively, you can download the [Lutron Homeworks Interactive Software](https://files.lutron.com/hwi/software/Hwi5491-HOV.Exe) and attempt to download the database from your system either via RS-232 or TCP/IP using the "Extract Project" button found in the "Terminal" window of the software. If you are successful in doing so, you can open the extracted project and click the "Address Assignment" button to view all the devices programmed into your Homeworks system.

## Actions

### Action `send_command`

Send a custom command to the Lutron Homeworks controller. This is only necessary to use if the default lights/dimmer and keypad functionality is not sufficient for your use case.  

| Data attribute | Optional | Example                 | Description                                         |
| ---------------------- | -------- | ----------------------- | --------------------------------------------------- |
| `controller_id`        | No       | `homeworks`             | The controller to which the command should be sent to. |
| `command`              | No       | `KBP, [02:08:02:01], 1` | The command you want to send. This can either be a single command or a list of commands. In addition to the [commands supported by the controller](https://assets.lutron.com/a/documents/hwi%20rs232%20protocol.pdf), the special command `DELAY <ms>` is supported, where ms is the number of milliseconds to sleep. |

#### Sending a list of commands 

The example shows how to send `KBP`, wait 0.5 seconds, then send `KBR` to simulate a keypad button keypress with a duration of a half second.

```yaml
action: homeworks.send_command
data:
  controller_id: "homeworks"
  command:
    - "KBP, [02:08:02:01], 1"
    - "DELAY 500"
    - "KBR, [02:08:02:01], 1"
```
