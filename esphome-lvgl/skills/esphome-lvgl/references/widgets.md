# LVGL Widgets Reference

Complete reference for all LVGL widgets available in ESPHome.

## Widget Common Properties

All widgets share these common properties:

### Positioning

```yaml
x: 10              # X coordinate (pixels)
y: 20              # Y coordinate (pixels)
align: CENTER      # Alignment point
width: 100         # Width (pixels, %, or SIZE_CONTENT)
height: 50         # Height (pixels, %, or SIZE_CONTENT)
```

**Alignment options**: `CENTER`, `TOP_LEFT`, `TOP_MID`, `TOP_RIGHT`, `BOTTOM_LEFT`, `BOTTOM_MID`, `BOTTOM_RIGHT`, `LEFT_MID`, `RIGHT_MID`, `OUT_TOP_LEFT`, `OUT_TOP_MID`, etc.

### Visibility and State

```yaml
hidden: false      # Hide/show widget
state:
  checked: true    # Checked state (for checkable widgets)
  disabled: false  # Disabled state
  focused: false   # Focused state
```

### Styling

All style properties from SKILL.md apply to widgets. Common ones:

```yaml
bg_color: 0xFF0000
bg_opa: COVER
border_width: 2
border_color: 0x000000
radius: 10
pad_all: 5
text_color: 0xFFFFFF
```

### Groups and Focus

```yaml
group: default     # Input device group
scroll_on_focus: true
```

### Event Triggers

```yaml
on_click:          # Single click
on_short_click:    # Short click (released quickly)
on_long_press:     # Held down
on_press:          # Pressed down
on_release:        # Released
on_value:          # Value changed
```

## Label

Display text, single or multi-line.

```yaml
label:
  id: my_label
  text: "Hello World"
  text_font: montserrat_20
  text_color: 0x000000
  text_align: CENTER
  long_mode: WRAP    # Text overflow behavior
  recolor: false     # Enable color codes in text
```

**Long mode options**: `WRAP`, `DOT`, `SCROLL`, `SCROLL_CIRCULAR`, `CLIP`

**Update action**:
```yaml
- lvgl.label.update:
    id: my_label
    text: "New text"
    # OR with formatting:
    text:
      format: "Temp: %.1f°C"
      args: ['x']
```

## Button

Clickable button widget.

```yaml
button:
  id: my_button
  width: 100
  height: 50
  checkable: true        # Acts as toggle
  on_click:
    - logger.log: "Clicked!"
  widgets:               # Child widgets (usually label)
    - label:
        text: "Press me"
        align: CENTER
```

**States**: `default`, `pressed`, `checked`, `disabled`, `focused`

**Update action**:
```yaml
- lvgl.widget.update:
    id: my_button
    state:
      checked: true
```

## Switch

Toggle switch (native ESPHome component).

```yaml
# ESPHome component
switch:
  - platform: lvgl
    widget: my_switch
    name: "LVGL Switch"

# LVGL widget
lvgl:
  widgets:
    - switch:
        id: my_switch
        on_click:
          - logger.log: "Switch toggled"
```

**Parts**: `main` (track), `knob` (moving part), `indicator` (checked state background)

## Slider

Horizontal or vertical value slider.

```yaml
slider:
  id: my_slider
  min_value: 0
  max_value: 100
  value: 50
  width: 200
  height: 10         # Small height = horizontal
  adv_hittest: true  # Only clickable on knob
  on_value:
    - logger.log:
        format: "Value: %.0f"
        args: ['x']
  on_release:
    - logger.log: "Released"
```

**Parts**: `main` (background), `indicator` (filled portion), `knob` (draggable handle)

**Update action**:
```yaml
- lvgl.slider.update:
    id: my_slider
    value: 75
    animated: true
    anim_time: 300ms
```

## Arc

Circular slider/gauge.

```yaml
arc:
  id: my_arc
  min_value: 0
  max_value: 100
  value: 50
  start_angle: 135   # Starting position (degrees * 10)
  end_angle: 45      # Ending position (degrees * 10)
  rotation: 0        # Rotation offset
  width: 150
  height: 150
  adjustable: true   # User can adjust value
  on_value:
    - logger.log:
        format: "Arc: %.0f"
        args: ['x']
```

**Parts**: `main` (background arc), `indicator` (filled arc), `knob` (adjustment point)

**Update action**:
```yaml
- lvgl.arc.update:
    id: my_arc
    value: 75
```

## Checkbox

Checkbox with text label.

```yaml
checkbox:
  id: my_checkbox
  text: "I agree"
  text_font: montserrat_14
  on_value:
    - logger.log:
        format: "Checked: %s"
        args: ['x ? "yes" : "no"']
```

**Parts**: `main` (background), `indicator` (checkmark)

## Dropdown

Dropdown selection menu.

```yaml
dropdown:
  id: my_dropdown
  options: "Option 1\nOption 2\nOption 3"
  selected: 0
  symbol: "\uF078"   # Down arrow symbol
  on_value:
    - logger.log:
        format: "Selected: %d"
        args: ['x']
```

