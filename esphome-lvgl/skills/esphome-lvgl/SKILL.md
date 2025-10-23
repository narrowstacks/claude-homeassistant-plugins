---
name: esphome-lvgl
description: Create and modify ESPHome LVGL (Light and Versatile Graphics Library) configurations for graphical displays. Use for building widget-based UIs with buttons, sliders, labels, meters, and interactive controls. Includes layouts, styling, themes, pages, and Home Assistant integration.
---

# ESPHome LVGL Graphics Library

Generate ESPHome configurations using LVGL (Light and Versatile Graphics Library) for creating sophisticated, widget-based user interfaces on graphical displays.

## Overview

LVGL is a powerful graphics library that provides pre-built widgets (buttons, sliders, labels, etc.) with built-in touch handling, layouts, and styling. It's ideal for interactive displays that need professional-looking UIs.

**Key differences from native display rendering**:
- Widget-based instead of pixel-by-pixel drawing
- Automatic layout management (Flex, Grid)
- Built-in touch/input handling per widget
- Theme and style system
- Direct integration with ESPHome components

**Requirements**:
- ESP32 or RP2040 (ESP8266 not supported)
- PSRAM recommended for large/color displays
- Display configured with `auto_clear_enabled: false`
- No `lambda` in display configuration

## Quick Start

Minimal configuration with automatic "Hello World":

```yaml
display:
  - platform: ili9xxx
    id: my_display
    # ... display config
    auto_clear_enabled: false
    update_interval: never

lvgl:
  # Empty config shows default "Hello World" page
```

Basic configuration with custom content:

```yaml
lvgl:
  widgets:
    - label:
        align: CENTER
        text: 'Hello World!'
```

## Core Concepts

### Widgets

Widgets are graphical elements (buttons, labels, sliders, etc.). Every widget has:
- **Parent**: The container it's created in (page, obj, etc.)
- **Parts**: Sub-elements that can be styled separately (main, indicator, knob, etc.)
- **States**: Visual states (default, pressed, checked, disabled, focused, etc.)
- **ID**: Optional identifier for referencing in automations

### Pages

Pages are full-screen layouts. Only one page is active at a time:

```yaml
lvgl:
  pages:
    - id: main_page
      widgets:
        - label:
            text: 'Page 1'
    
    - id: settings_page
      widgets:
        - label:
            text: 'Settings'
```

Navigate between pages:

```yaml
button:
  - platform: gpio
    on_press:
      - lvgl.page.next: 
          animation: OVER_LEFT
          time: 300ms
```

### Layouts

Layouts automatically position widgets:

**Flex**: Like CSS Flexbox, arranges widgets in rows/columns

```yaml
obj:
  layout:
    type: FLEX
    flex_flow: ROW_WRAP
    flex_align_main: SPACE_BETWEEN
  widgets:
    - button: { width: 50, height: 50 }
    - button: { width: 50, height: 50 }
    - button: { width: 50, height: 50 }
```

**Grid**: Like CSS Grid, arranges widgets in a 2D grid

```yaml
obj:
  layout:
    type: GRID
    grid_rows: [FR(1), FR(1), FR(1)]
    grid_columns: [FR(1), FR(1)]
  widgets:
    - button:
        grid_cell_row_pos: 0
        grid_cell_column_pos: 0
```

## Common Widgets

### Label

Display text, supports multi-line and formatting:

```yaml
label:
  id: my_label
  text: "Temperature: 25.5°C"
  align: CENTER
  text_color: 0xFF0000
```

Update dynamically:

```yaml
sensor:
  - platform: dht
    temperature:
      on_value:
        - lvgl.label.update:
            id: my_label
            text:
              format: "Temp: %.1f°C"
              args: ['x']
```

### Button

Interactive push button:

```yaml
button:
  id: my_button
  width: 100
  height: 50
  checkable: true  # Acts as toggle
  on_click:
    - homeassistant.action:
        action: light.toggle
        data:
          entity_id: light.living_room
  widgets:
    - label:
        text: "Toggle"
        align: CENTER
```

**States**: Use `pressed`, `checked`, `disabled` for styling different states.

### Switch

Toggle switch widget (native ESPHome component):

```yaml
switch:
  - platform: lvgl
    widget: my_switch
    name: "LVGL Switch"

lvgl:
  widgets:
    - switch:
        id: my_switch
        align: CENTER
```

### Slider

Adjustable value slider:

```yaml
slider:
  id: brightness_slider
  min_value: 0
  max_value: 255
  width: 200
  on_release:
    - homeassistant.action:
        action: light.turn_on
        data:
          entity_id: light.bedroom
          brightness: !lambda return int(x);
```

