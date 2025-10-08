# RouterOS - Home Assistant via MQTT

Home Assistant - RouterOS bridge with Scripting &amp; MQTT. Fully Autodiscover (not required to edit `configuration.yaml`)

If you have multiple mikrotik/ routeros, you should rename them with different name under `/system identity` menu.

## Requirements

- IoT Package
- RouterOS v7 that support `:serialize` (dont know which exact version, so check with running `:serialize` in terminal)

## Setup

1. Install IoT Package (under `/system package`)
2. Add MQTT Broker (under `/iot mqtt` menu) with name `Hass`
3. Run script bellow in the terminal and reboot

```rsc
add name=BootstrapMQTT policy=read,write,policy,test start-time=startup on-event=":do {\
    \n:global SN [/system routerboard get serial-number]\
    \n} on-error={\
    \n:global SN [/system license get system-id]\
    \n}\
    \n:global DEV ({\"ids\"=\$SN; \"sn\"=\$SN; \"mf\"=\"Mikrotik\"; \"sw\"=[/system resource get version];\"mdl\"=[\
    /system resource get board-name]; \"name\"=[/system identity get name]})\
    \n:global OBJ ({\"name\"=\"ros-iot\"; \"sw\"=[/system resource get version]; \"url\"=\"https://help.mikrotik.co\
    m/docs/spaces/ROS/pages/46759978/MQTT\"})"
```

After that you can copy-paste whatever feature that you want to discover.
Make sure that the script/ scheduler policy at least have `read,write,policy,test`

## Features

- [IP Address](/ip-address.md) - IP Address stats
- [Traffic Monitor](/traffic.md) - Interfaces Traffic stats
- [System Resource](/system-resource.md) - System Resource & System Health stats
- [Detect Internet](/detect-internet.md) - Detect Internet State
- [SFP xPON ONU Stick](/sfp-pon.md) - Send SFP xPON ONU Stick stats

more to come...

## How to schedule

1. Under `/system scheduler` menu
2. Add new
3. Set interval (for example `00:05:00` = 5 minutes)
4. Paste the code in On Event
5. Apply

## How to make graph looks beautifull?

Ask GPT on `How to make statistic helper of an sensor on Home Assistant`

## Autoinstall script?

No where is the Fun in that? You must learn how to do scripting/ schedule!
