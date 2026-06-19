# WLANThermo for Home Assistant

---

Discontinued due to availablity of [HACS repo](https://github.com/WLANThermo-nano/homeassistant)

---

The WLANThermo bbq thermometer (https://wlanthermo.de/) uses a local wifi and optional MQTT server to push the temperature changes to.

## Setup MQTT in WLANThermo
Link your WLANThermo to the same MQTT server which Home Assistant is using. You can do this easily on the provided web interface.


## Home Assistant Configuration

### Update to new template integration (no more legacy templates)

### Sensor
First we add a sensor containing all information.

Adjust your MQTT topic appropriately, so change the topic prefix `WLanThermo/NANO-2a21b5` to your modelname.

```yaml
mqtt:
# Basic sensor, read everything from MQTT
  sensor:
    - name: "WLANThermo"
      # change topic to your modelname!
      state_topic: "WLanThermo/NANO-2a21b5/status/data"
      value_template: "{{ value_json.system.soc }}"
      # change topic to your modelname!
      json_attributes_topic: "WLanThermo/NANO-2a21b5/status/data"
      json_attributes_template: "{{ value_json | tojson }}"
      #device_class: battery
      #unit_of_measurement: "%"
      expire_after: 60

# Template sensor per channel
template:
  - sensor:
    # wlanthermo WIFI
      - name: "WLANThermo Signalstärke"
        unique_id: "wlanthermo_signalstaerke"
        state: "{{ state_attr('sensor.wlanthermo', 'system')['rssi'] }}"
        availability: "{{ has_value('sensor.wlanthermo') }}"
        unit_of_measurement: "dBm"
        device_class: signal_strength

    # wlanthermo Channel 1
      - name: "BBQ Kanal 1"
        unique_id: 'wlanthermo_channel_1_all'
        state: "{{ state_attr('sensor.wlanthermo', 'channel')[0] }}"
        availability: "{{ has_value('sensor.wlanthermo') }}"
        
      - name: "BBQ Kanal 1 Temperatur"
        unique_id: 'wlanthermo_channel_1_temp'
        state: "{{ state_attr('sensor.wlanthermo', 'channel')[0]['temp'] }}"
        unit_of_measurement: "°C"
        device_class: temperature

    # wlanthermo Channel 2
      - name: "BBQ Kanal 2"
        unique_id: 'wlanthermo_channel_2_all'
        state: "{{ state_attr('sensor.wlanthermo', 'channel')[1] }}"
        availability: "{{ has_value('sensor.wlanthermo') }}"
        
      - name: "BBQ Kanal 2 Temperatur"
        unique_id: 'wlanthermo_channel_2_temp'
        state: "{{ state_attr('sensor.wlanthermo', 'channel')[1]['temp'] }}"
        unit_of_measurement: "°C"
        device_class: temperature

    # add further channels here...
```



### Input Number
Now we need to define input_number elements, so that we are able to control a slider in the UI.

```yaml
input_number:
  wlanthermo_channel_1_min:
      name: Channel 1 min
      min: 0
      max: 100
      step: 1
      unit_of_measurement: °C
      icon: mdi:thermometer
  wlanthermo_channel_1_max:
      name: Channel 1 max
      min: 0
      max: 150
      step: 1
      unit_of_measurement: °C
      icon: mdi:thermometer
  wlanthermo_channel_2_min:
      name: Channel 2 min
      min: 0
      max: 100
      step: 1
      unit_of_measurement: °C
      icon: mdi:thermometer
  wlanthermo_channel_2_max:
      name: Channel 2 max
      min: 0
      max: 150
      step: 1
      unit_of_measurement: °C
      icon: mdi:thermometer
```

### Automations (updating sensor values)
In the automations you have to define how to set the defined input_number elements (we want to use the values which are coming from MQTT) and what to do if someone changes the slider in the UI (we want to update the WLANThermo).

```yaml
# First automation: set Home Assistant sensor value from MQTT messages, so we
# update if someone changes the values on web interface
- id: '1565774399160'
  alias: WLANThermo update channels min/max from MQTT
  trigger:
  - entity_id: sensor.wlanthermo
    platform: state
  condition: []
  action:
  - data_template:
      entity_id: input_number.wlanthermo_channel_1_min
      value: '{{ state_attr(''sensor.wlanthermo'', ''channel'').0.min }}'
    service: input_number.set_value
  - data_template:
      entity_id: input_number.wlanthermo_channel_1_max
      value: '{{ state_attr(''sensor.wlanthermo'', ''channel'').0.max }}'
    service: input_number.set_value
  - data_template:
      entity_id: input_number.wlanthermo_channel_2_min
      value: '{{ state_attr(''sensor.wlanthermo'', ''channel'').1.min }}'
    service: input_number.set_value
  - data_template:
      entity_id: input_number.wlanthermo_channel_2_max
      value: '{{ state_attr(''sensor.wlanthermo'', ''channel'').1.max }}'
    service: input_number.set_value

# automation per input_number: update value set in Lovelace UI on WLANThermo
- id: '1565867288673'
  alias: WLANThermo set channel 1 min
  trigger:
  - entity_id: input_number.wlanthermo_channel_1_min
    platform: state
  condition: []
  action:
  - service: mqtt.publish
    data_template:
      topic: WLanThermo/NANO-2a21b5/set/channels
      retain: true
      payload: '{{ ''{"number":1,"min":''+states(''input_number.wlanthermo_channel_1_min'')+''}''
        }}'

# automation per input_number: update value set in Lovelace UI on WLANThermo
- id: '1565867288674'
  alias: WLANThermo set channel 1 max
  trigger:
  - entity_id: input_number.wlanthermo_channel_1_max
    platform: state
  condition: []
  action:
  - service: mqtt.publish
    data_template:
      topic: WLanThermo/NANO-2a21b5/set/channels
      retain: true
      payload: '{{ ''{"number":1,"max":''+states(''input_number.wlanthermo_channel_1_max'')+''}''
        }}'

# automation per input_number: update value set in Lovelace UI on WLANThermo
- id: '1565867288675'
  alias: WLANThermo set channel 2 min
  trigger:
  - entity_id: input_number.wlanthermo_channel_2_min
    platform: state
  condition: []
  action:
  - service: mqtt.publish
    data_template:
      topic: WLanThermo/NANO-2a21b5/set/channels
      retain: true
      payload: '{{ ''{"number":2,"min":''+states(''input_number.wlanthermo_channel_2_min'')+''}''
        }}'

# automation per input_number: update value set in Lovelace UI on WLANThermo
- id: '1565867288676'
  alias: WLANThermo set channel 2 max
  trigger:
  - entity_id: input_number.wlanthermo_channel_2_max
    platform: state
  condition: []
  action:
  - service: mqtt.publish
    data_template:
      topic: WLanThermo/NANO-2a21b5/set/channels
      retain: true
      payload: '{{ ''{"number":2,"max":''+states(''input_number.wlanthermo_channel_2_max'')+''}''
        }}'

# add additional channels here...
```

### Lovelace configuration
![Screenshot Lovelace UI WLANThermo](/screenshot.png?raw=true "WLANThermo Lovelace")
We add a seperate View for this:
```yaml
theme: Backend-selected
title: BBQ
path: bbq
icon: mdi:pig
badges: []
cards:
  - type: conditional
    conditions:
      - condition: state
        entity: sensor.wlanthermo
        state_not: unavailable
    card:
      type: vertical-stack
      cards:
        - type: gauge
          entity: sensor.bbq_kanal_1_temperatur
          min: 0
          max: 350
          severity:
            green: 90
            yellow: 115
            red: 200
          name: BBQ Kanal 1
        - type: entities
          entities:
            - entity: input_number.wlanthermo_channel_1_min
            - entity: input_number.wlanthermo_channel_1_max
  - type: conditional
    conditions:
      - condition: state
        entity: sensor.wlanthermo
        state_not: unavailable
    card:
      type: vertical-stack
      cards:
        - type: gauge
          entity: sensor.bbq_kanal_2_temperatur
          min: 0
          max: 350
          severity:
            green: 0
            yellow: 0
            red: 0
          name: BBQ Kanal 2
        - type: entities
          entities:
            - entity: input_number.wlanthermo_channel_2_min
            - entity: input_number.wlanthermo_channel_2_max
  - type: conditional
    conditions:
      - condition: state
        entity: sensor.wlanthermo
        state_not: unavailable
    card:
      type: history-graph
      title: BBQ Temperaturverlauf
      entities:
        - entity: sensor.bbq_kanal_1_temperatur
        - entity: sensor.bbq_kanal_2_temperatur
      hours_to_show: 4
```
