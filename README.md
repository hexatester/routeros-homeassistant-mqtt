# routeros-homeassistant-mqtt

Home Assistant - RouterOS bridge with Scripting &amp; MQTT. Fully Autodiscover a.k.a not required to edit `configuration.yaml`

## Requirements

- IoT Package
- RouterOS v7 that support `:serialize` (dont know which exact version, so check with running `:serialize` in terminal)

## Setup

1. Install IoT Package (under `/system package`)
2. Add MQTT Broker (under `/iot mqtt` menu) with name `Hass`

After that you can copy-paste whatever feature that you want to discover

## Features

- System Resource
- Detect Internet
- SFP PON Module Stats

Many more to come...
