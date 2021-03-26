# HomeAssistant - Peugeot Integration
Here is a small description to do the integration between your home assistant, and your Peugeot vehicle, to have all information in a dashboard:

![image](https://user-images.githubusercontent.com/15648175/112605213-58321000-8e17-11eb-862b-97d2704cc2d4.png)

To do this, you need:
- A running Home Assistant
- The psa_car_controller (on a dedicated machine or on your home assistant server): https://github.com/flobz/psa_car_controller

The goal will be to retrive information, from the URL http://yourIP:5000/get_vehicleinfo/yourVIN.
It is a JSON, that will be parsed by Home Assistant.

This script will evolve, depending of releases of the psa_car_controller;

**/config/sensor.yaml**:

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
        value_template: '{{ states.sensor.peugeot_e2008.attributes["battery"]["voltage"] }}'
      e2008_battery_level:
        friendly_name: "Battery"
        unit_of_measurement: "%"
        value_template: '{{ states.sensor.peugeot_e2008.attributes["energy"][0]["level"] }}'
      e2008_battery_autonomy:
        friendly_name: "Autonomy"
        unit_of_measurement: "km"
        value_template: '{{ states.sensor.peugeot_e2008.attributes["energy"][0]["autonomy"] }}'
      e2008_charging_status:
        friendly_name: "Charging Status"
        value_template: '{{ states.sensor.peugeot_e2008.attributes["energy"][0]["charging"]["status"] }}'
      e2008_mileage:
        friendly_name: "Mileage"
        unit_of_measurement: "km"
        value_template: '{{ states.sensor.peugeot_e2008.attributes["timed_odometer"]["mileage"] }}'
```
