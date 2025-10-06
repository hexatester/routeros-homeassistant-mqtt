# Traffic Monitor

Send interfaces traffic. **Make sure to change the interface that you want to monitor in `tomonitor` variable**.

To change the unit of measurement for a Home Assistant entity, go to the Settings > Devices & Services > Entities page, find the entity, and click the cog icon to open its more info dialog. Then, click the settings cog within the dialog, find the Unit of measurement option, and select your desired unit from the dropdown menu.

You can smooth up the data with statistic helper.

## Discovery script

Schedule this every 5 minutes

```rsc
:global DEV
:global OBJ
:global SN

:local tomonitor ({"ether1"; "ether2"; "ether3"; "ether4"; "ether5"}) # change this
:local cmps ({})
:local indx (0)
:foreach trafmon in=$tomonitor do={
:set ($cmps->"ros-traffic$indx-rx-$SN") ({"p"="sensor"; "name"="$trafmon RX"; "value_template"="{{ value_json.rx$indx}}"; "device_class"="data_rate"; "unit_of_measurement"="bit/s"; "unique_id"="rx_traffic$indx_ros_$SN"})
:set ($cmps->"ros-traffic$indx-tx-$SN") ({"p"="sensor"; "name"="$trafmon TX"; "value_template"="{{ value_json.tx$indx}}"; "device_class"="data_rate"; "unit_of_measurement"="bit/s"; "unique_id"="tx_traffic$indx_ros_$SN"})
:set indx ($indx + 1)
}

:local discover ({"dev"=$DEV; "o"=$OBJ; "cmps"=$cmps; "qos"=2; "state_topic"="ros/$SN/traffic"})
:local rossys [:serialize value=$discover to=json]
/iot mqtt publish broker=Hass message=$rossys topic="homeassistant/device/ROS_$SN/config" qos=2
```

## Push Script

Schedule this every 1 minutes

```rsc
:global SN

:local tomonitor ({"ether1"; "ether2"; "ether3"; "ether4"; "ether5"}) # change this
:local msg ({})
:local indx (0)
:foreach trafmon in=$tomonitor do={
:do {
/interface monitor-traffic $trafmon once do={
:set ($msg->"rx$indx") ($"rx-bits-per-second")
:set ($msg->"tx$indx") ($"tx-bits-per-second")
}
} on-error={
:log warning "Invalid interface name: $trafmon"
}
:set indx ($indx + 1)
}

:local rossys [:serialize value=$msg to=json]
#:log info "$rossys"
/iot mqtt publish broker=Hass message=$rossys qos=2 topic="ros/$SN/traffic"
```
