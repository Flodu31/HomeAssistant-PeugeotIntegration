# HomeAssistant - Peugeot Integration
Here is a small description to do the integration between your home assistant, and your Peugeot vehicle, to have all information in a dashboard:

![image](https://user-images.githubusercontent.com/15648175/113413427-f9dad380-93ba-11eb-848b-1a290904a242.png)

To do this, you need:
- A running Home Assistant with Lovelace installed: https://www.home-assistant.io/lovelace/
- The psa_car_controller (on a dedicated machine or on your home assistant server): https://github.com/flobz/psa_car_controller

The goal will be to retrive information, from the URL http://yourIP:5000/get_vehicleinfo/yourVIN and http://yourIP:5000/charge_control?vin=yourVIN.
It is a JSON, that will be parsed by Home Assistant.

This script will evolve, depending of releases of the psa_car_controller. And my time :)

Edit the file **/config/sensor.yaml** and add the following code, by replacing the URL with your own:

```yaml
# Peugeot e2008  
  - platform: rest
    name: peugeot_e2008
    resource: http://IPofTheSoftware:5000/get_vehicleinfo/YourVIN
    scan_interval: 60
    timeout: 30
    value_template: 'OK'
    json_attributes:
     - energy
     - timed_odometer
     - battery
  - platform: template
    sensors:
      e2008_battery_voltage:
        friendly_name: "Battery Voltage"
        unit_of_measurement: "V"
        value_template: '{{ states.sensor.peugeot_e2008.attributes["battery"]["voltage"] * 4 }}'
      e2008_battery_level:
        friendly_name: "Battery"
        unit_of_measurement: "%"
        value_template: '{{ states.sensor.peugeot_e2008.attributes["energy"][1]["level"] }}'
      e2008_battery_autonomy:
        friendly_name: "Autonomy"
        unit_of_measurement: "km"
        value_template: '{{ states.sensor.peugeot_e2008.attributes["energy"][1]["autonomy"] }}'
      e2008_charging_status:
        friendly_name: "Charging Status"
        value_template: '{{ states.sensor.peugeot_e2008.attributes["energy"][1]["charging"]["status"] }}'
      e2008_mileage:
        friendly_name: "Mileage"
        unit_of_measurement: "km"
        value_template: '{{ states.sensor.peugeot_e2008.attributes["timed_odometer"]["mileage"] }}'
# Peugeot e2008 charge_control
  - platform: rest
    name: peugeot_e2008_charge_control
    resource: http://IPofTheSoftware:5000/charge_control?vin=YourVIN
    scan_interval: 60
    timeout: 30
    value_template: 'OK'
    json_attributes:
     - _next_stop_hour
     - percentage_threshold
  - platform: template
    sensors:
      e2008_stop_hour:
        friendly_name: "Next Stop Time"
        value_template: '{{ states.sensor.peugeot_e2008_charge_control.attributes["_next_stop_hour"]}}'
      e2008_threshold:
        friendly_name: "Threshold"
        unit_of_measurement: "%"
        value_template: '{{ states.sensor.peugeot_e2008_charge_control.attributes["percentage_threshold"] }}'
```

Edit the file **/config/configuration.yaml** and add the following code, by replacing the URL with your own:

```
# e2008 communication
switch:
  - platform: command_line
    switches:
      e2008_change_threshold:
        command_on: curl -s "http://IPofTheSoftware:5000/charge_control?vin=YourVIN&percentage=80"
        command_off: curl -s "http://IPofTheSoftware:5000/charge_control?vin=YourVIN&percentage=100"
      e2008_change_charge_hour:
        command_on: curl -s "http://IPofTheSoftware:5000/charge_control?vin=YourVIN&hour=5&minute=0"
        command_off: curl -s "http://IPofTheSoftware:5000/charge_control?vin=YourVIN&hour=0&minute=0"
      e2008_clim:
        command_on: curl -s "http://IPofTheSoftware:5000/preconditioning/YourVIN/1"
        command_off: curl -s "http://IPofTheSoftware:5000/preconditioning/YourVIN/0"
       
# e2008 WakeUp
rest_command:
  e2008_wakeup:
    url: "http://IPofTheSoftware:5000/wakeup/YourVIN"
```
The first part is to change values and the second part is to wake up the vehicle, every morning at 6 AM:

```
- id: '1617352487'
  alias: WakeUp e2008
  description: ''
  trigger:
  - platform: time
    at: '6:00:00'
  condition: []
  action:
  - service: rest_command.e2008_wakeup
  mode: single
```

Restart the Home assistant.
You should now be able to see these entities:

![image](https://user-images.githubusercontent.com/15648175/113413669-7a99cf80-93bb-11eb-8744-78edec8c92e2.png)

In my **customize.yaml** file, I also added this:

```
# Add an entry for each entity that you want to overwrite.
  switch.e2008_change_threshold:
    assumed_state: false
  switch.e2008_change_charge_hour:
    assumed_state: false
  switch.e2008_clim:
    assumed_state: false
```

You can now add a beautiful dashboard, to see your values:

```
type: entities
entities:
  - entity: sensor.e2008_mileage
    secondary_info: last-updated
  - entity: sensor.e2008_battery_level
    secondary_info: last-updated
  - entity: sensor.e2008_battery_voltage
    secondary_info: last-updated
  - entity: sensor.e2008_battery_autonomy
    secondary_info: last-updated
  - entity: sensor.e2008_charging_status
    secondary_info: last-updated
  - entity: sensor.e2008_stop_hour
    secondary_info: last-updated
  - entity: sensor.e2008_threshold
    secondary_info: last-updated
  - entity: switch.e2008_change_threshold
    name: Change Threshold
    secondary_info: last-updated
  - entity: switch.e2008_change_charge_hour
    secondary_info: last-updated
    name: Charge before 5AM
  - entity: switch.e2008_clim
    secondary_info: last-updated
    name: Activate Clim
  - entity: automation.wakeup_e2008
    secondary_info: last-triggered
title: Peugeot e2008
header:
  type: picture
  image: /local/images/e2008.png
  tap_action:
    action: none
  hold_action:
    action: none
show_header_toggle: false
state_color: false
```
To have a more viewable view of the battery, like this:

![image](https://user-images.githubusercontent.com/15648175/113413772-b6349980-93bb-11eb-87a5-5e4b80df3ca3.png)

Use the following code in your dashboard:

```
type: gauge
entity: sensor.e2008_battery_level
min: 0
max: 100
name: e2008 Battery
severity:
  green: 60
  yellow: 40
  red: 20
```

Don't hesitate if you have any remarks/suggestions/questions :)
