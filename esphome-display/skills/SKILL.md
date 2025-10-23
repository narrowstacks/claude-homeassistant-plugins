---
name: esphome-display
description: Create and modify ESPHome display lambda code for graphical displays (OLED, E-Paper, TFT). Use when writing display rendering code, drawing shapes, displaying sensor values, formatting text, working with pages, or troubleshooting display issues.
---

# ESPHome Display Code Generator

Generate ESPHome display lambda code for graphical displays including OLED, E-Paper, and TFT displays.

## Core Concepts

### Display Lambda Structure

All display code goes in a `lambda` block that executes on each display update:

```yaml
display:
  - platform: ssd1306_i2c  # or other display type
    id: my_display
    lambda: |-
      // Your rendering code here using 'it' object
      it.print(0, 0, id(my_font), "Hello");
```

The `it` variable represents the display buffer and provides all drawing methods.

### Coordinate System

- Origin (0,0) is at **top-left corner**
- X increases rightward, Y increases downward
- Use `it.get_width()` and `it.get_height()` for display dimensions
- Rotation can be set via `rotation: 90` (0, 90, 180, or 270 degrees)

## Drawing Functions

### Basic Shapes

```yaml
lambda: |-
  // Lines
  it.line(x1, y1, x2, y2);
  it.line(x1, y1, x2, y2, COLOR_ON);  // with color
  it.horizontal_line(x, y, width);
  it.vertical_line(x, y, height);
  
  // Rectangles
  it.rectangle(x, y, width, height);
  it.filled_rectangle(x, y, width, height);
  
  // Circles
  it.circle(center_x, center_y, radius);
  it.filled_circle(center_x, center_y, radius);
  
  // Rings and gauges
  it.filled_ring(center_x, center_y, outer_radius, inner_radius);
  it.filled_gauge(center_x, center_y, outer_radius, inner_radius, percent);
  
  // Triangles
  it.triangle(x1, y1, x2, y2, x3, y3);
  it.filled_triangle(x1, y1, x2, y2, x3, y3);
  
  // Polygons
  it.regular_polygon(x, y, radius, 6);  // hexagon
  it.filled_regular_polygon(x, y, radius, 8);  // octagon
  
  // Single pixel
  it.draw_pixel_at(x, y);
  it.fill(COLOR_ON);  // fill entire display
```

### Colors

**Monochrome displays**: Use `COLOR_ON` (default, pixel on) and `COLOR_OFF` (pixel off).

**Color displays**: Create Color objects:

```yaml
lambda: |-
  auto red = Color(255, 0, 0);
  auto green = Color(0, 255, 0);
  auto custom = Color(123, 45, 67);
  
  it.filled_circle(50, 50, 20, red);
```

Or define colors globally:

```yaml
color:
  - id: my_red
    red: 100%
    green: 20%
    blue: 0%
  - id: my_blue
    hex: 0000FF

lambda: |-
  it.print(0, 0, id(font), id(my_red), "Red text");
```

## Text Rendering

### Static Text

```yaml
font:
  - file: "fonts/arial.ttf"
    id: my_font
    size: 12
  - file: "fonts/arial.ttf"
    id: large_font
    size: 24

lambda: |-
  // Basic print
  it.print(0, 0, id(my_font), "Hello World");
  
  // With color
  it.print(0, 10, id(my_font), COLOR_ON, "Text");
  
  // With alignment
  it.print(64, 0, id(my_font), TextAlign::TOP_CENTER, "Centered");
  it.print(128, 0, id(my_font), TextAlign::TOP_RIGHT, "Right");
  
  // Multiple alignment options:
  // TOP_LEFT, TOP_CENTER, TOP_RIGHT
  // CENTER_LEFT, CENTER, CENTER_RIGHT
  // BOTTOM_LEFT, BOTTOM_CENTER, BOTTOM_RIGHT, BASELINE_LEFT, BASELINE_CENTER, BASELINE_RIGHT
```

### Formatted Text (printf)

Display sensor values and dynamic content:

```yaml
sensor:
  - platform: dht
    temperature:
      id: temp_sensor
    humidity:
      id: humidity_sensor

lambda: |-
  // Basic sensor display
  it.printf(0, 0, id(my_font), "Temp: %.1f°C", id(temp_sensor).state);
  
  // Multiple values
  it.printf(0, 10, id(my_font), "T: %.1f°C H: %.0f%%", 
            id(temp_sensor).state, id(humidity_sensor).state);
  
  // With color and alignment
  it.printf(64, 20, id(my_font), COLOR_ON, TextAlign::CENTER, 
            "Value: %.2f", id(temp_sensor).state);
```

**Common printf specifiers**:
- `%.1f` - Float with 1 decimal place
- `%.0f` - Float with no decimals (rounds)
- `%d` - Integer
- `%s` - String (use `.c_str()` for text sensors)
- `%%` - Literal % sign

