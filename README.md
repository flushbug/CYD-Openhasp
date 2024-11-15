# CYD-Openhasp

Small project to collect https://www.openhasp.com/ based Screens on cheap EPS32 Display "CYD" and Home Assistant.
Communication is done via MQTT (has to be installed/configured on HA).

Example : Display of Home energy values + Control of PV based EV Charge Controller http://evcc.io/

![IMG_6280](https://github.com/user-attachments/assets/fb8e2cf4-05e3-47e7-a053-e1133fd6aaa4)

Needed configuration Steps :

1. get a "CYD" like ESP32-2432S028. See https://www.openhasp.com/0.7.0/hardware/sunton/esp32-8048s0xx/ for more screen sizes.
2. Install openhasp (Serial flasher over USB out of the browser. Details on https://www.openhasp.com/0.7.0/firmware/esp32/
3. Connect via Http/Browser to openhasp "plate" device and configure/calibrate the display
4. Configure MQTT
5. replace/modify pages.json (example 
