# A9TY-V1.3DL
Replacement the module WR2 with esp8285

A sort of https://github.com/ianchi/athom-configs/blob/patch-1/athom-smart-plug.yaml
and of https://zry.io/archives/783

Замена tuya модуля wr2 в розетке Perenio на esp-02s(esp8285) 

Модуль можно заказать на али https://a.aliexpress.com/_Dnqn3gn

Было:
![photo_5398096145688936565_x](https://user-images.githubusercontent.com/64173457/180664538-5d51b894-e19e-467b-a7cc-2b846606abc6.jpg)

Стало:
![photo_5399890071924096236_y](https://user-images.githubusercontent.com/64173457/180664547-dda255e4-87c4-4a99-aaee-682d2f316b51.jpg)


Код прошивки:
```
substitutions:
  board_name: "socket"

esphome:
  name: $board_name
  platform: ESP8266
  board: esp01_1m

# disable logging
logger:
  baud_rate: 0

api:
  password: !secret passwordapi

ota:
  password: !secret passwordota


wifi:
  networks:
  - ssid: !secret wifi1
    password: !secret password1
web_server:
  port: 80

button:
  - platform: restart
    id: restart_button
    name: "${board_name} Restart"

status_led:
  pin:
    number: GPIO13
    inverted: true

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO03
      mode: INPUT_PULLUP
      inverted: true
    name: "${board_name} Power Button"
    internal: true
    on_multi_click:
      - timing:
          - ON for at most 1s
          - OFF for at least 0.2s
        then:
          - switch.toggle: relay
      - timing:
          - ON for at least 5s
        then:
          - button.press: restart_button

switch:
  - platform: gpio
    name: relay_$board_name
    pin: GPIO14
    id: relay
    restore_mode: RESTORE_DEFAULT_OFF

sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
    id: rssi
    update_interval: 120s
  - platform: uptime
    name: "Uptime Raw"
    id: device_uptime
    update_interval: 120s
    on_raw_value:
      then:
        - text_sensor.template.publish:
            id: uptime_human
            state: !lambda |-
              int seconds = round(id(device_uptime).raw_state);
              int days = seconds / (24 * 3600);
              seconds = seconds % (24 * 3600);
              int hours = seconds / 3600;
              seconds = seconds % 3600;
              int minutes = seconds /  60;
              seconds = seconds % 60;
              return (
                (days ? to_string(days) + "d " : "") +
                (hours ? to_string(hours) + "h " : "") +
                (minutes ? to_string(minutes) + "m " : "") +
                (to_string(seconds) + "s")
              ).c_str();
  - platform: hlw8012
    sel_pin: 
      number: GPIO12
      inverted: True
    cf_pin: GPIO04
    cf1_pin: GPIO05
    current:
      name: Current_$board_name
    voltage:
      name: Voltage_$board_name
      accuracy_decimals: 0
      filters:
        - median:
    power:
      name: Power_$board_name
      # unit_of_measurement: W
      accuracy_decimals: 0
    energy:
      name: Energy_$board_name
      filters:
        - multiply: 0.001
      unit_of_measurement: 'kWh'
      device_class: energy
      state_class: total_increasing
      accuracy_decimals: 2
    update_interval: 20s
    voltage_divider: 2043
    change_mode_every: 3
    model: BL0937


text_sensor:
  - platform: template
    name: "Uptime"
    id: uptime_human
    icon: mdi:clock-start
    entity_category: diagnostic
```
