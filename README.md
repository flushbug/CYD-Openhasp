# CYD-Openhasp

Small project to collect https://www.openhasp.com/ based Screens on cheap EPS32 Display "CYD" and Home Assistant.
Communication is done via MQTT (has to be installed/configured on HA).

Example : Display of Home energy values + Control of PV based EV Charge Controller http://evcc.io/

![IMG_6280](https://github.com/user-attachments/assets/fb8e2cf4-05e3-47e7-a053-e1133fd6aaa4)

Needed configuration Steps :

1. get a "CYD" like ESP32-2432S028. See https://www.openhasp.com/0.7.0/hardware/sunton/esp32-8048s0xx/ for more screen sizes.
2. Install openhasp (Serial flasher over USB out of the browser. Details on https://www.openhasp.com/0.7.0/firmware/esp32/
3. Connect via Http/Browser to openhasp "plate" device and configure/calibrate the display
4. Configure MQTT on openhasp.
(5.) In case you are running EVCC somewhere and wish to control it, also establish EVCC->MQTT connection
6. replace/modify pages.json (example https://github.com/flushbug/CYD-Openhasp/blob/main/pages.jsonl)
7. To connect to HAOS, install openhasp Integration.
8. Add following code to your configuration.yaml in HAOS and adjust the sensor values to your needs/setup
The "obj" line will update your HAOS Sensor values on the Display.
```
openhasp:
  plate:
    objects:
      - obj: "p1b3"  # Grid Power
        properties:
          "text": '{{ states("sensor.leistungsaufnahme_stromnetz") }}W'
      - obj: "p1b5"  # Home Power
        properties:
          "text": '{{ states("sensor.homeconsumption_leistung_ksem") }}W'
      - obj: "p1b7"  # PV Power
        properties:
          "text": '{{ states("sensor.pv_leistung_gesamt") }}W'
      - obj: "p1b9"  # temperature label on all pages
        properties:
          "text": '{{ states("sensor.ladezustand_pv_batterie") }}%'
      - obj: "p1b11"  # temperature label on all pages
        properties:
          "text": '{{ states("sensor.ladeleistung_wallbox") }}W'

mqtt: #connection to EVCC
  - sensor:
      name: "Leistungsaufnahme Hausnetz"
      state_topic: "evcc/site/homePower"
      unit_of_measurement: "W"
      
  - sensor:
      name: "PV Leistung Gesamt"
      state_topic: "evcc/site/pvPower"
      unit_of_measurement: "W"

  - sensor:
      name: "PV Leistung Dach"
      state_topic: "evcc/site/pv/1/power"
      unit_of_measurement: "W"

  - sensor:
      name: "PV Leistung Garage"
      state_topic: "evcc/site/pv/2/power"
      unit_of_measurement: "W"

  - sensor:
      name: "Ladezustand Pv Batterie"
      state_topic: "evcc/site/batterySoc"
      unit_of_measurement: "%"
      
  - sensor:
      name: "Batterieleistung"
      state_topic: "evcc/site/batteryPower"
      unit_of_measurement: "W"

  - sensor:
      name: "Leistungsaufnahme Stromnetz"
      state_topic: "evcc/site/gridPower"
      unit_of_measurement: "W"
      
  - sensor:
      name: "Ladeleistung Wallbox"
      state_topic: "evcc/loadpoints/1/chargePower"
      unit_of_measurement: "W"

  - sensor:
      name: "Lademodus Wallbox"
      state_topic: "evcc/loadpoints/1/mode"
      json_attributes_topic: "evcc/loadpoints/1/mode"
      json_attributes_template: >
        {"chargemode":"{{value}}"}

  - sensor: #capture EVCC buttonstate on Openhasp Display
      name: "openhaspEVCCmode"
      state_topic: "hasp/plate/state/p1b12"
      json_attributes_topic: "hasp/plate/state/p1b12"
      json_attributes_template: >
        {"chargemode":"{{value_json.val}}"} 
```
9. Remote control of http://evcc.io/ charging mode is done via HomeAssistant Automations (Openhasp->HomeAssistant->EVCC)
Create a new automation with the following code
```
alias: OpenHaspDisplay EVCC Mode Change
description: ""
triggers:
  - alias: Capture OpenHaspEVCC Mode change
    entity_id:
      - sensor.openhaspevccmode
    trigger: state
    attribute: chargemode
conditions: []
actions:
  - choose:
      - conditions:
          - condition: state
            state: "0"
            entity_id: sensor.openhaspevccmode
            attribute: chargemode
        sequence:
          - alias: Set chargemode to off
            metadata: {}
            data:
              qos: "0"
              topic: evcc/loadpoints/1/mode/set
              retain: true
              payload: "off"
            enabled: true
            action: mqtt.publish
        alias: Changing chargemode to off
      - conditions:
          - condition: state
            state: "1"
            entity_id: sensor.openhaspevccmode
            attribute: chargemode
        sequence:
          - metadata: {}
            data:
              qos: "0"
              topic: evcc/loadpoints/1/mode/set
              retain: true
              payload: pv
            enabled: true
            alias: Set chargemode to pv
            action: mqtt.publish
        alias: Changing chargemode to pv
      - conditions:
          - condition: state
            state: "2"
            entity_id: sensor.openhaspevccmode
            attribute: chargemode
        sequence:
          - alias: Set chargemode to minpv
            metadata: {}
            data:
              qos: "0"
              topic: evcc/loadpoints/1/mode/set
              retain: true
              payload: minpv
            enabled: true
            action: mqtt.publish
        alias: Changing chargemode to min+pv
      - conditions:
          - condition: state
            state: "3"
            entity_id: sensor.openhaspevccmode
            attribute: chargemode
        sequence:
          - alias: Set chargemode to now
            metadata: {}
            data:
              qos: "0"
              topic: evcc/loadpoints/1/mode/set
              retain: true
              payload: now
            enabled: true
            action: mqtt.publish
        alias: Changing chargemode to now
mode: single
```
