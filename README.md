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
```yaml
substitutions:
  name: "socket"
esphome:
  name: "${name}"
esp8266:
  board: esp8285
logger:
  baud_rate: 0
api:
  password: !secret api_password
ota:
  password: !secret api_password
wifi:
  networks:
  - ssid: !secret wifi_ssid
    password: !secret wifi_password
  - ssid: !secret wifi_ssid2
    password: !secret wifi_password2
  ap:
    ssid: "${name} Fallback Hotspot"
    password: !secret ap_password
captive_portal:
web_server:
  port: 80


switch:
  - platform: gpio
    pin: GPIO14
    name: "${name} Switch"
    id: relay
    restore_mode: RESTORE_DEFAULT_OFF

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO03
      mode: INPUT_PULLUP
      inverted: true
    name: "${name} Power Button"
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

button:
  - platform: restart
    id: restart_button
    name: "${name} Restart"

status_led:
  pin:
    number: GPIO13
    inverted: true
    
sensor:
  - platform: wifi_signal
    name: "${name} WiFi Signal"
    id: rssi
    update_interval: 120s
  - platform: uptime
    name: "${name} Uptime UTC"
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
    cf_pin: GPIO4
    cf1_pin: GPIO5
    # 1382 om\ 1998000 \\ 1446?
    # 994 om\ 1990000 \\ 2003?
    # 1725? 1600?
    voltage_divider: 2043
    model: BL0937
    current:
      name: "${name} Current"
    voltage:
      name: "${name} Voltage"
      accuracy_decimals: 0
      filters:
        - median:
    power:
      name: "${name} Power"
      id: socket_my_power
      unit_of_measurement: W
      accuracy_decimals: 0
    energy:
      name: "${name} Total Energy"
      unit_of_measurement: 'kWh'
      device_class: energy
      state_class: total_increasing
      accuracy_decimals: 2
      filters:
        - multiply: 0.001
    change_mode_every: 3
    update_interval: 5s

text_sensor:
  - platform: template
    name: "${name} Uptime"
    id: uptime_human
    icon: mdi:clock-start
    entity_category: diagnostic
```
