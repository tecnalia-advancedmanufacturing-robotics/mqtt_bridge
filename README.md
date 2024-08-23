# mqtt_bridge

[![CircleCI](https://circleci.com/gh/groove-x/mqtt_bridge.svg?style=svg)](https://circleci.com/gh/groove-x/mqtt_bridge)

mqtt_bridge provides a functionality to bridge between ROS and MQTT in bidirectional.

*mqtt_bridge is not actively maintained now. Feel free to check out [mqtt_client](https://github.com/ika-rwth-aachen/mqtt_client), a high-performance C++ ROS nodelet with recent development!*

## FOR USE ON UBUNTU 20.04 with ROS NOETIC

`mqtt_bridge` and dependencies require python 3.9. otherwise it throws an error 'TypeError: 'type' object is not subscriptable'. 
20.04 and ROS Noetic are using python 3.8 and it is NOT reocmmended to upgrade to 3.9. 
So the work-around is to also install 3.9 on your machine, install the mqtt_bridge dependencies also in python 3.9.
  1) Install python3.9 :   apt install python3.9
  2) Instal the depending libraries within python3.9(defined in the "dev-requirements.txt" :   python3.9 -m pip install -r requirements-dev.txt
 
And then FORCE scripts/mqtt_bridge_node.py to use python 3.9, by modifying the first line to '#!/usr/bin/env python3.9'

## Principle

`mqtt_bridge` uses ROS message as its protocol. Messages from ROS are serialized by json (or messagepack) for MQTT, and messages from MQTT are deserialized for ROS topic. So MQTT messages should be ROS message compatible. (We use `rosbridge_library.internal.message_conversion` for message conversion.)

This limitation can be overcome by defining custom bridge class, though.


## Demo

### Prerequisites

```
$ sudo apt install python3-pip
$ sudo apt install ros-noetic-rosbridge-library
$ sudo apt install mosquitto mosquitto-clients
```

### Install python modules

```bash
$ pip3 install -r requirements.txt
```

### launch node

``` bash
$ roslaunch mqtt_bridge demo.launch
```

Publish to `/ping`,

```
$ rostopic pub /ping std_msgs/Bool "data: true"
```

and see response to `/pong`.

```
$ rostopic echo /pong
data: True
---
```

Publish "hello" to `/echo`,

```
$ rostopic pub /echo std_msgs/String "data: 'hello'"
```

and see response to `/back`.

```
$ rostopic echo /back
data: hello
---
```

You can also see MQTT messages using `mosquitto_sub`

```
$ mosquitto_sub -t '#'
```

## Usage

parameter file (config.yaml):

``` yaml
mqtt:
  client:
    protocol: 4      # MQTTv311
  connection:
    host: localhost
    port: 1883
    keepalive: 60
bridge:
  # ping pong
  - factory: mqtt_bridge.bridge:RosToMqttBridge
    msg_type: std_msgs.msg:Bool
    topic_from: /ping
    topic_to: ping
  - factory: mqtt_bridge.bridge:MqttToRosBridge
    msg_type: std_msgs.msg:Bool
    topic_from: ping
    topic_to: /pong
```

you can use any msg types like `sensor_msgs.msg:Imu`.

launch file:

``` xml
<launch>
  <node name="mqtt_bridge" pkg="mqtt_bridge" type="mqtt_bridge_node.py" output="screen">
    <rosparam file="/path/to/config.yaml" command="load" />
  </node>
</launch>
```


## Configuration

### mqtt

Parameters under `mqtt` section are used for creating paho's `mqtt.Client` and its configuration.

#### subsections

* `client`: used for `mqtt.Client` constructor
* `tls`: used for tls configuration
* `account`: used for username and password configuration
* `message`: used for MQTT message configuration
* `userdata`: used for MQTT userdata configuration
* `will`: used for MQTT's will configuration

See `mqtt_bridge.mqtt_client` for detail.

### mqtt private path

If `mqtt/private_path` parameter is set, leading `~/` in MQTT topic path will be replaced by this value. For example, if `mqtt/pivate_path` is set as "device/001", MQTT path "~/value" will be converted to "device/001/value".

### serializer and deserializer

`mqtt_bridge` uses `msgpack` as a serializer by default. But you can also configure other serializers. For example, if you want to use json for serialization, add following configuration.

``` yaml
serializer: json:dumps
deserializer: json:loads
```

### bridges

You can list ROS <--> MQTT tranfer specifications in following format.

``` yaml
bridge:
  # ping pong
  - factory: mqtt_bridge.bridge:RosToMqttBridge
    msg_type: std_msgs.msg:Bool
    topic_from: /ping
    topic_to: ping
  - factory: mqtt_bridge.bridge:MqttToRosBridge
    msg_type: std_msgs.msg:Bool
    topic_from: ping
    topic_to: /pong
```

* `factory`: bridge class for transfering message from ROS to MQTT, and vise versa.
* `msg_type`: ROS Message type transfering through the bridge.
* `topic_from`: topic incoming from (ROS or MQTT)
* `topic_to`: topic outgoing to (ROS or MQTT)

Also, you can create custom bridge class by inheriting `mqtt_brige.bridge.Bridge`.


## License

This software is released under the MIT License, see LICENSE.txt.