**Update action**:
```yaml
- lvgl.dropdown.update:
    id: my_dropdown
    selected: 1
    options: "New1\nNew2\nNew3"
```

## Roller

Scrollable wheel selector.

```yaml
roller:
  id: my_roller
  options: "Option 1\nOption 2\nOption 3"
  selected: 1
  mode: NORMAL       # or INFINITE
  visible_row_count: 3
  on_value:
    - logger.log:
        format: "Selected: %d"
        args: ['x']
```

**Update action**:
```yaml
- lvgl.roller.update:
    id: my_roller
    selected: 2
    animated: true
    anim_time: 300ms
```

## Spinbox

Numeric value with increment/decrement.

```yaml
spinbox:
  id: my_spinbox
  range_from: 0
  range_to: 100
  step: 1
  rollover: false
  digits: 3
  decimal_places: 0
  on_value:
    - logger.log:
        format: "Value: %.1f"
        args: ['x']
```

**Actions**:
```yaml
- lvgl.spinbox.increment: my_spinbox
- lvgl.spinbox.decrement: my_spinbox
- lvgl.spinbox.update:
    id: my_spinbox
    value: 50
```

## Meter

Analog meter/gauge with scales and indicators.

```yaml
meter:
  id: my_meter
  height: 200
  width: 200
  scales:
    - range_from: 0
      range_to: 100
      angle_range: 270     # Arc angle
      rotation: 135        # Starting angle
      ticks:
        count: 11          # Number of ticks
        width: 2           # Tick width
        length: 10         # Tick length
        color: 0x808080
        major:             # Major tick marks
          stride: 5        # Every 5th tick
          width: 4
          length: 15
          label_gap: 10    # Distance from tick to label
      indicators:
        - line:            # Needle indicator
            id: needle
            width: 4
            color: 0xFF0000
            r_mod: -10     # Radius modifier (negative = shorter)
            value: 50
        - arc:             # Arc indicator
            start_value: 0
            end_value: 75
            color: 0x00FF00
            width: 10
            r_mod: -20
        - tick_style:      # Colored tick range
            start_value: 80
            end_value: 100
            color_start: 0xFFFF00
            color_end: 0xFF0000
```

**Update action**:
```yaml
- lvgl.indicator.update:
    id: needle
    value: 75
```

## Bar

Horizontal or vertical progress bar.

```yaml
bar:
  id: my_bar
  min_value: 0
  max_value: 100
  value: 50
  width: 200
  height: 20
  mode: NORMAL       # or SYMMETRICAL, RANGE
```

**Parts**: `main` (background), `indicator` (filled portion)

## Spinner

Animated loading spinner.

```yaml
spinner:
  id: my_spinner
  align: CENTER
  height: 50
  width: 50
  spin_time: 1s
  arc_length: 60deg
  arc_width: 8
```

**Parts**: `main` (background), `indicator` (spinning arc)

## LED

Simple LED indicator.

```yaml
led:
  id: my_led
  color: 0xFF0000
  brightness: 100%
```

**Update action**:
```yaml
- lvgl.led.update:
    id: my_led
    color: 0x00FF00
    brightness: 50%
```

## Image

Display an image.

```yaml
image:
  id: my_image
  src: my_image_id   # Reference to ESPHome image
  align: CENTER
```

Images must be defined in ESPHome:
```yaml
image:
  - file: "image.png"
    id: my_image_id
    type: RGB565
```

## Animimg

Animated image sequence.

```yaml
animimg:
  id: my_anim
  src: [frame1, frame2, frame3]
  duration: 1000ms
  repeat_count: -1   # -1 = infinite
```

**Actions**:
```yaml
- lvgl.animimg.start: my_anim
- lvgl.animimg.stop: my_anim
```

## Buttonmatrix

Grid of buttons.

```yaml
buttonmatrix:
  id: my_matrix
  width: 240
  height: 100
  items:
    bg_color: 0x2196F3
    pressed:
      bg_color: 0x1976D2
    rows:
      - buttons:
          - id: btn1
            text: "1"
            width: 1       # Relative width
          - id: btn2
            text: "2"
            width: 1
          - id: btn3
            text: "3"
            width: 1
      - buttons:
          - id: btn4
            text: "4"
          - id: btn5
            text: "5"
            control:       # Button controls
              hidden: false
              no_repeat: false
              disabled: false
              checkable: false
              checked: false
```

**Button controls**:
- `hidden`: Hide button
- `no_repeat`: Disable auto-repeat on long press
- `disabled`: Disable button
- `checkable`: Button can be toggled
- `checked`: Initial checked state

## Obj

Container object for grouping widgets.

```yaml
obj:
  id: container
  width: 200
  height: 100
  align: CENTER
  layout:            # Optional layout
    type: FLEX
    flex_flow: ROW
  widgets:
    - label: { text: "Child 1" }
    - label: { text: "Child 2" }
```

The most flexible widget - use for grouping, cards, panels, etc.

## Textarea

Multi-line text input.

```yaml
textarea:
  id: my_textarea
  text: "Initial text"
  placeholder: "Enter text..."
  one_line: false
  password_mode: false
  max_length: 100
  on_value:
    - logger.log:
        format: "Text: %s"
        args: ['x.c_str()']
```