### Text Sensors

```yaml
text_sensor:
  - platform: template
    id: status_text

lambda: |-
  it.printf(0, 0, id(my_font), "Status: %s", id(status_text).state.c_str());
```

### Binary Sensors

```yaml
binary_sensor:
  - platform: gpio
    id: door_sensor

lambda: |-
  // Method 1: Conditional
  if (id(door_sensor).state) {
    it.print(0, 0, id(my_font), "Door: OPEN");
  } else {
    it.print(0, 0, id(my_font), "Door: CLOSED");
  }
  
  // Method 2: Inline ternary
  it.printf(0, 0, id(my_font), "Door: %s", 
            id(door_sensor).state ? "OPEN" : "CLOSED");
```

## Display Pages

Switch between multiple screen layouts:

```yaml
display:
  - platform: ssd1306_i2c
    id: my_display
    pages:
      - id: page1
        lambda: |-
          it.print(0, 0, id(my_font), "Page 1: Temperature");
          it.printf(0, 20, id(my_font), "%.1f°C", id(temp_sensor).state);
      
      - id: page2
        lambda: |-
          it.print(0, 0, id(my_font), "Page 2: Humidity");
          it.printf(0, 20, id(my_font), "%.0f%%", id(humidity_sensor).state);
      
      - id: page3
        lambda: |-
          it.print(0, 0, id(my_font), "Page 3: Status");
          it.printf(0, 20, id(my_font), "%s", id(status_text).state.c_str());

# Auto-cycle through pages
interval:
  - interval: 5s
    then:
      - display.page.show_next: my_display
      - component.update: my_display

# Manual page control
button:
  - platform: gpio
    id: next_button
    on_press:
      - display.page.show_next: my_display
      - component.update: my_display
```

## Common Patterns

### Sensor Dashboard

```yaml
lambda: |-
  // Title
  it.print(0, 0, id(large_font), "Home Status");
  
  // Temperature with icon
  it.print(0, 20, id(my_font), "Temperature:");
  it.printf(100, 20, id(my_font), "%.1f°C", id(temp_sensor).state);
  
  // Humidity
  it.print(0, 35, id(my_font), "Humidity:");
  it.printf(100, 35, id(my_font), "%.0f%%", id(humidity_sensor).state);
  
  // Status indicator
  if (id(temp_sensor).state > 25) {
    it.filled_circle(120, 50, 5, COLOR_ON);  // Warning dot
  }
```

### Progress Bar

```yaml
lambda: |-
  float progress = id(sensor).state;  // 0-100
  int bar_width = 100;
  int bar_height = 10;
  int x = 10, y = 30;
  
  // Outline
  it.rectangle(x, y, bar_width, bar_height);
  
  // Filled portion
  int filled = (bar_width - 2) * progress / 100;
  it.filled_rectangle(x + 1, y + 1, filled, bar_height - 2);
  
  // Percentage text
  it.printf(x + bar_width/2, y - 5, id(my_font), 
            TextAlign::BOTTOM_CENTER, "%.0f%%", progress);
```

### Gauge Display

```yaml
lambda: |-
  int center_x = 64, center_y = 32;
  int outer_radius = 25, inner_radius = 20;
  float value = id(sensor).state;  // 0-100
  
  // Background ring
  it.filled_ring(center_x, center_y, outer_radius, inner_radius, COLOR_OFF);
  
  // Filled gauge (value percentage)
  it.filled_gauge(center_x, center_y, outer_radius, inner_radius, value);
  
  // Center text
  it.printf(center_x, center_y, id(my_font), TextAlign::CENTER, "%.0f", value);
```

### Multi-line Text

```yaml
lambda: |-
  int line_height = 12;
  int y = 0;
  
  it.print(0, y, id(my_font), "Line 1");
  y += line_height;
  it.print(0, y, id(my_font), "Line 2");
  y += line_height;
  it.print(0, y, id(my_font), "Line 3");
```

### Time Display

```yaml
time:
  - platform: sntp
    id: sntp_time

lambda: |-
  it.strftime(64, 0, id(large_font), TextAlign::TOP_CENTER,
              "%H:%M", id(sntp_time).now());
  it.strftime(64, 30, id(my_font), TextAlign::TOP_CENTER,
              "%A, %B %d", id(sntp_time).now());
```

**Common strftime formats**:
- `%H:%M` - 24-hour time (14:30)
- `%I:%M %p` - 12-hour time (2:30 PM)
- `%A` - Full weekday (Monday)
- `%a` - Short weekday (Mon)
- `%B %d` - Month day (January 15)
- `%Y-%m-%d` - ISO date (2025-01-15)

## Screen Clipping

Restrict drawing to specific regions:

```yaml
lambda: |-
  // Draw normally across full screen
  it.print(0, 0, id(my_font), "Full screen text");
  
  // Start clipping to region
  it.start_clipping(20, 10, 100, 50);  // left, top, right, bottom
  
  // This text will be clipped to the region
  it.print(0, 20, id(my_font), "Clipped text...");
  
  // End clipping
  it.end_clipping();
  
  // Back to full screen
  it.print(0, 55, id(my_font), "More text");
```

Advanced clipping:

```yaml
lambda: |-
  it.start_clipping(10, 10, 110, 50);
  
  // Expand clipping region
  it.extend_clipping(5, 5, 115, 55);
  
  // Shrink clipping region
  it.shrink_clipping(2, 2, 2, 2);
  
  // Check if clipping is active
  if (it.is_clipping()) {
    auto rect = it.get_clipping();
    // rect.x, rect.y, rect.w, rect.h
  }
  
  it.end_clipping();
```

## Images

```yaml
image:
  - file: "logo.png"
    id: my_logo
    resize: 32x32

lambda: |-
  // Draw at position
  it.image(0, 0, id(my_logo));
  
  // Draw with alignment
  it.image(64, 32, id(my_logo), ImageAlign::CENTER);
  
  // For monochrome, specify pixel colors
  it.image(0, 0, id(my_logo), COLOR_ON, COLOR_OFF);
```

## Configuration Tips

### Display Setup

```yaml
display:
  - platform: ssd1306_i2c
    id: my_display
    model: "SSD1306 128x64"
    address: 0x3C
    rotation: 0  # 0, 90, 180, 270
    update_interval: 1s
    auto_clear_enabled: true  # Clear before each lambda
    lambda: |-
      // Rendering code
```

### Font Optimization

Only include glyphs you need to save memory:

```yaml
font:
  - file: "arial.ttf"
    id: my_font
    size: 14
    glyphs: "0123456789:.°C% "  # Only these characters
```

### Anti-aliased Fonts

For color displays with anti-aliasing:

```yaml
lambda: |-
  // Specify foreground and background colors
  it.print(0, 0, id(aa_font), COLOR_ON, TextAlign::TOP_LEFT, 
           "Anti-aliased", COLOR_OFF);
```

## Troubleshooting

### Display Test Card

Verify display is working correctly:

```yaml
display:
  - platform: ssd1306_i2c
    show_test_card: true  # Shows color bars and position markers
```

### Common Issues

**Nothing displays**:
- Check `update_interval` is not set to `never`
- Verify display address and wiring
- Ensure fonts are properly defined
- Check sensor IDs are correct

**Text cut off**:
- Use `it.get_width()` and `it.get_height()` to stay within bounds
- Check text alignment settings
- Verify font size fits the display

**Slow updates**:
- Reduce `update_interval` or use `component.update` action
- Simplify lambda code
- Use pages instead of complex conditionals

**Colors wrong** (color displays):
- Use `show_test_card: true` to verify color output
- Check if display is in correct color mode
- Verify Color() RGB values are in 0-255 range

## Advanced Examples

See [examples.md](references/examples.md) for:
- Complex multi-page layouts
- Animated graphics
- Graph rendering
- QR code display
- Custom widgets
- Performance optimization

## Code Generation Guidelines

When generating display code:

1. **Always include required components** (fonts, sensors, etc.)
2. **Use appropriate data types** (`.state` for sensors, `.c_str()` for text sensors)
3. **Consider display size** (check dimensions fit the content)
4. **Add helpful comments** for complex layouts
5. **Use constants** for repeated values (positions, colors)
6. **Test with realistic data** (consider min/max sensor values)
7. **Handle edge cases** (missing sensors, null states, long text)

### Template Structure

```yaml
# Required components
font:
  - file: "arial.ttf"
    id: my_font
    size: 12

sensor:
  - platform: ...
    id: my_sensor

# Display configuration
display:
  - platform: ssd1306_i2c
    id: my_display
    lambda: |-
      // Clear handled automatically by default
      
      // Title/header
      it.print(0, 0, id(my_font), "Title");
      
      // Main content
      it.printf(0, 15, id(my_font), "Value: %.1f", id(my_sensor).state);
      
      // Footer/status
      it.print(0, 55, id(my_font), "Status: OK");
```

## Quick Reference

**Drawing**: line, rectangle, circle, triangle, polygon, pixel  
**Text**: print, printf, strftime  
**Display info**: get_width(), get_height()  
**Colors**: COLOR_ON, COLOR_OFF, Color(r, g, b)  
**Clipping**: start_clipping, end_clipping, extend_clipping, shrink_clipping  
**Pages**: pages list, display.page.show_next, display.page.show  
**Alignment**: TOP_LEFT, CENTER, BOTTOM_RIGHT, etc.  
**Format**: %.1f (decimal), %d (int), %s (string), %% (percent)

