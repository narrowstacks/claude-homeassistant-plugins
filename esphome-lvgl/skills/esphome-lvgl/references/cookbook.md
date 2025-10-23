# ESPHome LVGL Cookbook

Practical examples and patterns for building LVGL interfaces in ESPHome.

## Table of Contents

- Local Light Control
- Remote Light Toggle with Home Assistant
- Brightness and Volume Sliders
- Semicircle Gauge
- Thermometer Display
- Climate Control (Thermostat)
- Cover Control Panel
- Theme and Styling
- Page Navigation
- Status Icons and Boot Screen
- Material Design Icons
- Analog Clock
- Numeric Keypad
- Weather Forecast Panel
- Screen Idle and Anti-burn-in

## Local Light Switch

Simple switch widget controlling a local light:

```yaml
light:
  - platform: ...
    id: local_light
    name: 'Local light'
    on_state:
      - lvgl.widget.update:
          id: light_switch
          state:
            checked: !lambda return id(local_light).current_values.is_on();

lvgl:
  pages:
    - id: main_page
      widgets:
        - switch:
            align: CENTER
            id: light_switch
            on_click:
              light.toggle: local_light
```

## Remote Light Toggle with Home Assistant

Control a Home Assistant light entity with checkable button:

```yaml
binary_sensor:
  - platform: homeassistant
    id: remote_light
    entity_id: light.remote_light
    publish_initial_state: true
    on_state:
      then:
        lvgl.widget.update:
          id: light_btn
          state:
            checked: !lambda return x;

lvgl:
  pages:
    - id: room_page
      widgets:
        - button:
            id: light_btn
            align: CENTER
            width: 100
            height: 70
            checkable: true
            widgets:
              - label:
                  align: CENTER
                  text: 'Remote light'
            on_click:
              - homeassistant.action:
                  action: light.toggle
                  data:
                    entity_id: light.remote_light
```

## Light Brightness Slider

Control dimmable light brightness (0-255):

```yaml
sensor:
  - platform: homeassistant
    id: light_brightness
    entity_id: light.your_dimmer
    attribute: brightness
    on_value:
      - lvgl.slider.update:
          id: dimmer_slider
          value: !lambda return x;

lvgl:
  pages:
    - id: room_page
      widgets:
        - slider:
            id: dimmer_slider
            x: 20
            y: 50
            width: 30
            height: 220
            pad_all: 8
            min_value: 0
            max_value: 255
            on_release:
              - homeassistant.action:
                  action: light.turn_on
                  data:
                    entity_id: light.your_dimmer
                    brightness: !lambda return int(x);
```

## Media Player Volume Slider

Control media player volume (0.0-1.0 converted to 0-100):

```yaml
sensor:
  - platform: homeassistant
    id: media_player_volume
    entity_id: media_player.your_room
    attribute: volume_level
    on_value:
      - lvgl.slider.update:
          id: slider_media_player
          value: !lambda return (x * 100);

lvgl:
  pages:
    - id: mediaplayer_page
      widgets:
        - slider:
            id: slider_media_player
            x: 60
            y: 50
            width: 30
            height: 220
            pad_all: 8
            min_value: 0
            max_value: 100
            adv_hittest: true  # Prevents accidental touches
            on_value:
              - homeassistant.action:
                  action: media_player.volume_set
                  data:
                    entity_id: media_player.your_room
                    volume_level: !lambda return (x / 100);
```

## Semicircle Gauge

Gauge showing values from -10 to +10 with color-coded arcs:

