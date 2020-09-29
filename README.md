## NOTE:

This is a fork of the great work of the1laz. This code has been modified to better support Home Assistant, in particular:
- Dimming from Home Assistant
- MQTT Discovery to auto discover services on the C-Bus network. This requires the 'getallonstart' option to be enabled.



cgateweb
========

MQTT interface for C-Bus written in Node.js

### Install:
(for raspberry pi)
```
cd /usr/local/bin
sudo git clone https://github.com/the1laz/cgateweb.git
cd cgateweb
npm install
sudo nano settings.js
```
Put in your settings.
```
sudo cp cgateweb.service /etc/systemd/system
sudo systemctl start cgateweb
sudo systemctl enable cgateweb
```
### Usage if not using as service:

1) Put your settings in settings.js
2) Run node index.js

Detailed guides and supporting articles will be posted on my blog: https://blog.addictedtopi.com/category/project/cbus/

### Updates get published on these topics:

 - cbus/read/#1/#2/#3/state  -  ON/OFF gets published to these topics if the light is turned on/off

 - cbus/read/#!/#2/#3/level  -  The level of the light gets published to these topics

### Publish to these topics to control the lights:

 - cbus/write/#1/#2/#3/switch  -  Publish ON/OFF to these topics to turn lights on/off

 - cbus/write/#1/#2/#3/ramp  -  Publish a % to ramp to that %. Optionally add a comma then a time (e.g. 50,4s or 100,2m). Also, INCREASE/DECREASE ramp by 5% up or down and ON/OFF turns on/off.

### This requests an update from all lights:

 - cbus/write/#1/#2//getall - current values get published on the cbus/read topics

 #1,#2 and #3 should be replaced by your c-bus network number, application number, and the group number.

Requesting an update on start or periodic updates can be set in the settings file.

### This requests the network tree:

 - cbus/write/#1///gettree - result gets published as JSON on cbus/read/#1///tree

### Other notes:
I made this for working with OpenHAB
It assumes the default cgate ports

---
## Home Assistant extensions

### Home Assistant discovery:
Enable home assistant discovery with the following in settings.js
```
exports.topicPrefix = "homeassistant"
exports.enableHassDiscovery = true;
```

Assumes the following MQTT configuration for Home Assistant (configuration.yaml)
```
broker: 127.0.0.1
port: 1883
keepalive: 60
birth_message:
  topic: "hass/status"
  payload: "online"
will_message:
  topic: "hass/status"
  payload: "offline"
discovery: true
discovery_prefix: homeassistant
```

Discovery messages are sent when cgateweb starts. When home assistant restarts entities become "unavailable". Sending the following topic will trigger discovery messages to be resent.
- cbus/write/#1/#2//discovery

### Useful automation
```
- id: poll_cbus_on_startup
  alias: Poll c-gate for entities on startup
  description: Polls c-gate service for list of entities and discovery messages, otherwise they all show up disabled
  trigger:
  - event: start
    platform: homeassistant
  condition: []
  action:
  - delay: '120'
  - data:
      topic: homeassistant/cbus/write/254/56//discovery
    service: mqtt.publish
  mode: single
  ```