# SFP xPON ONU Stick

Send SFP xPON ONU Stick stats.

- More info about the SFP PON Module https://github.com/Anime4000/RTL960x
- Derrived from https://github.com/Anime4000/RTL960x/issues/99#issuecomment-1611726668 by [smnrock](https://github.com/smnrock)

## Discovery script

Schedule this every 5 minutes

```rsc
:global DEV
:global OBJ
:global SN

:local cmps ({})
:set ($cmps->"ros-sfp1-temp-$SN") ({"p"="sensor"; "name"="SFP1 Temperature"; "device_class"="temperature"; "unit_of_measurement"="°C"; "value_template"="{{ value_json.temperature}}"; "unique_id"="temp_sfp1_ros_$SN"})
:set ($cmps->"ros-sfp1-volt-$SN") ({"p"="sensor"; "name"="SFP1 Voltage"; "device_class"="voltage"; "unit_of_measurement"="V"; "value_template"="{{ value_json.voltage}}"; "unique_id"="volt_sfp1_ros_$SN"})
:set ($cmps->"ros-sfp1-txpow-$SN") ({"p"="sensor"; "name"="SFP1 TX Power"; "device_class"="signal_strength"; "unit_of_measurement"="dBm"; "value_template"="{{ value_json.txpower}}"; "unique_id"="txpow_sfp1_ros_$SN"})
:set ($cmps->"ros-sfp1-rxpow-$SN") ({"p"="sensor"; "name"="SFP1 RX Power"; "device_class"="signal_strength"; "unit_of_measurement"="dBm"; "value_template"="{{ value_json.rxpower}}"; "unique_id"="rxpow_sfp1_ros_$SN"})
:set ($cmps->"ros-sfp1-bcurr-$SN") ({"p"="sensor"; "name"="SFP1 Bias Current"; "device_class"="current"; "unit_of_measurement"="mA"; "value_template"="{{ value_json.bcurr}}"; "unique_id"="bcurr_sfp1_ros_$SN"})

:local discover ({"dev"=$DEV; "o"=$OBJ; "cmps"=$cmps; "qos"=2; "state_topic"="ros/$SN/sfp1"})
:local rossys [:serialize value=$discover to=json]

/iot mqtt publish broker=Hass message=$rossys topic="homeassistant/device/ROS_$SN/config" qos=2
```

## Push Script

Schedule this every 1 minutes

```rsc
:global SN

:local msg ({})
:local indx (0)
:foreach detn in=[/interface detect-internet state find] do={
:set ($msg->"state$indx") ([/interface detect-internet state get $detn state])
:set indx ($indx + 1)
}

:local rossys [:serialize value=$msg to=json]
/iot mqtt publish broker=Hass message=$rossys qos=2 topic="ros/$SN/detnet"
```