```yaml
sensor:
  - platform: ...
    id: values_between_-10_and_10
    on_value:
      - lvgl.indicator.update:
          id: val_needle
          value: !lambda return x;
      - lvgl.label.update:
          id: val_text
          text:
            format: "%.0f"
            args: ['x']

lvgl:
  pages:
    - id: gauge_page
      widgets:
        - obj:
            height: 240
            width: 240
            align: CENTER
            bg_color: 0xFFFFFF
            border_width: 0
            pad_all: 4
            widgets:
              - meter:
                  height: 100%
                  width: 100%
                  border_width: 0
                  bg_opa: TRANSP
                  align: CENTER
                  scales:
                    - range_from: -10
                      range_to: 10
                      angle_range: 180
                      ticks:
                        count: 0
                      indicators:
                        - line:
                            id: val_needle
                            width: 8
                            r_mod: 12
                            value: -2
                        - arc:
                            color: 0xFF3000
                            r_mod: 10
                            width: 31
                            start_value: -10
                            end_value: 0
                        - arc:
                            color: 0x00FF00
                            r_mod: 10
                            width: 31
                            start_value: 0
                            end_value: 10
              
              - obj:
                  height: 146
                  width: 146
                  radius: 73
                  align: CENTER
                  border_width: 0
                  bg_color: 0xFFFFFF
                  pad_all: 0
              
              - label:
                  id: val_text
                  text_font: montserrat_48
                  align: CENTER
                  y: -5
                  text: "0"
              
              - label:
                  text_font: montserrat_18
                  align: CENTER
                  y: 8
                  x: -90
                  text: "-10"
              
              - label:
                  text_font: montserrat_18
                  align: CENTER
                  y: 8
                  x: 90
                  text: "+10"
```

## Thermometer with Meter

Precise temperature gauge with gradient ticks:

```yaml
sensor:
  - platform: ...
    id: outdoor_temperature
    on_value:
      - lvgl.indicator.update:
          id: temperature_needle
          value: !lambda return x * 10;
      - lvgl.label.update:
          id: temperature_text
          text:
            format: "%.1f°C"
            args: ['x']

lvgl:
  pages:
    - id: meter_page
      widgets:
        - obj:
            height: 240
            width: 240
            align: CENTER
            y: -18
            bg_color: 0xFFFFFF
            border_width: 0
            pad_all: 14
            widgets:
              - meter:
                  height: 100%
                  width: 100%
                  border_width: 0
                  align: CENTER
                  bg_opa: TRANSP
                  scales:
                    - range_from: -15
                      range_to: 35
                      angle_range: 180
                      ticks:
                        count: 70
                        width: 1
                        length: 31
                      indicators:
                        - tick_style:
                            start_value: -15
                            end_value: 35
                            color_start: 0x3399ff
                            color_end: 0xffcc66
                    
                    - range_from: -150
                      range_to: 350
                      angle_range: 180
                      ticks:
                        count: 0
                      indicators:
                        - line:
                            id: temperature_needle
                            width: 8
                            r_mod: 2
                            value: -150
              
              - obj:
                  height: 123
                  width: 123
                  radius: 73
                  align: CENTER
                  border_width: 0
                  pad_all: 0
                  bg_color: 0xFFFFFF
              
              - label:
                  id: temperature_text
                  text: "--.-°C"
                  align: CENTER
                  y: -26
              
              - label:
                  text: "Outdoor"
                  align: CENTER
                  y: -6
```

## Climate Control with Spinbox

Thermostat control with increment/decrement buttons:

```yaml
sensor:
  - platform: homeassistant
    id: room_thermostat
    entity_id: climate.room_thermostat
    attribute: temperature
    on_value:
      - lvgl.spinbox.update:
          id: spinbox_id
          value: !lambda return x;

lvgl:
  pages:
    - id: thermostat_control
      widgets:
        - obj:
            align: BOTTOM_MID
            y: -50
            layout:
              type: FLEX
              flex_flow: ROW
              flex_align_cross: CENTER
            width: SIZE_CONTENT
            height: SIZE_CONTENT
            widgets:
              - button:
                  id: spin_down
                  on_click:
                    - lvgl.spinbox.decrement: spinbox_id
                  widgets:
                    - label:
                        text: "-"
              
              - spinbox:
                  id: spinbox_id
                  align: CENTER
                  text_align: CENTER
                  width: 50
                  range_from: 15
                  range_to: 35
                  step: 0.5
                  rollover: false
                  digits: 3
                  decimal_places: 1
                  on_value:
                    then:
                      - homeassistant.action:
                          action: climate.set_temperature
                          data:
                            temperature: !lambda return x;
                            entity_id: climate.room_thermostat
              
              - button:
                  id: spin_up
                  on_click:
                    - lvgl.spinbox.increment: spinbox_id
                  widgets:
                    - label:
                        text: "+"
```

