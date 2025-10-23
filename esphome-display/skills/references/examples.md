# ESPHome Display Advanced Examples

This file contains advanced patterns and complete examples for ESPHome display code.

## Table of Contents

- Multi-page Weather Station
- Animated Loading Indicator
- Scrolling Text
- Graph Rendering
- QR Code Display
- Custom Button Widgets
- Menu System
- Performance Optimization

## Multi-page Weather Station

Complete weather dashboard with multiple pages:

```yaml
# Sensors
sensor:
  - platform: dht
    temperature:
      id: temp_indoor
    humidity:
      id: humidity_indoor
  - platform: bme280
    temperature:
      id: temp_outdoor
    pressure:
      id: pressure

binary_sensor:
  - platform: gpio
    id: next_button
    pin: GPIO0
    on_press:
      - display.page.show_next: weather_display
      - component.update: weather_display

# Fonts
font:
  - file: "fonts/arial.ttf"
    id: font_small
    size: 10
  - file: "fonts/arial.ttf"
    id: font_medium
    size: 14
  - file: "fonts/arial.ttf"
    id: font_large
    size: 20

# Display
display:
  - platform: ssd1306_i2c
    id: weather_display
    model: "SSD1306 128x64"
    pages:
      - id: page_temp
        lambda: |-
          // Header
          it.print(64, 0, id(font_large), TextAlign::TOP_CENTER, "Temperature");
          it.line(0, 22, 128, 22);
          
          // Indoor temp
          it.print(0, 28, id(font_small), "Indoor:");
          it.printf(64, 26, id(font_medium), TextAlign::TOP_CENTER, 
                    "%.1f°C", id(temp_indoor).state);
          
          // Outdoor temp
          it.print(0, 46, id(font_small), "Outdoor:");
          it.printf(64, 44, id(font_medium), TextAlign::TOP_CENTER,
                    "%.1f°C", id(temp_outdoor).state);
          
          // Page indicator
          it.filled_circle(60, 58, 2);
          it.circle(68, 58, 2);
          it.circle(76, 58, 2);
      
      - id: page_humidity
        lambda: |-
          it.print(64, 0, id(font_large), TextAlign::TOP_CENTER, "Humidity");
          it.line(0, 22, 128, 22);
          
          // Large humidity display
          it.printf(64, 28, id(font_large), TextAlign::TOP_CENTER,
                    "%.0f%%", id(humidity_indoor).state);
          
          // Visual bar
          int bar_width = 100;
          int filled = bar_width * id(humidity_indoor).state / 100;
          it.rectangle(14, 50, bar_width, 8);
          it.filled_rectangle(15, 51, filled, 6);
          
          // Page indicator
          it.circle(60, 58, 2);
          it.filled_circle(68, 58, 2);
          it.circle(76, 58, 2);
      
      - id: page_pressure
        lambda: |-
          it.print(64, 0, id(font_large), TextAlign::TOP_CENTER, "Pressure");
          it.line(0, 22, 128, 22);
          
          it.printf(64, 32, id(font_large), TextAlign::CENTER,
                    "%.0f", id(pressure).state);
          it.print(64, 48, id(font_small), TextAlign::TOP_CENTER, "hPa");
          
          // Page indicator
          it.circle(60, 58, 2);
          it.circle(68, 58, 2);
          it.filled_circle(76, 58, 2);

# Auto-cycle pages
interval:
  - interval: 5s
    then:
      - display.page.show_next: weather_display
      - component.update: weather_display
```

## Animated Loading Indicator

Rotating dots animation:

```yaml
globals:
  - id: animation_frame
    type: int
    initial_value: '0'

interval:
  - interval: 100ms
    then:
      - lambda: |-
          id(animation_frame) = (id(animation_frame) + 1) % 8;
      - component.update: my_display

display:
  - platform: ssd1306_i2c
    id: my_display
    lambda: |-
      int center_x = 64, center_y = 32;
      int radius = 15;
      int dot_count = 8;
      
      for (int i = 0; i < dot_count; i++) {
        float angle = (i * 360.0 / dot_count + id(animation_frame) * 45) * 0.017453;
        int x = center_x + cos(angle) * radius;
        int y = center_y + sin(angle) * radius;
        
        // Fade effect based on position
        int size = (i == id(animation_frame)) ? 3 : 2;
        it.filled_circle(x, y, size);
      }
```

## Scrolling Text

Horizontal text scroll:

```yaml
globals:
  - id: scroll_offset
    type: int
    initial_value: '128'

text_sensor:
  - platform: template
    id: long_message
    lambda: 'return {"This is a very long message that needs to scroll"};'

interval:
  - interval: 50ms
    then:
      - lambda: |-
          id(scroll_offset) = id(scroll_offset) - 2;
          if (id(scroll_offset) < -200) {
            id(scroll_offset) = 128;
          }
      - component.update: my_display

display:
  - platform: ssd1306_i2c
    id: my_display
    lambda: |-
      it.printf(id(scroll_offset), 20, id(my_font), 
                "%s", id(long_message).state.c_str());
```

## Graph Rendering

Display sensor history:

```yaml
sensor:
  - platform: dht
    temperature:
      id: temperature
      on_value:
        - component.update: my_display

globals:
  - id: temp_history
    type: float[60]
    initial_value: '{0}'
  - id: history_index
    type: int
    initial_value: '0'

interval:
  - interval: 60s
    then:
      - lambda: |-
          id(temp_history)[id(history_index)] = id(temperature).state;
          id(history_index) = (id(history_index) + 1) % 60;

display:
  - platform: ssd1306_i2c
    id: my_display
    lambda: |-
      // Title
      it.print(0, 0, id(my_font), "Temperature History");
      
      // Current value
      it.printf(128, 0, id(my_font), TextAlign::TOP_RIGHT, 
                "%.1f°C", id(temperature).state);
      
      // Graph area
      int graph_y = 15, graph_h = 45, graph_w = 120;
      it.rectangle(0, graph_y, graph_w, graph_h);
      
      // Find min/max for scaling
      float min_temp = 100, max_temp = -100;
      for (int i = 0; i < 60; i++) {
        if (id(temp_history)[i] != 0) {
          if (id(temp_history)[i] < min_temp) min_temp = id(temp_history)[i];
          if (id(temp_history)[i] > max_temp) max_temp = id(temp_history)[i];
        }
      }
      
      // Draw graph
      float range = max_temp - min_temp;
      if (range > 0) {
        for (int i = 0; i < 59; i++) {
          if (id(temp_history)[i] != 0 && id(temp_history)[i+1] != 0) {
            int x1 = i * graph_w / 60;
            int y1 = graph_y + graph_h - ((id(temp_history)[i] - min_temp) / range * graph_h);
            int x2 = (i + 1) * graph_w / 60;
            int y2 = graph_y + graph_h - ((id(temp_history)[i+1] - min_temp) / range * graph_h);
            it.line(x1, y1, x2, y2);
          }
        }
      }
      
      // Labels
      it.printf(0, graph_y + graph_h, id(font_small), "%.1f", min_temp);
      it.printf(graph_w, graph_y + graph_h, id(font_small), 
                TextAlign::TOP_RIGHT, "%.1f", max_temp);
```

## QR Code Display

Display QR codes:

```yaml
qr_code:
  - id: wifi_qr
    value: "WIFI:T:WPA;S:MyNetwork;P:MyPassword;;"

display:
  - platform: ssd1306_i2c
    id: my_display
    lambda: |-
      it.print(64, 0, id(my_font), TextAlign::TOP_CENTER, "WiFi Login");
      it.qr_code(32, 15, id(wifi_qr), COLOR_ON, 2);
```

## Custom Button Widget

Reusable button drawing function:

```yaml
display:
  - platform: ssd1306_i2c
    id: my_display
    lambda: |-
      // Helper function to draw button
      auto draw_button = [&](int x, int y, int w, int h, 
                             const char* label, bool pressed) {
        if (pressed) {
          it.filled_rectangle(x, y, w, h);
          it.print(x + w/2, y + h/2, id(my_font), 
                   TextAlign::CENTER, COLOR_OFF, label);
        } else {
          it.rectangle(x, y, w, h);
          it.print(x + w/2, y + h/2, id(my_font), 
                   TextAlign::CENTER, label);
        }
      };
      
      // Draw buttons
      draw_button(10, 10, 50, 20, "OK", id(ok_button).state);
      draw_button(68, 10, 50, 20, "Cancel", id(cancel_button).state);
      draw_button(10, 40, 108, 20, "Submit", id(submit_button).state);
```

## Menu System

Interactive menu:

```yaml
globals:
  - id: menu_index
    type: int
    initial_value: '0'
  - id: menu_items
    type: int
    initial_value: '4'

binary_sensor:
  - platform: gpio
    id: up_button
    pin: GPIO12
    on_press:
      - lambda: |-
          id(menu_index) = (id(menu_index) - 1 + id(menu_items)) % id(menu_items);
      - component.update: my_display
  
  - platform: gpio
    id: down_button
    pin: GPIO13
    on_press:
      - lambda: |-
          id(menu_index) = (id(menu_index) + 1) % id(menu_items);
      - component.update: my_display
  
  - platform: gpio
    id: select_button
    pin: GPIO14
    on_press:
      - lambda: |-
          // Handle menu selection
          switch(id(menu_index)) {
            case 0: ESP_LOGD("menu", "Settings selected"); break;
            case 1: ESP_LOGD("menu", "Status selected"); break;
            case 2: ESP_LOGD("menu", "About selected"); break;
            case 3: ESP_LOGD("menu", "Exit selected"); break;
          }

display:
  - platform: ssd1306_i2c
    id: my_display
    lambda: |-
      const char* items[] = {"Settings", "Status", "About", "Exit"};
      
      // Header
      it.print(0, 0, id(font_large), "Menu");
      it.line(0, 18, 128, 18);
      
      // Menu items
      int item_height = 12;
      int start_y = 22;
      
      for (int i = 0; i < id(menu_items); i++) {
        int y = start_y + i * item_height;
        
        if (i == id(menu_index)) {
          // Selected item - inverted
          it.filled_rectangle(0, y - 1, 128, item_height);
          it.print(5, y, id(my_font), COLOR_OFF, items[i]);
          it.print(2, y, id(my_font), COLOR_OFF, ">");
        } else {
          // Normal item
          it.print(5, y, id(my_font), items[i]);
        }
      }
```

## Performance Optimization

### Conditional Rendering

Only redraw changed elements:

```yaml
globals:
  - id: last_temp
    type: float
    initial_value: '0.0'

display:
  - platform: ssd1306_i2c
    id: my_display
    auto_clear_enabled: false  # Manual clearing
    lambda: |-
      // Only update if temperature changed
      if (abs(id(temperature).state - id(last_temp)) > 0.1) {
        // Clear only the temperature area
        it.filled_rectangle(0, 20, 128, 20, COLOR_OFF);
        
        // Redraw temperature
        it.printf(64, 25, id(font_large), TextAlign::CENTER,
                  "%.1f°C", id(temperature).state);
        
        id(last_temp) = id(temperature).state;
      }
```

### Pre-calculated Values

Store complex calculations:

```yaml
globals:
  - id: gauge_points
    type: int[16]
    initial_value: '{0}'

# Calculate once on boot
esphome:
  on_boot:
    - lambda: |-
        // Pre-calculate gauge positions
        for (int i = 0; i < 16; i++) {
          float angle = i * 22.5 * 0.017453;
          id(gauge_points)[i] = 64 + cos(angle) * 30;
        }

display:
  - platform: ssd1306_i2c
    lambda: |-
      // Use pre-calculated values
      for (int i = 0; i < 16; i++) {
        it.draw_pixel_at(id(gauge_points)[i], 32);
      }
```

### Update Interval Tuning

```yaml
display:
  - platform: ssd1306_i2c
    update_interval: 500ms  # Slower for static content
    
# Or manual updates only when needed
sensor:
  - platform: dht
    temperature:
      on_value:
        - component.update: my_display  # Update only on change
```

## Color Display Examples

### Gradient Background

```yaml
display:
  - platform: ili9xxx
    lambda: |-
      // Vertical gradient
      for (int y = 0; y < it.get_height(); y++) {
        int blue = y * 255 / it.get_height();
        auto color = Color(0, 100, blue);
        it.horizontal_line(0, y, it.get_width(), color);
      }
      
      // White text on top
      it.print(120, 160, id(my_font), Color(255,255,255), 
               TextAlign::CENTER, "Hello");
```

### Multi-color Status

```yaml
display:
  - platform: ili9xxx
    lambda: |-
      auto green = Color(0, 255, 0);
      auto yellow = Color(255, 255, 0);
      auto red = Color(255, 0, 0);
      
      float temp = id(temperature).state;
      auto color = (temp < 20) ? green : (temp < 25) ? yellow : red;
      
      it.printf(120, 160, id(font_large), color, TextAlign::CENTER,
                "%.1f°C", temp);
```

## E-Paper Specific

### Partial Update Pattern

```yaml
display:
  - platform: waveshare_epaper
    full_update_every: 30  # Full refresh every 30 updates
    lambda: |-
      // Static elements (drawn once)
      it.print(0, 0, id(my_font), "Temperature Monitor");
      it.line(0, 15, 200, 15);
      
      // Dynamic elements (updated each cycle)
      it.printf(0, 20, id(font_large), "%.1f°C", id(temperature).state);
```

