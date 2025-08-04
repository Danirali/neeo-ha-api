# Using NEEO Remote (prior control4) with Home Assistant
Contents:
- [HA REST API](#HA-Rest-API)
  - [Media Player Driver](#media-player-driver-rest-api)
  - [Light Driver](#light-driver-rest-api)
  - [Generic IR Device](#generic-ir-rest-api-broadlink)
- [HA Emulated Hue](#emulated-hue)
- [MQTT Bridge to HA](#mqtt-bridge-to-ha)
<br>

There are a few ways to integrate Home Assistant to the NEEO remote with the help of [metadriver](https://github.com/jac459/metadriver) The three popular methods are:
- MQTT (Advanced)
- HA Rest API (Moderate Difficulty for less savy users)
- HA Emulated Hue (Easiest)

Due to my lack of expertise with MQTT, I will be showing you how you can integrate HA through Emulated Hue and Rest API, I've left *templates* in the repo files as generic drivers to run your devices.

<br>

# HA Rest API

The HA Rest API is the easiest to integrate and you can get help from many AI models. When making a driver, the layout is usually the similar. When using the HA API the beginning of the drive is always the same. These variables store you HA instance's login credientials (it is advised to create a new login).

```
  "variables": {
    "HA_IP": "YOUR_HA_IP",
    "HA_Port": "8123",
    "HA_Token": "HAP_API_KEY",
  },
```

I have currently developed my drivers to rely on static configs, I _do not_ use *discovery* as I like to pick the devices I want and therefore, manually integrate them. Here is the structure of the discovery section.

```
"discover": {
    "welcomeheadertext": "Driver Title",
    "welcomedescription": "Driver Description",
    "command": {
      "type": "static",
      "command": ".",
      "queryresult": "$.*"
    }
  },
```
<br>
<br>

## Media Player Driver (Rest API)
The Media Player Driver can be adjusted without altering too much code by changing the variables. The media player driver was originally designed to work with Apple TV however can work with (nearly) any Media Player. I've tested the driver on LG webOS TV and Apple TV (integrated into HA).

This driver directly maps the controls to the controlpad of the NEEO remote. For example, using the Volume buttons on the NEEO remote adjusts the volume on your device. 

To driver relies on the `remote.send_command` service. To change your commands, you can edit the following variables to match your TV.

```
"template": {
    "name": "$Name",
    "dynamicname": "$Name",
    "dynamicid": "$EntityId_ha_http",
    "variables": {
      "HA_IP": "$HA_IP",
      "HA_Port": "$HA_Port",
      "HA_Token": "$HA_Token",
      "EntityId": "$EntityId",
      "SwitchedOn": false,
      "DEBUG": "",
      "Volume": 0,
      "PlayerState": "unknown",
      "MediaCoverURI": "",
      "UP": "up",
      "DOWN": "down",
      "LEFT": "left",
      "RIGHT": "right",
      "ENTER": "select",
      "BACK": "menu",
      "MENU": "home",
      "MUTE": "mute"
  },
```

#### NOTE ####
Do not edit any variables until after 'MediaCoverURI'. The variables after this one are your controls, you edit the data on the right for example:

For Apple TV: to go back the key is `menu` so `"BACK": "menu"`

<br>
<br>

## Light Driver (Rest API)
The driver for the the light driver can be found in the files as `HA_API_Light.json`. To configure the driver to your spec you will need to modify the following variables:
```
  HA_IP, HA_Token, EntityId
```
These variables are located at the top of the file under the variables section.
Full file context:
```
{
  "name": "HA API Light by Danirali",
  "manufacturer": "Home Assistant",
  "type": "LIGHT",
  "version": 2,
  "variables": {
    "HA_IP": "<HA_IP>",
    "HA_Port": "8123",   
    "HA_Token": "<HA_API_TOKEN>",
    "EntityId": "light.kitchen_island_lights"
  },
```
This driver supports basic light functions;
- ON
- OFF
- BRIGHTNESS

<br>
<br>

## Generic IR (Rest API) Broadlink
This driver was designed for my IR fireplace that is controlled via a Broadlink RM4 Pro, I learned all the remote buttons via home assistant using `remote.learn_command` I used the `device` field since I have multiple IR devices.

Main config:
```
"variables": {
    "HA_Ip": "<HA_IP>", 
    "HA_Port": "8123",
    "EntityId": "<DUMMY_TOGGLE>",
    "HA_ApiToken": "<HA_API_Token>", 
    "IRController": "remote.rm4_pro", // Entity ID of remote entity of IR controller (delete label if pasting)
    "Name": "Fireplace" // Name of device as learnt in IR controller (delete label if pasting)
  },
```

The config for the action will have to be added manually added since every IR device is different.

```
"template": {
    "name": "$Name",
    "dynamicname": "$Name",
    "dynamicid": "$Name_ha_http", 
    "variables": {
      "HA_Ip": "$HA_Ip",
      "HA_Port": "$HA_Port",
      "HA_ApiToken": "$HA_ApiToken",
      "EntityId": "$EntityId",
      "IRController": "$IRController",
      "RemoteStatus": "",
      "cmd_brightness": "COLOUR1",
      "cmd_colour": "COLOUR2",
      "cmd_heat_on": "TEMP_INC",
      "cmd_heat_off": "TEMP_DEC",
      "SwitchedOn": false
    },
```

I have an IR Fireplace and I have an automation that turns it on and off using an `input_boolean` (toggle helper), I would put `input_boolean.fireplace` in the <DUMMY_TOGGLE>. The commands I sent are:
`COLOUR1, COLOUR2, TEMP_INC, TEMP_DEC`
I assigned them as variables so that IR commands can easily be changed (great for debugging).

<br>
<br>

# Emulated Hue

This method is by far the easiest to integrate the entirety of your HA instance.
In HA will need edit your `configuration.yaml` with following:
```
emulated_hue:
  expose_by_default: true
  exposed_domains:
  - light
```
The documentation can be found [here](https://www.home-assistant.io/integrations/emulated_hue/).
### Please Note ###
Home Assistant is capable of exposing more domains to Emulated Hue however the driver has been written to work on LIGHT components, if you wish to use other components, refer to metadriver's documentation of allowed domains.
The driver for this has been adapted from the original driver: [meta-hue](https://github.com/jac459/meta-hue) developed by [JAC459](https://github.com/jac459)

Driver:
```
{ "name":"Emulated Hue by Danirali",
  "manufacturer":"Home Assistant",
  "type":"LIGHT",
  "version":21,
  "persistedvariables":{
    "ToInitiate":true,
    "HueBridgeIp":"<HA_IP>",
    "HueUserName":"v2",
    "HueBridgePort":"<EMULATED_PORT (8300)>"
  },
  "discover":{
    "welcomeheadertext":"Hue Driver",
    "welcomedescription":"Warning ! Press the button on your Hue bridge before pressing next.",
    "command": {"type":"http-get", "command":"http://$HueBridgeIp:$HueBridgePort/api/$HueUserName/lights", "queryresult":["$.*~","$..name"]}
  },
  "template" : {
    "name":"Hue", 
    "dynamicname":"DYNAMIK_INST_START DYNAMIK \"Light \" + JSON.parse(\"$Result\")[1] DYNAMIK_INST_END",
    "dynamicid":"DYNAMIK_INST_START DYNAMIK \"Light \" + JSON.parse(\"$Result\")[0] DYNAMIK_INST_END",
    
    "variables":{
      "HueBridgeIp":"$HueBridgeIp",
      "HueUserName":"$HueUserName",
      "HueBridgePort":"$HueBridgePort",  
      "LightNumber":"DYNAMIK_INST_START DYNAMIK JSON.parse(\"$Result\")[0] DYNAMIK_INST_END",
      "Brightness":"100",
      "SavedBright":"100",
      "Color":"",
      "SwitchedOn":false
    },
    "listeners" : {
      "HueStatus" : {"type":"http-get", "ishub":true, "command":"http://$HueBridgeIp:$HueBridgePort/api/$HueUserName/lights", "pooltime":"3000", "poolduration":"", "queryresult" : ["$.*~","$.[*].state"], 
        "evalwrite" : [ 
          {"variable" : "Brightness", "value" : "DYNAMIK JSON.parse(\"$Result\").find(elt => elt[0] === \"$LightNumber\")[1].on ? Math.round(JSON.parse(\"$Result\").find(elt => elt[0] == \"$LightNumber\")[1].bri/2.54) : 0"},
          {"variable" : "SwitchedOn", "value" : "DYNAMIK JSON.parse(\"$Result\").find(elt => elt[0] === \"$LightNumber\")[1].on"}
   ]
        }
    },
    "switches":{
      "POWER" : {"label":"On", "listen":"SwitchedOn", "evaldo":[{"test":"DYNAMIK $Result", "then":"SWITCHON", "or":"SWITCHOFF"}]}
    },
    "sliders":{
        "BRIGHTNESS": {"label":"Brightness", "unit" : "Lux", "listen" : "Brightness", "evaldo":[{"test":true, "then":"BRIGHTNESSCHANGE", "or":""}]}
    },
    "buttons":{
        "POWER ON": {"label":"", "type":"static", "command":"", "evaldo":[{"test":true,"then":"__INITIALISE", "or":""}]},
        "POWER OFF": {"label":"", "type":"static", "command":"", "evaldo":[{"test":true,"then":"__CLEANUP", "or":""}]},
        "SWITCHON": {"label":"", "type":"http-rest", "command":"{\"verb\":\"put\",\"call\":\"http://$HueBridgeIp:$HueBridgePort/api/$HueUserName/lights/$LightNumber/state\", \"message\":{\"on\": true}}", "evalwrite":[{"variable":"SwitchedOn","value":true}, {"variable":"Brightness","value":"DYNAMIK Math.round($SavedBright)"}]},
        "SWITCHOFF": {"label":"", "type":"http-rest", "command":"{\"verb\":\"put\",\"call\":\"http://$HueBridgeIp:$HueBridgePort/api/$HueUserName/lights/$LightNumber/state\", \"message\":{\"on\": false}}", "evalwrite":[{"variable":"SwitchedOn","value":false}, {"variable":"SavedBright","value":"DYNAMIK Math.round($Brightness)"}, {"variable":"Brightness","value":0}]},
        "BRIGHTNESSCHANGE": {"label":"", "type":"http-rest", "command":"DYNAMIK \"{\\\"verb\\\":\\\"put\\\",\\\"call\\\":\\\"http://$HueBridgeIp:$HueBridgePort/api/$HueUserName/lights/$LightNumber/state\\\", \\\"message\\\":{\\\"bri\\\": \" + Math.round(254/100*$Brightness) + \"}}\""}
    },
    "directories":{
      "Colors": {"label":"Color Picker", "feeders": {
          "color":{"label":"", "commandset": [{"type":"static", "command": "[{\"color\":{\"hue\":25652,\"sat\":254, \"xy\": [ 0.674, 0.322 ], \"ct\":289}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Red.jpg\"}, {\"color\":{\"hue\":7458,\"sat\":252, \"xy\": [ 0.5958, 0.3791 ], \"ct\":153}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Orange.jpg\"}, {\"color\":{\"hue\":37458,\"sat\":149, \"xy\": [ 0.5158, 0.3721 ], \"ct\":235}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Light_Orange.jpg\"}, {\"color\":{\"hue\":65511,\"sat\":43, \"xy\": [ 0.4302, 0.3675 ], \"ct\":322}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Warm_White.jpg\"}, {\"color\":{\"hue\":34153,\"sat\":243, \"xy\": [ 0.3157, 0.33 ], \"ct\":156}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Full_White.jpg\"}, {\"color\":{\"hue\":35616,\"sat\":238, \"xy\": [ 0.3016, 0.3005 ], \"ct\":153}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Cold_White.jpg\"}, {\"color\":{\"hue\":25653,\"sat\":254, \"xy\": [ 0.4084, 0.5168 ], \"ct\":289}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Green.jpg\"}, {\"color\":{\"hue\":43110,\"sat\":252, \"xy\": [ 0.2102, 0.1248 ], \"ct\":153}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Light_Blue.jpg\"}, {\"color\":{\"hue\":46989,\"sat\":251, \"xy\": [ 0.1708, 0.0465 ], \"ct\":153}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Deep_Blue.jpg\"}, {\"color\":{\"hue\":59221,\"sat\":253, \"xy\": [ 0.5005, 0.226 ], \"ct\":443}, \"imageurl\":\"https://raw.githubusercontent.com/jac459/NeeoHueDriver/master/Resources/Col_Pink.jpg\"}]", "queryresult":"$.*", "itemtype": "tile", "itemaction":"colorSet", "itemimage":"DYNAMIK JSON.parse(\"$Result\").imageurl", "evalwrite":[{"variable":"Color","value":"DYNAMIK JSON.stringify(JSON.parse(\"$Result\").color)"}]}]},
          "colorSet":{"label":"", "commandset": [{"type":"http-rest", "command":"{\"verb\":\"put\",\"call\":\"http://$HueBridgeIp:$HueBridgePort/api/$HueUserName/lights/$LightNumber/state\", \"message\":$Color}"}]}
        }
      }
    }
  }
}
```

<br>
<br>

# MQTT Bridge to HA

I am using an MQTT bridge as part of my setup, by bridging the two mosquitto instances. It can be setup by modifying `/etc/mosquitto/mosquitto.conf` and adding the following:
```
#### HA BRIDGE #### 

connection ha_bridge
address <HA_IP>:<MQTT_PORT>
remote_username metadriver
remote_password meta
topic # out 0
try_private true
cleansession false

```