Update from sensor:

```yaml
sensor:
  - platform: homeassistant
    entity_id: light.bedroom
    attribute: brightness
    on_value:
      - lvgl.slider.update:
          id: brightness_slider
          value: !lambda return x;
```

### Arc

Circular slider/gauge:

```yaml
arc:
  id: volume_arc
  min_value: 0
  max_value: 100
  value: 50
  width: 150
  height: 150
  on_value:
    - homeassistant.action:
        action: media_player.volume_set
        data:
          entity_id: media_player.living_room
          volume_level: !lambda return x / 100.0;
```

### Meter

Analog meter with scales and indicators:

```yaml
meter:
  height: 200
  width: 200
  scales:
    - range_from: 0
      range_to: 100
      angle_range: 270
      rotation: 135
      indicators:
        - line:
            id: temp_needle
            width: 4
            color: 0xFF0000
            r_mod: -10
            value: 25
```

### Spinner

Loading animation:

```yaml
spinner:
  align: CENTER
  height: 50
  width: 50
  spin_time: 1s
  arc_length: 60deg
```

### Checkbox

Checkable box with label:

```yaml
checkbox:
  id: agree_checkbox
  text: "I agree to terms"
  on_value:
    then:
      - logger.log:
          format: "Checkbox is %s"
          args: ['x ? "checked" : "unchecked"']
```

## Styling and Themes

### Basic Styling

Style properties can be applied to any widget:

```yaml
button:
  bg_color: 0x2196F3
  bg_grad_color: 0x1976D2
  bg_grad_dir: VER
  border_width: 2
  border_color: 0x0D47A1
  radius: 10
  text_color: 0xFFFFFF
  pressed:
    bg_color: 0x1976D2
```

### Colors

Specify colors in multiple formats:

```yaml
color:
  - id: my_red
    hex: FF0000

lvgl:
  widgets:
    - label:
        text_color: 0xFF0000     # Hex
        # OR
        text_color: springgreen  # CSS name
        # OR
        text_color: id(my_red)   # Reference
```

### Opacity

Specify transparency:

```yaml
obj:
  bg_opa: COVER   # Fully opaque
  bg_opa: 0.5     # 50% transparent
  bg_opa: 50%     # 50% transparent
  bg_opa: TRANSP  # Fully transparent
```

### Theme Configuration

Apply styles globally to all widgets of a type:

```yaml
lvgl:
  theme:
    button:
      bg_color: 0x2196F3
      border_width: 2
      pressed:
        bg_color: 0x0D47A1
      checked:
        bg_color: 0x4CAF50
    
    label:
      text_color: 0x000000
      text_font: my_font
```

### Style Definitions

Reusable style sets:

```yaml
lvgl:
  style_definitions:
    - id: card_style
      bg_color: 0xFFFFFF
      radius: 10
      pad_all: 10
      shadow_width: 5
      shadow_color: 0x000000
      shadow_opa: 30%
  
  widgets:
    - obj:
        styles: card_style
        # ... widget content
```

## Layouts

### Flex Layout

Automatically arrange widgets in rows or columns:

```yaml
obj:
  width: 240
  height: 100
  layout:
    type: FLEX
    flex_flow: ROW_WRAP
    flex_align_main: SPACE_EVENLY
    flex_align_cross: CENTER
    pad_row: 5
    pad_column: 5
  widgets:
    - button: { width: 70, height: 40, flex_grow: 1 }
    - button: { width: 70, height: 40 }
    - button: { width: 70, height: 40 }
```

**Flex flow options**:
- `ROW`, `COLUMN`: Single row/column
- `ROW_WRAP`, `COLUMN_WRAP`: Multiple rows/columns with wrapping
- `ROW_REVERSE`, `COLUMN_REVERSE`: Reversed order

**Alignment options**:
- `START`, `END`, `CENTER`
- `SPACE_EVENLY`, `SPACE_AROUND`, `SPACE_BETWEEN`

### Grid Layout

Arrange widgets in a 2D grid:

```yaml
obj:
  layout:
    type: GRID
    grid_rows: [50, 50, 50]       # Fixed heights
    grid_columns: [FR(1), FR(2)]  # Proportional widths
    pad_row: 5
    pad_column: 5
  widgets:
    - label:
        text: "Cell 0,0"
        grid_cell_row_pos: 0
        grid_cell_column_pos: 0
    
    - button:
        grid_cell_row_pos: 0
        grid_cell_column_pos: 1
        grid_cell_row_span: 2     # Spans 2 rows
```