Requires keyboard widget for input.

## Keyboard

On-screen keyboard for textarea.

```yaml
keyboard:
  id: my_keyboard
  textarea: my_textarea
  mode: TEXT_LOWER   # TEXT_LOWER, TEXT_UPPER, NUMBER, SPECIAL
```

## Tabview

Tabbed interface.

```yaml
tabview:
  id: my_tabview
  tabs:
    - name: "Tab 1"
      widgets:
        - label: { text: "Content 1" }
    - name: "Tab 2"
      widgets:
        - label: { text: "Content 2" }
```

## Line

Draw a line between points.

```yaml
line:
  id: my_line
  points:
    - x: 10
      y: 20
    - x: 100
      y: 80
    - x: 50
      y: 120
  line_color: 0xFF0000
  line_width: 3
  line_rounded: true
```

## Chart

Data visualization chart.

```yaml
chart:
  id: my_chart
  width: 200
  height: 100
  series:
    - id: series1
      type: LINE
      color: 0xFF0000
      points: [10, 20, 15, 30, 25]
```

## Widget Flags

Control widget behavior:

```yaml
widget:
  flags:
    hidden: false
    clickable: true
    checkable: false
    scrollable: false
    scroll_elastic: true
    scroll_momentum: true
    scroll_on_focus: false
    click_focusable: true
    snappable: false
    floating: false       # Ignore layouts
    ignore_layout: false  # Same as floating
```

## State-based Styling

Apply different styles based on widget state:

```yaml
button:
  # Default state
  bg_color: 0x2196F3
  
  # Pressed state
  pressed:
    bg_color: 0x1976D2
  
  # Checked state (checkable buttons)
  checked:
    bg_color: 0x4CAF50
    text_color: 0xFFFFFF
  
  # Disabled state
  disabled:
    bg_opa: 50%
    text_opa: 50%
  
  # Focused state (keyboard/encoder navigation)
  focused:
    border_color: 0xFFEB3B
    border_width: 2
```

## Parts

Some widgets have multiple parts that can be styled separately:

### Slider Parts

```yaml
slider:
  # Main part (background)
  bg_color: 0xE0E0E0
  
  # Indicator part (filled section)
  indicator:
    bg_color: 0x2196F3
  
  # Knob part (draggable handle)
  knob:
    bg_color: 0xFFFFFF
    border_width: 2
```

### Switch Parts

```yaml
switch:
  # Main part (track)
  bg_color: 0x9E9E9E
  
  # Indicator part (checked state track)
  indicator:
    checked:
      bg_color: 0x4CAF50
  
  # Knob part (moving circle)
  knob:
    bg_color: 0xFFFFFF
```

### Checkbox Parts

```yaml
checkbox:
  # Main part (box background)
  bg_color: 0xFFFFFF
  border_width: 2
  
  # Indicator part (checkmark)
  indicator:
    checked:
      text_color: 0xFFFFFF
      bg_color: 0x2196F3
```

## Native ESPHome Integration

Some LVGL widgets act as native ESPHome components:

### Switch

```yaml
switch:
  - platform: lvgl
    widget: lvgl_switch_id
    name: "My Switch"
```

### Number

```yaml
number:
  - platform: lvgl
    widget: lvgl_slider_id
    name: "My Slider"
    min_value: 0
    max_value: 100
    step: 1
```

### Select

```yaml
select:
  - platform: lvgl
    widget: lvgl_dropdown_id
    name: "My Dropdown"
```

### Text

```yaml
text:
  - platform: lvgl
    widget: lvgl_label_id
    name: "My Label"
    mode: text
```

### Light

```yaml
light:
  - platform: lvgl
    widget: lvgl_led_id
    name: "My LED"
```

## Animation

Animate widget property changes:

```yaml
- lvgl.widget.update:
    id: my_widget
    x: 100
    animated: true
    anim_time: 500ms
```

## Widget Refresh

Re-evaluate lambda properties:

```yaml
widgets:
  - label:
      id: dynamic_label
      text: !lambda return id(sensor).state.c_str();

# Later, refresh the lambda
- lvgl.widget.refresh: dynamic_label
```

## Multiple Widgets Update

Update multiple widgets at once:

```yaml
- lvgl.widget.update:
    id: [label1, label2, label3]
    text_color: 0xFF0000
```

## Widget Hierarchy

Widgets can contain other widgets:

```yaml
obj:
  widgets:
    - button:
        widgets:
          - label:
              text: "Click"
          - led:
              color: 0xFF0000
```

Parent widget properties affect children (inheritance).

## Best Practices

1. **Use IDs**: Assign IDs to widgets you need to update
2. **Minimize updates**: Only update changed values
3. **Use layouts**: Let Grid/Flex position widgets automatically
4. **State styling**: Style different states for better UX
5. **Native components**: Use LVGL native components for HA integration
6. **Parts**: Style widget parts separately for fine control
7. **Groups**: Organize widgets in groups for input devices
8. **Buffer size**: Adjust for your display and memory constraints

