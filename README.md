# Using NEEO Remote (prior control4) with Home Assistant
Contents:
- [HA REST API](#HA-Rest-API)
  - [Media Player Driver](#media-player-driver-rest-api)
  - [Light Driver](#light-driver-rest-api)
  - [Generic IR Device](#generic-ir-rest-api-broadlink)
- [HA Emulated Hue](#emulated-hue)
- [MQTT Bridge to HA](#mqtt-bridge-to-ha)
- [How to Install Drivers](#install-drivers)
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

The driver can be found under files as `EmulatedHue_Light.json`
`wget https://github.com/Danirali/neeo-metadriver-homeassistant/blob/main/EmulatedHue_Light.json`
<br>
<br>

# MQTT Bridge to HA

I am using an MQTT bridge as part of my setup, by bridging the two mosquitto instances. It can be setup by modifying `/etc/mosquitto/mosquitto.conf` and adding the following:
```
#### HA BRIDGE #### 

connection ha_bridge
address <HA_IP>:<MQTT_PORT>
remote_username <MQTT_USERNAME>
remote_password <MQTT_PASSWORD>
topic # out 0
try_private true
cleansession false

```
 <br>
 <br>

 ## Install Drivers

To install these drivers to your metadriver, locate to your meta install and navigate to the `active` folder usually `/home/YOUR_USER/meta/active`. You then need to move the driver into this folder. This can be done by downloading the file to the folder by using `wget` on Linux or if you are using another PC to the one running meta you can use `scp`.

Examples:
- Linux
  `wget https://github.com/Danirali/neeo-metadriver-homeassistant/blob/main/EmulatedHue_Light.json`
- MacOS/Windows to Linux
  `scp ~/Downloads/driver.json user@ip-address:/home/USER/meta/active`
 