**Size units**:
- Pixels: `50`, `100px`
- Content: `CONTENT` (fits to widget size)
- Free units: `FR(1)`, `FR(2)` (proportional distribution)

## Input Devices

### Touchscreen

Automatically used if configured:

```yaml
touchscreen:
  - platform: cst816
    id: my_touch

lvgl:
  touchscreens:
    - touchscreen_id: my_touch
      long_press_time: 400ms
```

### Rotary Encoder

For navigation and value adjustment:

```yaml
sensor:
  - platform: rotary_encoder
    id: my_encoder
    pin_a: GPIO32
    pin_b: GPIO33

binary_sensor:
  - platform: gpio
    id: encoder_button
    pin: GPIO34

lvgl:
  encoders:
    - sensor: my_encoder
      enter_button: encoder_button
      group: default
```

Turning encoder: Focus next/previous widget
Pressing encoder: Click/edit widget
Long press: Exit edit mode

### Keypad

Custom button-based input:

```yaml
lvgl:
  keypads:
    - group: default
      up: button_up
      down: button_down
      left: button_left
      right: button_right
      enter: button_enter
```

## Integration with ESPHome Components

### Binary Sensor

Sync LVGL switch with binary sensor:

```yaml
binary_sensor:
  - platform: gpio
    id: motion_sensor
    on_state:
      - lvgl.widget.update:
          id: motion_indicator
          state:
            checked: !lambda return x;
```

### Sensor Values

Display sensor data:

```yaml
sensor:
  - platform: dht
    temperature:
      id: temp_sensor
      on_value:
        - lvgl.label.update:
            id: temp_label
            text:
              format: "%.1f°C"
              args: ['x']
```

### Text Sensor

Display text values:

```yaml
text_sensor:
  - platform: homeassistant
    entity_id: sensor.weather_condition
    on_value:
      - lvgl.label.update:
          id: weather_label
          text: !lambda return x.c_str();
```

### Home Assistant Actions

Control Home Assistant from LVGL:

```yaml
button:
  on_click:
    - homeassistant.action:
        action: climate.set_temperature
        data:
          entity_id: climate.living_room
          temperature: 22.5
```

## Common Patterns

### Temperature Dashboard

```yaml
lvgl:
  pages:
    - id: main_page
      widgets:
        - obj:
            align: CENTER
            width: 200
            height: 150
            bg_color: 0xFFFFFF
            radius: 10
            pad_all: 15
            widgets:
              - label:
                  text: "Living Room"
                  align: TOP_MID
                  y: 5
              
              - label:
                  id: temp_label
                  text: "--°C"
                  text_font: montserrat_48
                  align: CENTER
              
              - label:
                  id: humidity_label
                  text: "--% RH"
                  align: BOTTOM_MID
                  y: -5
```

### Control Panel with Grid

```yaml
obj:
  layout:
    type: GRID
    grid_rows: [FR(1), FR(1)]
    grid_columns: [FR(1), FR(1)]
    pad_row: 10
    pad_column: 10
  widgets:
    - button:
        grid_cell_row_pos: 0
        grid_cell_column_pos: 0
        widgets:
          - label: { text: "Light 1", align: CENTER }
    
    - button:
        grid_cell_row_pos: 0
        grid_cell_column_pos: 1
        widgets:
          - label: { text: "Light 2", align: CENTER }
```

### Gauge with Indicator

```yaml
meter:
  height: 180
  width: 180
  align: CENTER
  scales:
    - range_from: 0
      range_to: 100
      angle_range: 270
      rotation: 135
      ticks:
        count: 11
        width: 2
        length: 10
        color: 0x808080
      indicators:
        - line:
            id: gauge_needle
            width: 4
            color: 0xFF0000
            r_mod: -10
            value: 0
```

### Multi-page with Navigation

```yaml
lvgl:
  pages:
    - id: page1
      widgets:
        - label: { text: "Page 1", align: CENTER }
    
    - id: page2
      widgets:
        - label: { text: "Page 2", align: CENTER }
  
  top_layer:
    widgets:
      - obj:
          align: BOTTOM_MID
          width: 240
          height: 40
          layout:
            type: FLEX
            flex_flow: ROW
          widgets:
            - button:
                flex_grow: 1
                on_press:
                  - lvgl.page.previous:
                widgets:
                  - label: { text: "◀", align: CENTER }
            
            - button:
                flex_grow: 1
                on_press:
                  - lvgl.page.next:
                widgets:
                  - label: { text: "▶", align: CENTER }
```

## Actions and Conditions

### Widget Update Actions

Update widget properties at runtime:

```yaml
on_...:
  - lvgl.widget.update:
      id: my_label
      text: "New text"
      text_color: 0xFF0000
      hidden: false
```

### Page Navigation

```yaml
# Show specific page
- lvgl.page.show: settings_page

# Next/previous page
- lvgl.page.next:
    animation: OVER_LEFT
    time: 300ms

- lvgl.page.previous:
```

### Pause and Resume

```yaml
# Pause LVGL (stop rendering)
- lvgl.pause:
    show_snow: true  # Show random pixels (anti-burn-in)

# Resume LVGL
- lvgl.resume:
```

### Conditions

```yaml
# Check if idle
- if:
    condition:
      lvgl.is_idle:
        timeout: 30s
    then:
      - light.turn_off: backlight

# Check if paused
- if:
    condition: lvgl.is_paused
    then:
      - lvgl.resume:

# Check active page
- if:
    condition:
      lvgl.page.is_showing: main_page
    then:
      - logger.log: "Main page active"
```

## Fonts

### ESPHome Fonts (Recommended)

Use custom fonts with specific glyphs:

```yaml
font:
  - file: "fonts/Roboto-Regular.ttf"
    id: my_font
    size: 20
    bpp: 4
    glyphs: "0123456789:.°C% ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"

lvgl:
  default_font: my_font
```

### Material Design Icons

Add icons to fonts:

```yaml
font:
  - file: "fonts/Roboto-Regular.ttf"
    id: font_with_icons
    size: 24
    bpp: 4
    extras:
      - file: "fonts/materialdesignicons-webfont.ttf"
        glyphs: [
          "\U000F0335",  # mdi-lightbulb
          "\U000F0594",  # mdi-weather-sunny
          "\U000F050F",  # mdi-thermometer
        ]

label:
  text: "Temperature \U000F050F 25°C"
  text_font: font_with_icons
```

### Built-in LVGL Fonts

Use pre-rendered Montserrat fonts:

```yaml
label:
  text_font: montserrat_20  # 8-48px available
```

## Performance Tips

### Buffer Size

Adjust buffer size for performance vs memory:

```yaml
lvgl:
  buffer_size: 25%  # 12-100%, smaller uses less RAM
```

- 100%: Best performance, high RAM usage
- 25%: Good balance
- 12%: Minimal RAM, slower updates

### Update Interval

```yaml
display:
  update_interval: never  # LVGL handles updates

# Manual updates when needed
sensor:
  on_value:
    - component.update: my_display
```

### Conditional Updates

Only update changed values:

```yaml
sensor:
  - platform: dht
    temperature:
      on_value:
        then:
          - if:
              condition:
                lambda: 'return abs(x - id(last_temp)) > 0.1;'
              then:
                - lvgl.label.update:
                    id: temp_label
                    text:
                      format: "%.1f°C"
                      args: ['x']
```

## Troubleshooting

### Display not updating

Check that:
- `auto_clear_enabled: false` on display
- `update_interval` not set to incompatible value
- LVGL configuration is valid

### Widgets not visible

- Check widget `hidden` flag
- Verify `opa` (opacity) settings
- Ensure widget is within parent bounds
- Check `bg_opa` is not `TRANSP`

### Touch not working

- Verify touchscreen is configured in `lvgl.touchscreens`
- Check touchscreen driver configuration
- Test touchscreen separately

### Out of memory

- Reduce `buffer_size`
- Use smaller fonts
- Simplify layouts
- Enable PSRAM if available

## Advanced Features

See [references/cookbook.md](references/cookbook.md) for:
- Complete examples with Home Assistant integration
- Weather panels
- Climate controls
- Custom themes
- Animations
- Anti-burn-in protection
- Advanced layouts

See [references/widgets.md](references/widgets.md) for:
- Complete widget reference
- All configuration options
- Widget-specific features
- Event handling details

## Quick Reference

**Common Widgets**: label, button, switch, slider, arc, meter, checkbox, spinner, obj  
**Layouts**: FLEX (rows/columns), GRID (2D table), NONE  
**Alignment**: CENTER, TOP_LEFT, BOTTOM_RIGHT, etc.  
**Colors**: 0xRRGGBB, CSS names, color IDs  
**Opacity**: TRANSP, 0.0-1.0, 0-100%, COVER  
**Actions**: lvgl.widget.update, lvgl.page.show/next/previous, lvgl.pause/resume  
**Conditions**: lvgl.is_idle, lvgl.is_paused, lvgl.page.is_showing  
**Fonts**: montserrat_8 to montserrat_48, custom ESPHome fonts

