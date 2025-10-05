# SFP xPON ONU Stick

Send `/ip address` stats

## Discovery script

Schedule this every 5 minutes

```rsc
:global DEV
:global OBJ
:global SN

:local indx (0)
:foreach ipaddr in=[/ip address find] do={
:local iface [/ip address get $ipaddr interface]
:set ($cmps->"ros-ipaddress$indx-address-$SN") ({"p"="sensor"; "name"="$iface IP Address"; "value_template"="{{ value_json.address$indx}}"; "unique_id"="address_ipaddr$indx_ros_$SN"; "icon"="mdi:ip"})
:set ($cmps->"ros-ipaddress$indx-network-$SN") ({"p"="sensor"; "name"="$iface IP Network"; "value_template"="{{ value_json.network$indx}}"; "unique_id"="network_ipaddr$indx_ros_$SN"; "icon"="mdi:ip-network"})
:set indx ($indx + 1)
}

:local discover ({"dev"=$DEV; "o"=$OBJ; "cmps"=$cmps; "qos"=2; "state_topic"="ros/$SN/ipaddress"})
:local rossys [:serialize value=$discover to=json]
/iot mqtt publish broker=Hass message=$rossys topic="homeassistant/device/ROS_$SN/config" qos=2
```

## Push Script

Schedule this every 1 minutes

```rsc
:global SN

:local msg ({})
:local indx (0)
:foreach detn in=[/ip address find] do={
:set ($msg->"address$indx") ([/ip address get $detn address])
:set ($msg->"network$indx") ([/ip address get $detn network])
:set indx ($indx + 1)
}

:local rossys [:serialize value=$msg to=json]
/iot mqtt publish broker=Hass message=$rossys qos=2 topic="ros/$SN/ipaddress"
```
