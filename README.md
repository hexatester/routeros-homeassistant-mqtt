# routeros-homeassistant-mqtt

Home Assistant - RouterOS bridge with Scripting &amp; MQTT. Fully Autodiscover a.k.a not required to edit `configuration.yaml`

## Requirements

- IoT Package
- RouterOS v7 that support `:serialize` (dont know which exact version, so check with running `:serialize` in terminal)

## Setup

1. Install IoT Package (under `/system package`)
2. Add MQTT Broker (under `/iot mqtt` menu) with name `Hass`
3. Run script bellow in terminal and reboot (this for makin sure autodiscover working correcly)

```rsc
/system scheduler add name=BootStrapMQTT on-event=":global SN [/system routerboard get serial-number]\
    \n:global DEV ({\"ids\"=[/system routerboard get serial-number]; \"sn\"=\$SN; \"mf\"=\"Mikrotik\"; \"sw\"=[/system resource get version];\"mdl\"=[/system routerboard get model]; \
    \"name\"=[/system identity get name]})\
    \n:global OBJ ({\"name\"=\"ros-iot\"; \"sw\"=[/system routerboard get current-firmware]; \"url\"=\"https://help.mikrotik.com/docs/spaces/ROS/pages/46759978/MQTT\"})" policy=\
    read,write,policy,test start-time=startup
```

After that you can copy-paste whatever feature that you want to discover
Make sure that the script/ scheduler policy at least have `read,write,policy,test`

## Features

- [System Resource](/system-resource.md) - System Resource & System Health stats
- [Detect Internet](/detect-internet.md) - Detect Internet State
- [SFP xPON ONU Stick](/sfp-pon.md) - Send SFP xPON ONU Stick stats

Many more to come...
