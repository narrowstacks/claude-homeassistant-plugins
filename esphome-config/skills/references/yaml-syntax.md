# ESPHome YAML Syntax Features

## Standard YAML Features

### Comments
Any text after `#` is a comment. To include `#` in a string, it must be quoted.

```yaml
# This is a comment
foo:
  bar: 3  # Another comment
  text: "# This is part of the string"
```

### Scalars
Scalars are values that don't contain a colon. They can be strings, numbers, booleans, or null.

Strings can be quoted (`"` or `'`) or unquoted. Escape sequences like `\n` work in double quotes only. Unquoted strings that aren't valid numbers or booleans are treated as strings (e.g., `23times`).

Boolean values are `true` or `false` (case-insensitive). ESPHome also recognizes `yes`, `on`, `enable` as `true` and `no`, `off`, `disable` as `false`.

Numbers can be integers or floats. Strings containing numbers are automatically converted in most ESPHome contexts.

```yaml
esp8266:
  board: esp8285           # String
  restore_from_flash: true # Boolean
web_server:
  port: 80                 # Integer
```

### Sequences
Lists using `-` or `[...]` syntax. Items can be scalars, sequences, or mappings.

```yaml
# JSON style (single line)
data_pins: [48, 47, 39, 40]

# YAML style (multiple lines)
data_pins:
  - 48
  - 47
  - 39
  - 40

# List of mappings
sensors:
  - platform: gpio
    name: "Temperature 1"
    pin: GPIO32
  - platform: gpio
    name: "Temperature 2"
    pin: GPIO33
```

**Important:** When a sequence item ends with a colon (e.g., `- label:`), its value must be a mapping with further indentation.

```yaml
# Correct
- label:
    text: "Temperature 1"

# Incorrect (will throw errors)
- label:
  text: "Temperature 1"
```

### Mappings
Key-value pairs using `key: value` or `{...}` syntax. Keys must be unique in a mapping.

```yaml
sensor:
  platform: gpio
  pin: GPIO32
  name: "Temperature 1"
  device_class: temperature
  unit_of_measurement: "°C"
```

### Anchors and Aliases
Define a block once with `&anchor` and reuse with `*alias`. Override specific values with `<<: *anchor`.

```yaml
sensor:
  - &common_adc
    pin: GPIO32
    platform: adc
    name: "Temperature 1"
    device_class: temperature
    unit_of_measurement: "°C"
    accuracy_decimals: 1
  - <<: *common_adc
    pin: GPIO33
    name: "Temperature 2"
```

The second sensor inherits all properties from `common_adc` but overrides `pin` and `name`.

### Multi-line Strings

**Quoted multi-line:** Break quoted strings across lines. Escape sequences work in double quotes only.

```yaml
sensor:
  - platform: template
    name: "Sensor
    Name"
```

**Block strings:** Use `|` (literal) or `>` (folded). Literal keeps embedded newlines; folded converts them to spaces. Chomping indicators: `-` removes trailing newlines, `+` keeps them.

```yaml
# Literal (keep internal newlines)
multiline_string: |-
  This is a string that is
  broken across multiple lines.
  Internal newlines will be kept.
```

## ESPHome YAML Extensions

### Secrets
Reference sensitive values from `secrets.yaml` using the `!secret` tag. The secrets file must be a flat mapping of keys to scalar values and should never be checked into version control.

```yaml
# In config.yaml
wifi:
  ssid: "MyWiFi"
  password: !secret wifi_password

# In secrets.yaml
wifi_password: my_super_secret_password
```

### Substitutions
Define reusable values referenced throughout the configuration. See the substitutions component documentation for full details.

```yaml
substitutions:
  device_name: my_device
  
esphome:
  name: ${device_name}
```

### !include
Insert the contents of another YAML file at a specific position. Works at any configuration level. Substitutions in the included file reference values passed to `!include`, overriding global substitutions.

```yaml
binary_sensor:
  - platform: gpio
    id: button1
    pin: GPIOXX
    on_multi_click: !include { file: on-multi-click.yaml, vars: { id: 1 } }
```

Or multi-line syntax:

```yaml
on_multi_click: !include
  file: on-multi-click.yaml
  vars:
    id: 2
```

### Hidden Items
Top-level keys starting with `.` are ignored and excluded from the final configuration. Useful for defining anchors that aren't part of the actual config.

```yaml
.number: &AnchorNumber
  optimistic: true
  min_value: 0
  max_value: 600

number:
  - platform: template
    <<: *AnchorNumber
    id: "SwitchMainDelay"
```

### Lambdas
Embed C++ code blocks for dynamic values and logic not possible in YAML. Defined using the `!lambda` tag.

```yaml
binary_sensor:
  - platform: template
    lambda: |-
      if (id(my_sensor).state > 20) {
        return true;
      } else {
        return false;
      }
```

## General Notes

- YAML is a superset of JSON; JSON syntax can be used in YAML files
- Block order in ESPHome YAML files is generally unimportant; all content is read before validation
- Indentation is critical in YAML; use spaces (not tabs) consistently
