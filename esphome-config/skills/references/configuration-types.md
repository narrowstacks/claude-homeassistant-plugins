# ESPHome Configuration Types

## IDs

IDs connect components across different domains. For example, define an output component with an ID, then reference that ID in a light component. IDs must be unique within a configuration, and ESPHome warns if you attempt to use the same ID twice.

Since ESPHome converts configuration to C++ code and IDs become C++ variable names, they must follow C++ naming conventions:

- Must start with a letter and can contain numbers at the end
- Cannot contain spaces
- Can contain underscores (`_`); no other special characters allowed
- Cannot be C++ keywords

```yaml
output:
  - platform: gpio
    id: my_output  # Valid: starts with letter, contains underscore

light:
  - platform: binary
    id: my_light_1  # Valid: contains number at end
    output: my_output
```

IDs are used only within ESPHome and are not translated to Home Assistant Entity IDs.

## Pins

ESPHome uses chip-internal GPIO numbers (integers like 16). Prefix with `GPIO` if desired (e.g., `GPIO16` or just `16`).

Most boards have aliases for certain pins. For example, NodeMCU ESP8266 uses pin names `D0` through `D8` as aliases for internal GPIO numbers. Using an alias or the GPIO number produces the same result.

```yaml
some_config:
  pin: GPIO16

some_config:
  # NodeMCU ESP8266 alias for GPIO16
  pin: D0
```

### Advanced Pin Schema

For more control, use an advanced pin schema as a mapping:

```yaml
pin:
  number: GPIO32
  inverted: false
  mode:
    input: true
    pullup: true
```

**Configuration variables:**

- `number` (Required, pin): The GPIO pin number
- `inverted` (Optional, boolean): Treat all read/write values as inverted. Default: `false`
- `allow_other_uses` (Optional, boolean): Allow the pin to be used elsewhere (rare cases). Default: `false`
- `mode` (Optional, string or mapping): Configures pin behavior. Can use shorthand strings or a mapping with individual settings:
  - `input` (Optional, boolean): Configure as input
  - `output` (Optional, boolean): Configure as output
  - `pullup` (Optional, boolean): Activate internal pullup resistor
  - `pulldown` (Optional, boolean): Activate internal pulldown resistor
  - `open_drain` (Optional, boolean): Set pin to open-drain (results in high-impedance when active)

**Shorthand mode strings:**
- `INPUT`
- `OUTPUT`
- `OUTPUT_OPEN_DRAIN`
- `ANALOG`
- `INPUT_PULLUP`
- `INPUT_PULLDOWN`
- `INPUT_OUTPUT_OPEN_DRAIN`

**Advanced ESP32 options:**

- `drive_strength` (Optional, string): Pad drive strength (maximum current). Default: `20mA`. Options: `5mA`, `10mA`, `20mA`, `40mA`. ESP-IDF framework only.
- `ignore_strapping_warning` (Optional, boolean): Suppress warnings for strapping pins (pins read on reset for chip configuration). Default: `false`. Set to `true` only if absolutely certain the pin won't cause problems.

## Time

Time periods appear in many ESPHome configurations. Multiple formats are supported:

```yaml
# Shorthand formats
some_time_option: 1000us     # Microseconds
some_time_option: 1000ms     # Milliseconds
some_time_option: 1.5s       # Seconds (decimal)
some_time_option: 0.5min     # Minutes
some_time_option: 2h         # Hours
some_time_option: '2:01'     # Hours:minutes (quote for safety)
some_time_option: '2:01:30'  # Hours:minutes:seconds
```

**Mapping format** (components separated):

```yaml
some_time_option:
  milliseconds: 10
  seconds: 30
  minutes: 25
  hours: 3
  days: 0
```

**Special values for update intervals:**

- `never` - Never update
- `0ms` - Update in every loop iteration
- `always` - Same as `0ms`

**Examples:**

```yaml
sensor:
  - platform: bmp280
    update_interval: 1min    # Update every minute

binary_sensor:
  - platform: gpio
    filters:
      - delayed_on: 100ms    # Delay of 100 milliseconds
```

## Common Usage Patterns

### IDs with Components

```yaml
# Define an output with an ID
output:
  - platform: pwm
    id: light_output
    pin: GPIO5

# Reference the output in a light component
light:
  - platform: monochromatic
    id: my_light
    output: light_output
    name: "Living Room Light"
```

### Pin Configurations

```yaml
# Simple usage
binary_sensor:
  - platform: gpio
    pin: GPIO23

# Advanced usage
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO23
      inverted: true
      mode: INPUT_PULLUP
```

### Time Configurations

```yaml
sensor:
  - platform: bmp280
    temperature:
      name: "Temperature"
      filters:
        - offset: -3.0
    pressure:
      name: "Pressure"
    # Update temperature and pressure every 10 seconds
    update_interval: 10s
```
