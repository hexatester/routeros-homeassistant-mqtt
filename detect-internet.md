# System Resource

Send detect internet state

## Discovery script

Schedule this every 5 minutes

```rsc
:global DEV
:global OBJ
:global SN

:local indx (0)
:foreach detn in=[/interface detect-internet state find] do={
:local name [/interface detect-internet state get $detn name]
:set ($cmps->"ros-detnet$indx-state-$SN") ({"p"="sensor"; "name"="Detnet $name"; "value_template"="{{ value_json.state$indx}}"; "unique_id"="state_detnet$indx_ros_$SN"; "icon"="mdi:web"})
:set indx ($indx + 1)
}

:local discover ({"dev"=$DEV; "o"=$OBJ; "cmps"=$cmps; "qos"=2; "state_topic"="ros/$SN/detnet"})
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
