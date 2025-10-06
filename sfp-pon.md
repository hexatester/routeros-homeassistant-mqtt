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
:set ($cmps->"ros-sfp1-txpow-$SN") ({"p"="sensor"; "name"="SFP1 TX Power"; "device_class"="signal_strength"; "unit_of_measurement"="dBm"; "value_template"="{{ value_json.txpower}}"; "unique_id"="txpow_sfp1_ros_$SN"; "icon"="mdi:laser-pointer"})
:set ($cmps->"ros-sfp1-rxpow-$SN") ({"p"="sensor"; "name"="SFP1 RX Power"; "device_class"="signal_strength"; "unit_of_measurement"="dBm"; "value_template"="{{ value_json.rxpower}}"; "unique_id"="rxpow_sfp1_ros_$SN"; "icon"="mdi:laser-pointer"})
:set ($cmps->"ros-sfp1-bcurr-$SN") ({"p"="sensor"; "name"="SFP1 Bias Current"; "device_class"="current"; "unit_of_measurement"="mA"; "value_template"="{{ value_json.bcurr}}"; "unique_id"="bcurr_sfp1_ros_$SN"})

:local discover ({"dev"=$DEV; "o"=$OBJ; "cmps"=$cmps; "qos"=2; "state_topic"="ros/$SN/sfp1"})
:local rossys [:serialize value=$discover to=json]

/iot mqtt publish broker=Hass message=$rossys topic="homeassistant/device/ROS_$SN/config" qos=2
```

## Push Script

Schedule this every 1 minutes

```rsc
:global SN

:do {
/tool fetch url="http://192.168.1.1/boaform/admin/formLogin?username=admin&password=admin&save=Login&submit-url=^%^2Fadmin^%^2Flogin.asp" keep-result=no 
} on-error={ :put "login failed"};
:delay 2s

:local response [/tool fetch url="http://192.168.1.1/status_pon.asp" as-value output=user]
:delay 3s
:local statuscontent ($response->"data")
:local msg ({})

## Bellow can be different in your setup
:set ($msg->"temperature") ([:pick $statuscontent 1132 1137])
:set ($msg->"voltage") ([:pick $statuscontent 1264 1271])
:set ($msg->"txpower") ([:pick $statuscontent 1396 1400])
:set ($msg->"rxpower") ([:pick $statuscontent 1531 1536])
:set ($msg->"bcurr") ([:pick $statuscontent 1672 1680])

:local rossys [:serialize value=$msg to=json]
#:log info "$rossys"
/iot mqtt publish broker=Hass message=$rossys qos=2 topic="ros/$SN/sfp1"
```
