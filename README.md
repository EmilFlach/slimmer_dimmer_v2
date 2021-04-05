# Slimmer dimmer v2
The slimmer dimmer is the best way to automate lights and still maintain a loving relationship with your family. Find out more here: https://graduation.emilflach.com/.

## General principle
The slimmer dimmer is to be used with one single light or a group of lights. These light are automated and might turn on when presence in a room is detected. Of course the light should only turn on when it's dark and maybe late in the evening it should be a bit less bright. 

All of these values are stored in an automatic state. This automatic state is updated throughout the day as brightness/weather/sleep sensors change states. Then when presence is detected, this state is applied to the lights of that room. 

Sometimes (more often than we want), these automations might be incorrect. You quickly want to correct the lights, so you can actually use the room as intended. When using the slimmer dimmer, automations will be disabled and you will gain full control of the lights again.

Lastly, when you want to give control back to the automations, you long press the slimmer dimmer and the room is in automatic state again. 

## Requirements
There are a few requirements for using the slimmer dimmer v2, you will need to following setup:
1. Home assistant
2. Zigbee2Mqtt 
3. Philips Hue lights
4. A slimmer dimmer

## Configuring Home Assistant
1. The first thing to configure is a place for our automatic state to be stored. For that we use Home Assistant's [input numbers](https://www.home-assistant.io/integrations/input_number/). You can configure these through the helpers in the GUI or configure them with YAML as follows:

```
input_number:
  bedroom_brightness:
    initial: 0
    min: 0
    max: 255
    step: 1
  bedroom_color_temp:
    initial: 360
    min: 150
    max: 500
    step: 1
```
2. Now we can configure what the brightness of the automatic state should be throughout the day. I use brightness sensors to set the input numbers. You can import this blueprint or look at the example automation at the end of the gist.

    [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgist.github.com%2FEmilFlach%2F26eeba2d836b143636cb39cdb0b74ec7)

3. Then we can use this value of the input numbers when detecting motion:

    [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgist.github.com%2FEmilFlach%2F80dd0b77689ec28f5336821a6ae14ec8)

4. Add all the automations that control the lights (such as entered bedroom) to a group. Sadly this can only be done with YAML. Do not add the automations that change the values of the input numbers, these automations should never interact with your lights directly and should be allowed to continue.
```
bedroom_automations:
  name: Bedroom automations
  entities:
    - automation.entered_bedroom
    - automation.left_bedroom
    - automation.closed_bedroom_window
    - automation.opened_bedroom_window
    - automation.wake_up_lights
    - automation.all_off_when_sleeping
    - automation.bedroom_flux_changed
```

## Configure the slimmer dimmer
At this point, your entire home assistant configuration should be ready and you can configure the slimmer dimmer. For this to work, you need to have esphome installed: https://esphome.io/guides/getting_started_command_line.html

1. Copy the example.secrets.yaml file and rename it to secrets.yaml. Then fill in the secrets in the file.

2. Configure these fields to match what you configured in home assistant. The Zigbee2Mqtt light topic is the light you want to control, this can also be a light group.
```
  device_name: "dimmer_bedroom"
  mqtt_light_topic: "zigbee2mqtt/bedroom"
  auto_brightness_id: "input_number.bedroom_brightness"
  auto_color_temp_id: "input_number.bedroom_color_temp"
  automation_group: "group.bedroom_automations"
```

3. Run the following esphome command in your console: `esphome dimmer_bedroom.yaml run`
4. Go to your home asssistant interface, go to integrations and add a new Esphome intergation. When prompted fill in your dimmer address as such `dimmer_bedroom.local`.
5. You should be up an running!

## (Optional) Add a "digital slimmer dimmer"
If you also use voice commands or the home assistant interface, you might want to have a similar slimmer dimmer principle there. If you use voice commands or the interface, you would want automations to stop until you enable them again. If desired, that behavior can be achieved using this blueprint:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgist.github.com%2FEmilFlach%2F4e70ed38a07c8921ec8c8151f7f86e2f)