## Cover Control Panel

Three-button cover control with status display:

```yaml
sensor:
  - platform: homeassistant
    id: cover_myroom_pos
    entity_id: cover.myroom
    attribute: current_position
    on_value:
      - if:
          condition:
            lambda: return x == 100;
          then:
            - lvgl.widget.update:
                id: cov_up_myroom
                text_opa: 60%
          else:
            - lvgl.widget.update:
                id: cov_up_myroom
                text_opa: 100%
      - if:
          condition:
            lambda: return x == 0;
          then:
            - lvgl.widget.update:
                id: cov_down_myroom
                text_opa: 60%
          else:
            - lvgl.widget.update:
                id: cov_down_myroom
                text_opa: 100%

text_sensor:
  - platform: homeassistant
    id: cover_myroom_state
    entity_id: cover.myroom
    on_value:
      - if:
          condition:
            lambda: return ((0 == x.compare(std::string{"opening"})) or (0 == x.compare(std::string{"closing"})));
          then:
            - lvgl.label.update:
                id: cov_stop_myroom
                text: "STOP"
          else:
            - lvgl.label.update:
                id: cov_stop_myroom
                text:
                  format: "%.0f%%"
                  args: ['id(cover_myroom_pos).get_state()']

lvgl:
  pages:
    - id: room_page
      widgets:
        - label:
            x: 10
            y: 6
            width: 70
            text: "My room"
            text_align: CENTER
        
        - button:
            x: 10
            y: 30
            width: 70
            height: 68
            widgets:
              - label:
                  id: cov_up_myroom
                  align: CENTER
                  text: "\uF077"
            on_press:
              then:
                - homeassistant.action:
                    action: cover.open
                    data:
                      entity_id: cover.myroom
        
        - button:
            x: 10
            y: 103
            width: 70
            height: 68
            widgets:
              - label:
                  id: cov_stop_myroom
                  align: CENTER
                  text: STOP
            on_press:
              then:
                - homeassistant.action:
                    action: cover.stop
                    data:
                      entity_id: cover.myroom
        
        - button:
            x: 10
            y: 178
            width: 70
            height: 68
            widgets:
              - label:
                  id: cov_down_myroom
                  align: CENTER
                  text: "\uF078"
            on_press:
              then:
                - homeassistant.action:
                    action: cover.close
                    data:
                      entity_id: cover.myroom
```

## Theme with Gradients

Global theme with gradient styles:

```yaml
lvgl:
  theme:
    button:
      bg_color: 0x2F8CD8
      bg_grad_color: 0x005782
      bg_grad_dir: VER
      bg_opa: COVER
      border_color: 0x0077b3
      border_width: 1
      text_color: 0xFFFFFF
      pressed:
        bg_color: 0x006699
        bg_grad_color: 0x00334d
      checked:
        bg_color: 0x1d5f96
        bg_grad_color: 0x03324A
        text_color: 0xfff300
    
    slider:
      border_width: 1
      border_opa: 15%
      bg_color: 0xcccaca
      bg_opa: 15%
      indicator:
        bg_color: 0x1d5f96
        bg_grad_color: 0x03324A
        bg_grad_dir: VER
        bg_opa: COVER
      knob:
        bg_color: 0x2F8CD8
        bg_grad_color: 0x005782
        bg_grad_dir: VER
        bg_opa: COVER
        border_color: 0x0077b3
        border_width: 1
  
  style_definitions:
    - id: header_footer
      bg_color: 0x2F8CD8
      bg_grad_color: 0x005782
      bg_grad_dir: VER
      bg_opa: COVER
      border_opa: TRANSP
      radius: 0
      pad_all: 0
      text_color: 0xFFFFFF
      width: 100%
      height: 30
```

## Page Navigation Footer

Bottom navigation bar using buttonmatrix:

```yaml
lvgl:
  top_layer:
    widgets:
      - buttonmatrix:
          align: BOTTOM_MID
          styles: header_footer
          pad_all: 0
          outline_width: 0
          id: top_layer
          items:
            styles: header_footer
            rows:
              - buttons:
                  - id: page_prev
                    text: "\uF053"
                    on_press:
                      then:
                        lvgl.page.previous:
                  - id: page_home
                    text: "\uF015"
                    on_press:
                      then:
                        lvgl.page.show: main_page
                  - id: page_next
                    text: "\uF054"
                    on_press:
                      then:
                        lvgl.page.next:
```

## API Status Icon

Show Home Assistant connection status:

```yaml
api:
  on_client_connected:
    - if:
        condition:
          lambda: 'return (0 == client_info.find("Home Assistant "));'
        then:
          - lvgl.widget.show: lbl_hastatus
  
  on_client_disconnected:
    - if:
        condition:
          lambda: 'return (0 == client_info.find("Home Assistant "));'
        then:
          - lvgl.widget.hide: lbl_hastatus

lvgl:
  top_layer:
    widgets:
      - label:
          text: "\uF1EB"
          id: lbl_hastatus
          hidden: true
          align: TOP_RIGHT
          x: -2
          y: 7
          text_align: RIGHT
          text_color: 0xFFFFFF
```

## Page Title Bar

Title bar for each page:

```yaml
lvgl:
  pages:
    - id: main_page
      widgets:
        - obj:
            align: TOP_MID
            styles: header_footer
            widgets:
              - label:
                  text: "ESPHome LVGL Display"
                  align: CENTER
                  text_align: CENTER
                  text_color: 0xFFFFFF
    
    - id: second_page
      widgets:
        - obj:
            align: TOP_MID
            styles: header_footer
            widgets:
              - label:
                  text: "A second page"
                  align: CENTER
                  text_align: CENTER
                  text_color: 0xFFFFFF
```

## Material Design Icons in Fonts

Use MDI icons inline with text:

```yaml
font:
  - file: "fonts/Roboto-Regular.ttf"
    id: roboto_icons_42
    size: 42
    bpp: 4
    extras:
      - file: "fonts/materialdesignicons-webfont.ttf"
        glyphs: [
          "\U000F02D1",  # mdi-heart
          "\U000F05D4",  # mdi-airplane-landing
        ]

lvgl:
  pages:
    - id: main_page
      widgets:
        - label:
            text: "Just\U000f05d4here. Already\U000F02D1this."
            align: CENTER
            text_align: CENTER
            text_font: roboto_icons_42
```

## Toggle State Icon Button

Button showing different icons based on state:

```yaml
font:
  - file: "fonts/materialdesignicons-webfont.ttf"
    id: mdi_42
    size: 42
    bpp: 4
    glyphs: [
      "\U000F0335",  # mdi-lightbulb
      "\U000F0336",  # mdi-lightbulb-outline
    ]

text_sensor:
  - platform: homeassistant
    id: ts_remote_light
    entity_id: light.remote_light
    on_value:
      then:
        - lvgl.widget.update:
            id: btn_lightbulb
            state:
              checked: !lambda return (0 == x.compare(std::string{"on"}));
              disabled: !lambda return ((0 == x.compare(std::string{"unavailable"})) or (0 == x.compare(std::string{"unknown"})));
        - lvgl.label.update:
            id: lbl_lightbulb
            text: !lambda |-
              static char buf[10];
              std::string icon;
              if (0 == x.compare(std::string{"on"})) {
                icon = "\U000F0335";
              } else {
                icon = "\U000F0336";
              }
              snprintf(buf, sizeof(buf), "%s", icon.c_str());
              return buf;

lvgl:
  pages:
    - id: room_page
      widgets:
        - button:
            x: 110
            y: 40
            width: 90
            height: 50
            checkable: true
            id: btn_lightbulb
            widgets:
              - label:
                  id: lbl_lightbulb
                  align: CENTER
                  text_font: mdi_42
                  text: "\U000F0336"
            on_short_click:
              - homeassistant.action:
                  action: light.toggle
                  data:
                    entity_id: light.remote_light
```

## Analog Clock

Clock with hour and minute hands using meter scales:

```yaml
time:
  - platform: homeassistant
    id: time_comp
    on_time_sync:
      - script.execute: time_update
    on_time:
      - minutes: '*'
        seconds: 0
        then:
          - script.execute: time_update

script:
  - id: time_update
    then:
      - lvgl.indicator.update:
          id: minute_hand
          value: !lambda return id(time_comp).now().minute;
      - lvgl.indicator.update:
          id: hour_hand
          value: !lambda |-
            auto now = id(time_comp).now();
            return std::fmod(now.hour, 12) * 60 + now.minute;
      - lvgl.label.update:
          id: date_label
          text: !lambda |-
            static const char * const mon_names[] = {"JAN", "FEB", "MAR", "APR", "MAY", "JUN",
                                                      "JUL", "AUG", "SEP", "OCT", "NOV", "DEC"};
            static char date_buf[8];
            auto now = id(time_comp).now();
            snprintf(date_buf, sizeof(date_buf), "%s %2d", mon_names[now.month-1], now.day_of_month);
            return date_buf;
      - lvgl.label.update:
          id: day_label
          text: !lambda |-
            static const char * const day_names[] = {"SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"};
            return day_names[id(time_comp).now().day_of_week - 1];

lvgl:
  pages:
    - id: clock_page
      widgets:
        - obj:
            height: SIZE_CONTENT
            width: 240
            align: CENTER
            pad_all: 0
            border_width: 0
            bg_color: 0xFFFFFF
            widgets:
              - meter:
                  height: 220
                  width: 220
                  align: CENTER
                  bg_opa: TRANSP
                  border_width: 0
                  text_color: 0x000000
                  scales:
                    - range_from: 0
                      range_to: 60
                      angle_range: 360
                      rotation: 270
                      ticks:
                        width: 1
                        count: 61
                        length: 10
                        color: 0x000000
                      indicators:
                        - line:
                            id: minute_hand
                            width: 3
                            color: 0xa6a6a6
                            r_mod: -4
                            value: 0
                    
                    - range_from: 1
                      range_to: 12
                      angle_range: 330
                      rotation: 300
                      ticks:
                        width: 1
                        count: 12
                        length: 1
                        major:
                          stride: 1
                          width: 4
                          length: 10
                          color: 0xC0C0C0
                          label_gap: 12
                    
                    - range_from: 0
                      range_to: 720
                      angle_range: 360
                      rotation: 270
                      ticks:
                        count: 0
                      indicators:
                        - line:
                            id: hour_hand
                            width: 5
                            color: 0xa6a6a6
                            r_mod: -30
                            value: 0
              
              - label:
                  id: day_label
                  y: -30
              
              - label:
                  id: date_label
                  y: 30
```

## Numeric Keypad with Key Collector

Keypad for PIN entry:

```yaml
lvgl:
  pages:
    - id: keypad_page
      widgets:
        - led:
            id: lvgl_led
            x: 30
            y: 47
            color: 0xFF0000
            brightness: 70%
        
        - obj:
            width: 140
            height: 25
            align_to:
              id: lvgl_led
              align: OUT_RIGHT_MID
            x: 17
            border_width: 1
            border_opa: 50%
            pad_all: 0
            bg_opa: 80%
            bg_color: 0xFFFFFF
            widgets:
              - label:
                  id: lvgl_label
                  align: CENTER
                  text: "Enter code and \uF00C"
                  text_align: CENTER
        
        - buttonmatrix:
            id: lvgl_keypad
            x: 20
            y: 85
            width: 200
            height: 190
            items:
              pressed:
                bg_color: 0xFFFF00
              rows:
                - buttons:
                    - text: 1
                      control: { no_repeat: true }
                    - text: 2
                      control: { no_repeat: true }
                    - text: 3
                      control: { no_repeat: true }
                - buttons:
                    - text: 4
                      control: { no_repeat: true }
                    - text: 5
                      control: { no_repeat: true }
                    - text: 6
                      control: { no_repeat: true }
                - buttons:
                    - text: 7
                      control: { no_repeat: true }
                    - text: 8
                      control: { no_repeat: true }
                    - text: 9
                      control: { no_repeat: true }
                - buttons:
                    - text: "\uF55A"
                      key_code: "*"
                      control: { no_repeat: true }
                    - text: 0
                      control: { no_repeat: true }
                    - text: "\uF00C"
                      key_code: "#"
                      control: { no_repeat: true }

key_collector:
  - source_id: lvgl_keypad
    min_length: 4
    max_length: 4
    end_keys: "#"
    end_key_required: true
    back_keys: "*"
    allowed_keys: "0123456789*#"
    timeout: 5s
    on_progress:
      - if:
          condition:
            lambda: return (0 != x.compare(std::string{""}));
          then:
            - lvgl.label.update:
                id: lvgl_label
                text: !lambda 'return x.c_str();'
          else:
            - lvgl.label.update:
                id: lvgl_label
                text: "Enter code and \uF00C"
    on_result:
      - if:
          condition:
            lambda: return (0 == x.compare(std::string{"1234"}));
          then:
            - lvgl.led.update:
                id: lvgl_led
                color: 0x00FF00
          else:
            - lvgl.led.update:
                id: lvgl_led
                color: 0xFF0000
```

