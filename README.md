# Using NEEO Remote (prior control4) with Home Assistant
Contents:
- [HA REST API](#HA-Rest-API)
  - [Media Player Driver](#media-player-driver-rest-api)
  - Light Driver
  - Generic IR Device
- [HA Emulated Hue](#emulated-hue)
<br>
There are a few ways to integrate Home Assistant to the NEEO remote with the help of [*metadriver*](https://github.com/jac459/metadriver). The three popular methods are:
- MQTT [Advanced]
- HA Rest API (SHOWN) [Moderate Difficulty for less savy users]
- HA Emulated Hue (SHOWN) [Easiest]

Due to my lack of expertise with MQTT, I will be showing you how you can integrate HA through Emulated Hue and Rest API, I've left *templates* in the repo files as generic drivers to run your devices.
<br>

## HA Rest API

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

# Media Player Driver (Rest API)
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
