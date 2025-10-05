# System Resource

Send system resource and system health

## Discovery script

Schedule this every 5 minutes

```rsc
:global DEV
:global OBJ
:global SN

:local cmps ({})

:set ($cmps->"ros-system-cpu-$sn") ({"p"="sensor"; "name"="CPU Load"; "unit_of_measurement"="%"; "value_template"="{{ value_json.cpu}}"; "unique_id"="cpu_ros_$SN"})
:set ($cmps->"ros-system-ram-$sn") ({"p"="sensor"; "name"="RAM Used"; "unit_of_measurement"="%"; "value_template"="{{ value_json.ram}}"; "unique_id"="ram_ros_$SN"})

:do {
:if ([:tobool [/system health find type="C"]]) do={
:set ($cmps->"ros-system-temp-$SN") ({"p"="sensor"; "device_class"="temperature"; "unit_of_measurement"="°C"; "value_template"="{{ value_json.temperature}}"; "unique_id"="temp_ros_$SN"})
}
:if ([:tobool [/system health find type="V"]]) do={
:set ($cmps->"ros-system-volt-$SN") ({"p"="sensor"; "device_class"="voltage"; "unit_of_measurement"="V"; "value_template"="{{ value_json.voltage}}"; "unique_id"="volt_ros_$SN"})
}
} on-error={}

:local discover ({"dev"=$DEV; "o"=$OBJ; "cmps"=$cmps; "qos"=2; "state_topic"="ros/$SN/system"})
:local rossys [:serialize value=$discover to=json]

/iot mqtt publish broker=Hass message=$rossys topic="homeassistant/device/ROS_$SN/config" qos=2
```

## Push Script

Schedule this every 1 minutes

```rsc
:global SN

:local msg ({})
:set ($msg->"cpu") ([/system resource get cpu-load])
:set ($msg->"ram") (100 * ([/system resource get total-memory]-[/system resource get free-memory])/[/system resource get total-memory])

:do {
:if ([:tobool [/system health find type="C"]]) do={
:set ($msg->"temperature") ([/system health get [find type="C"] value])
}
:if ([:tobool [/system health find type="V"]]) do={
:set ($msg->"voltage") ([/system health get [find type="V"] value])
}
} on-error={}

:local rossys [:serialize value=$msg to=json]

/iot mqtt publish broker=Hass message=$rossys qos=2 topic="ros/$SN/system"
```