## Turn Off Screen When Idle

Screen saver after inactivity:

```yaml
lvgl:
  on_idle:
    timeout: !lambda "return (id(display_timeout).state * 1000);"
    then:
      - logger.log: "LVGL is idle"
      - light.turn_off: display_backlight
      - lvgl.pause:

touchscreen:
  - platform: ...
    on_release:
      - if:
          condition: lvgl.is_paused
          then:
            - logger.log: "LVGL resuming"
            - lvgl.resume:
            - lvgl.widget.redraw:
            - light.turn_on: display_backlight

light:
  - platform: ...
    id: display_backlight

number:
  - platform: template
    name: LVGL Screen timeout
    optimistic: true
    id: display_timeout
    unit_of_measurement: "s"
    initial_value: 45
    restore_value: true
    min_value: 10
    max_value: 180
    step: 5
    mode: box
```

## Prevent LCD Burn-in

Periodic pixel exercise to prevent burn-in:

```yaml
time:
  - platform: ...
    on_time:
      - hours: 2,3,4,5
        minutes: 5
        seconds: 0
        then:
          - switch.turn_on: switch_antiburn
      - hours: 2,3,4,5
        minutes: 35
        seconds: 0
        then:
          - switch.turn_off: switch_antiburn

switch:
  - platform: template
    name: Antiburn
    id: switch_antiburn
    icon: mdi:television-shimmer
    optimistic: true
    entity_category: "config"
    turn_on_action:
      - logger.log: "Starting Antiburn"
      - if:
          condition: lvgl.is_paused
          then:
            - lvgl.resume:
            - lvgl.widget.redraw:
      - lvgl.pause:
          show_snow: true
    turn_off_action:
      - logger.log: "Stopping Antiburn"
      - if:
          condition: lvgl.is_paused
          then:
            - lvgl.resume:
            - lvgl.widget.redraw:

touchscreen:
  - platform: ...
    on_release:
      then:
        - if:
            condition: lvgl.is_paused
            then:
              - lvgl.resume:
              - lvgl.widget.redraw:
```

## Boot Screen with Spinner

Splash screen that auto-hides after boot:

```yaml
esphome:
  on_boot:
    - delay: 5s
    - lvgl.widget.hide: boot_screen

image:
  - file: https://esphome.io/favicon.ico
    id: boot_logo
    resize: 200x200
    type: RGB565

lvgl:
  top_layer:
    widgets:
      - obj:
          id: boot_screen
          x: 0
          y: 0
          width: 100%
          height: 100%
          bg_color: 0xffffff
          bg_opa: COVER
          radius: 0
          pad_all: 0
          border_width: 0
          widgets:
            - image:
                align: CENTER
                src: boot_logo
                y: -40
            
            - spinner:
                align: CENTER
                y: 95
                height: 50
                width: 50
                spin_time: 1s
                arc_length: 60deg
                arc_width: 8
                indicator:
                  arc_color: 0x18bcf2
                  arc_width: 8
          on_press:
            - lvgl.widget.hide: boot_screen
```

