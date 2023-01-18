# HomeAssistant - Peugeot Integration

## About
I adjusted the original https://github.com/Flodu31/HomeAssistant-PeugeotIntegration configs to my citroën. 
I also add icon_templates, so from now icon are changing based on some conditions.
I also added buttons to start/stop preconditioning. 

Here is a small description to do the integration between your home assistant, and your Citroën vehicle, to have all information in a dashboard:


## Pre-requirements

*Use [this full external tutorial](https://return2.net/opel-peugeot-electric-vehicle-set-charging-threshold-limit/) that describes the entire installation process of this integration, including the PSA Car Controller configuration.*

To do this, you need:

- A running Home Assistant with Lovelace installed: https://www.home-assistant.io/lovelace/
- The psa_car_controller (on a dedicated machine or on your home assistant server): https://github.com/flobz/psa_car_controller OR The psa_car_controller plugin available on the Flobz's repo: https://github.com/flobz/psacc-ha/tree/main/psacc-ha

The goal will be to retrive information, from the URL http://yourIP:5000/get_vehicleinfo/yourVIN and http://yourIP:5000/charge_control?vin=yourVIN.
It is a JSON, that will be parsed by Home Assistant.

This script will evolve, depending of releases of the psa_car_controller. And my time :)


## Configuration

Edit the file **/config/sensor.yaml** and add the following code, by replacing the URL with your own:

```yaml
# citroen ec4  
- platform: rest
  name: citroen_ec4
  resource: http://IPofTheSoftware:5000/get_vehicleinfo/YourVIN?from_cache=1
  scan_interval: 60
  timeout: 30
  value_template: 'OK'
  json_attributes:
   - energy
   - timed_odometer
   - battery
   - preconditionning
- platform: template
  sensors:
    ec4_clim_status:
      friendly_name: "Preconditioning status"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'preconditionning')['air_conditioning']['status'] }}"
      icon_template: mdi:thermometer-lines
    ec4_battery_voltage:
      friendly_name: "Battery Voltage"
      unit_of_measurement: "V"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'battery')['voltage'] * 4 }}"
      icon_template: mdi:flash-triangle-outline
    ec4_battery_level:
      friendly_name: "Battery"
      unit_of_measurement: "%"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'energy')[0]['level'] }}"
      # <15=empty  15-45=low 45-75=medium  >75=high
      icon_template: >-
        {% set battery_level = states('sensor.ec4_battery_level')|float -%}
        {% if battery_level <= 15 %}
          mdi:battery-outline
        {% elif battery_level > 15 and battery_level <= 45 %}
          mdi:battery-low
        {% elif battery_level > 45 and battery_level <= 75 %}
          mdi:battery-medium
        {% elif battery_level > 75 %}
          mdi:battery-high
        {% endif %}  
    ec4_battery_autonomy:
      friendly_name: "Autonomy"
      unit_of_measurement: "km"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'energy')[0]['autonomy'] }}"
      icon_template: mdi:map-marker-distance
     # Uncomment for miles instead of km
     #ec4_battery_autonomy:
      #friendly_name: "Autonomy"
      #unit_of_measurement: "m"    
      #value_template: "{{ ((state_attr('sensor.citroen_ec4', 'energy')[0]['autonomy']) / 1.609) | round(0)}}"
      #icon_template: mdi:map-marker-distance
    ec4_charging_status:
      friendly_name: "Charging Status"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'energy')[0]['charging']['status'] }}"
      icon_template: >-
        {% set battery_level = states('sensor.ec4_battery_level')|float -%}
        {% set charging = iif(is_state('switch.ec4_charging_status', 'InProgress'),'-charging', '') -%}
        {% if battery_level <= 15 -%}
          mdi:battery{{charging}}-outline
        {% elif battery_level > 15 and battery_level <= 45 -%}
          mdi:battery-charging-low
        {% elif battery_level > 45 and battery_level <= 75 -%}
          mdi:battery{{charging}}-medium
        {% elif battery_level > 75 -%}
          mdi:battery{{charging}}-high
        {% endif -%}
    ec4_mileage:
      friendly_name: "Mileage"
      unit_of_measurement: "km"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'timed_odometer')['mileage'] }}"
      icon_template: mdi:counter
      # Uncomment for miles instead of km
#     ec4_mileage:
#      friendly_name: "Mileage"
#      unit_of_measurement: "m"
#      value_template: "{{ ((state_attr('sensor.citroen_ec4', 'timed_odometer')['mileage']) / 1.609) | round(2)}}"
#      icon_template: mdi:counter
    ec4_charging_remaining_time:
      friendly_name: "Remaining time"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'energy')[0]['charging']['remaining_time'] }}"
      icon_template: mdi:clock-time-four-outline
    ec4_charging_plugged:
      friendly_name: "Plugged"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'energy')[0]['charging']['plugged'] }}"
      icon_template: >-
        mdi:power-plug{{ iif(is_state('switch.ec4_charging_plugged', 'True'),'', '-off') }}-outline
    ec4_charging_mode:
      friendly_name: "Charging mode"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'energy')[0]['charging']['charging_mode'] }}"
      icon_template: >-
          {% if is_state('sensor.ec4_charging_mode', 'Slow') -%}
            mdi:ev-plug-type2
          {% elif is_state('sensor.ec4_charging_mode', 'Quick') -%}
            mdi:ev-plug-ccs2
          {% else -%}
            mdi:power-plug-off-outline
          {% endif -%}
    ec4_charging_rate:
      friendly_name: "Charging rate"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'energy')[0]['charging']['charging_rate'] }}"
      unit_of_measurement: "km/h"
      icon_template: mdi:chart-bar
      # Uncomment for miles instead of km
#     ec4_charging_rate:
#      friendly_name: "Charging rate"
#      unit_of_measurement: "m/h"
#      value_template: "{{ ((state_attr('sensor.citroen_ec4', 'energy')[0]['charging']['charging_rate']) / 1.609) | round(2)}}"
#      icon_template: mdi:chart-bar
    ec4_charging_updated_at:
      friendly_name: "Charging status updated at"
      value_template: "{{ state_attr('sensor.citroen_ec4', 'energy')[0]['updated_at'] }}"
      icon_template: mdi:calendar
# citroen ec4 charge_control
- platform: rest
  name: citroen_ec4_charge_control
  resource: http://IPofTheSoftware:5000/charge_control?vin=YourVIN
  scan_interval: 60
  timeout: 30
  value_template: 'OK'
  json_attributes:
   - _next_stop_hour
   - percentage_threshold
- platform: template
  sensors:
    ec4_stop_hour:
      friendly_name: "Charging postponed until"
      value_template: "{{ state_attr('sensor.citroen_ec4_charge_control', '_next_stop_hour') }}"
      icon_template: mdi:clock-time-four-outline
    ec4_threshold:
      friendly_name: "Charging threshold"
      unit_of_measurement: "%"
      value_template: "{{ state_attr('sensor.citroen_ec4_charge_control', 'percentage_threshold') }}"
      icon_template: mdi:percent-outline
```

