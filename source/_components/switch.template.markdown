---
layout: page
title: "Template Switch"
description: "Instructions how to integrate Template Switches into Home Assistant."
date: 2016-02-07 07:00
sidebar: true
comments: false
sharing: true
footer: true
ha_category: Switch
ha_release: 0.13
ha_iot_class: "Local Push"
logo: home-assistant.png
---

The `template` platform creates switches that combines components.

For example, if you have a garage door with a toggle switch that operates the
motor and a sensor that allows you know whether the door is open or closed,
you can combine these into a switch that knows whether the garage door is open
or closed.

This can simplify the GUI and make it easier to write automations. You can mark
the components you have combined as `hidden` so they don't appear themselves.

To enable Template Switches in your installation, add the following to your
`configuration.yaml` file:

{% raw %}
```yaml
# Example configuration.yaml entry
switch:
  - platform: template
    switches:
      skylight:
        value_template: "{{ is_state('sensor.skylight', 'on') }}"
        turn_on:
          service: switch.turn_on
          data:
            entity_id: switch.skylight_open
        turn_off:
          service: switch.turn_on
          data:
            entity_id: switch.skylight_close
```
{% endraw %}

{% configuration %}
  switches:
    description: List of your switches.
    required: true
    type: map
    keys:
      friendly_name:
        description: Name to use in the frontend.
        required: false
        type: string
      entity_id:
        description: Add a list of entity IDs so the switch only reacts to state changes of these entities. This will reduce the number of times the switch will try to update its state.
        required: false
        type: [string, list]
      value_template:
        description: Defines a template to set the state of the switch.
        required: true
        type: template
      turn_on:
        description: Defines an action to run when the switch is turned on.
        required: true
        type: action
      turn_off:
        description: Defines an action to run when the switch is turned off.
        required: true
        type: action
{% endconfiguration %}

## {% linkable_title Considerations %}

If you are using the state of a platform that takes extra time to load, the
Template Switch may get an `unknown` state during startup. This results
in error messages in your log file until that platform has completed loading.
If you use `is_state()` function in your template, you can avoid this situation.
For example, you would replace
{% raw %}`{{ states.switch.source.state == 'on' }}`{% endraw %}
with this equivalent that returns `true`/`false` and never gives an unknown
result:
{% raw %}`{{ is_state('switch.source', 'on') }}`{% endraw %}

## {% linkable_title Examples %}

In this section you find some real life examples of how to use this switch.

### {% linkable_title Copy Switch %}

This example shows a switch that copies another switch.

{% raw %}
```yaml
switch:
  - platform: template
    switches:
      copy:
        value_template: "{{ is_state('switch.source', 'on') }}"
        turn_on:
          service: switch.turn_on
          data:
            entity_id: switch.source
        turn_off:
          service: switch.turn_off
          data:
            entity_id: switch.source
```
{% endraw %}

### {% linkable_title Toggle Switch %}

This example shows a switch that takes its state from a sensor, and toggles
a switch.

{% raw %}
```yaml
switch:
  - platform: template
    switches:
      blind:
        friendly_name: "Blind"
        value_template: "{{ is_state_attr('switch.blind_toggle', 'sensor_state', 'on') }}"
        turn_on:
          service: switch.toggle
          data:
            entity_id: switch.blind_toggle
        turn_off:
          service: switch.toggle
          data:
            entity_id: switch.blind_toggle
```
{% endraw %}

### {% linkable_title Sensor and Two Switches %}

This example shows a switch that takes its state from a sensor, and uses two
momentary switches to control a device.

{% raw %}
```yaml
switch:
  - platform: template
    switches:
      skylight:
        friendly_name: "Skylight"
        value_template: "{{ is_state('sensor.skylight.state', 'on') }}"
        turn_on:
          service: switch.turn_on
          data:
            entity_id: switch.skylight_open
        turn_off:
          service: switch.turn_on
          data:
            entity_id: switch.skylight_close
```
{% endraw %}

### {% linkable_title Change The Icon %}

This example shows how to change the icon based on the day/night cycle.

{% raw %}
```yaml
switch:
  - platform: template
    switches:
      garage:
        value_template: "{{ is_state('cover.garage_door', 'on') }}"
        turn_on:
          service: cover.open_cover
          data:
            entity_id: cover.garage_door
        turn_off:
          service: cover.close_cover
          data:
            entity_id: cover.garage_door
        icon_template: >-
          {% if is_state('cover.garage_door', 'open') %}
            mdi:garage-open
          {% else %}
            mdi:garage
          {% endif %}
```
{% endraw %}

### {% linkable_title Change The Entity Picture %}

This example shows how to change the entity picture based on the day/night cycle.

{% raw %}
```yaml
switch:
  - platform: template
    switches:
      garage:
        value_template: "{{ is_state('cover.garage_door', 'on') }}"
        turn_on:
          service: cover.open_cover
          data:
            entity_id: cover.garage_door
        turn_off:
          service: cover.close_cover
          data:
            entity_id: cover.garage_door
        entity_picture_template: >-
          {% if is_state('cover.garage_door', 'open') %}
            /local/garage-open.png
          {% else %}
            /local/garage-closed.png
          {% endif %}
```
{% endraw %}