Edit the file **/config/configuration.yaml** and add the following code, by replacing the URL with your own:

```yaml
rest_command:
  # ec4 WakeUp command
  ec4_wakeup:
    url: "http://IPofTheSoftware:5000/wakeup/YourVIN"
  # ec4 set custom threshold command
  ec4_change_charge_threshold:
    url: "http://IPofTheSoftware:5000/charge_control?vin=YourVIN&percentage={{ states('input_number.ec4_charge_threshold_slider') | int }}"
    method: GET
  # activate preconditioning
  ec4_activate_preconditioning:
    url: "http://IPofTheSoftware:5000/preconditioning/YourVIN/1"
    method: GET
  # stop preconditioning  
  ec4_stop_preconditioning:
    url: "http://IPofTheSoftware:5000/preconditioning/YourVIN/0"
    method: GET

# slider to adjust new charging threshold
input_number:
    ec4_charge_threshold_slider:
      name: Adjust new charging threshold
      initial: 80
      min: 50
      max: 100
      step: 1
      unit_of_measurement: "%"
  
# buttons to set charging threshold or activate/stop preconditioning      
input_button:
    ec4_apply_charge_threshold_button:
      name: Set new charging threshold
      icon: mdi:battery-charging
    ec4_activate_preconditioning_button:
      name: Activate preconditioning
      icon: mdi:play
    ec4_stop_preconditioning_button:
      name: Stop preconditioning
      icon: mdi:stop
```

Now add some automations to buttons:
```yaml
alias: ec4 - activate preconditioning
description: ""
trigger:
  - platform: state
    entity_id:
      - input_button.ec4_activate_preconditioning_button
condition: []
action:
  - service: rest_command.ec4_activate_preconditioning
    data: {}
mode: single
```
```yaml
alias: ec4 - stop - preconditioning
description: ""
trigger:
  - platform: state
    entity_id:
      - input_button.ec4_stop_preconditioning_button
condition: []
action:
  - service: rest_command.ec4_stop_preconditioning
    data: {}
mode: single
```
```yaml
alias: ec4 - apply - charging threshold
description: ""
trigger:
  - platform: state
    entity_id:
      - input_button.ec4_apply_charge_threshold_button
condition: []
action:
  - service: rest_command.ec4_change_charging_threshold
    data: {}
mode: single

```

Refreshing next threshold time needs to be handled differently, that's why is not mention here. But it will be :-) 

Restart the Home assistant.


## User Interface / Dashboard

You can now add a beautiful dashboard, to see your values:

```yaml
type: entities
entities:
  - entity: sensor.ec4_mileage
    secondary_info: last-updated
  - entity: sensor.ec4_battery_level
    secondary_info: last-updated
  - entity: sensor.ec4_battery_voltage
    secondary_info: last-updated
  - entity: sensor.ec4_battery_autonomy
    secondary_info: last-updated
  - entity: sensor.ec4_charging_plugged
    secondary_info: last-updated
  - entity: sensor.ec4_charging_status
    secondary_info: last-updated
  - entity: sensor.ec4_charging_remaining_time
    secondary_info: last-updated
  - entity: sensor.ec4_charging_mode
    secondary_info: last-updated
  - entity: sensor.ec4_charging_rate
    secondary_info: last-updated
  - entity: sensor.ec4_charging_updated_at
    secondary_info: last-updated
  - entity: sensor.ec4_stop_hour
    secondary_info: last-updated
  - entity: sensor.ec4_threshold
    secondary_info: last-updated
  - entity: input_number.ec4_charge_threshold_slider
  - entity: input_button.ec4_apply_charge_threshold_button
  - entity: sensor.ec4_clim_status
    secondary_info: last-updated
  - entity: input_button.ec4_activate_preconditioning_button
    secondary_info: last-updated
  - entity: input_button.ec4_stop_preconditioning_button
    secondary_info: last-updated
title: Citroën ë-C4
show_header_toggle: false
state_color: true
header:
  type: picture
  image: /local/images/ec4-512p.jpg
  tap_action:
    action: none
  hold_action:
    action: none
```

Don't hesitate if you have any remarks/suggestions/questions :)
