---
name: esphome-yaml
description: Create and modify ESPHome YAML configurations with proper syntax, packages, modularization, and component structure. Use when writing device configs, creating reusable packages, organizing multi-device deployments, or troubleshooting YAML structure and configuration types (IDs, pins, time periods).
license: Complete terms in LICENSE.txt
---

# ESPHome YAML Configuration

## Overview

ESPHome uses YAML for device configurations. This skill guides writing valid ESPHome configurations with proper syntax, component organization, modularization through packages, and correct usage of configuration types (IDs, pins, time values).

## Core Workflows

### Writing Basic Device Configuration

A typical ESPHome device configuration includes:

1. **ESPHome metadata** (name, platform, board)
2. **Connectivity** (Wi-Fi or Ethernet)
3. **Common infrastructure** (logger, API, web server)
4. **Components** (sensors, switches, lights, binaries)
5. **Automations and actions**

```yaml
esphome:
  name: my_device
  friendly_name: "My Device"

esp32:
  board: wemos_d1_mini32

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

logger:

api:
  encryption:
    key: !secret api_encryption_key

web_server:
  port: 80

sensor:
  - platform: adc
    pin: GPIO32
    name: "Temperature"
    update_interval: 60s
```

### Organizing Modular Configurations with Packages

When managing multiple devices, avoid repetition by extracting common configuration into packages. Packages are included files that merge with the main configuration.

```yaml
# config.yaml
packages:
  base: !include common/base.yaml
  wifi: !include common/wifi.yaml
  sensors: !include common/sensors.yaml

# Device-specific overrides
substitutions:
  device_name: living_room_sensor
```

```yaml
# common/base.yaml
esphome:
  name: ${device_name}
  friendly_name: "${device_name}"

esp32:
  board: wemos_d1_mini32

logger:

api:
  encryption:
    key: !secret api_encryption_key
```

Use templates with variables for configurations that repeat with variations:

```yaml
# config.yaml with three garage doors
packages:
  left_door: !include garage-door.yaml
    vars:
      door_name: Left
  center_door: !include garage-door.yaml
    vars:
      door_name: Center
  right_door: !include garage-door.yaml
    vars:
      door_name: Right
```

```yaml
# garage-door.yaml (template)
switch:
  - platform: gpio
    id: ${door_name}_switch
    name: "${door_name} Door"
    pin: GPIO${door_pin}
```

### Using IDs to Connect Components

Components reference each other through IDs. Define an output with an ID, then reference it in a light or switch.

```yaml
output:
  - platform: pwm
    id: kitchen_light_output
    pin: GPIO5
    frequency: 1000 Hz

light:
  - platform: monochromatic
    id: kitchen_light
    name: "Kitchen Light"
    output: kitchen_light_output
    # Now the light controls the PWM output
```

IDs must follow C++ naming rules: start with a letter, use underscores, no spaces or special characters, not C++ keywords.

### Protecting Sensitive Values with Secrets

Store passwords, API keys, and Wi-Fi credentials in `secrets.yaml`, separate from version control.

```yaml
# config.yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

api:
  encryption:
    key: !secret api_encryption_key
```

```yaml
# secrets.yaml (keep out of git)
wifi_ssid: "My WiFi"
wifi_password: "super_secret_password"
api_encryption_key: "ABABAB..."
```

## Syntax and Structure

### Indentation and Lists

Proper indentation is critical in YAML. Use spaces (not tabs). When a sequence item ends with a colon, its value must be indented further:

```yaml
# Correct: list of mappings
sensors:
  - platform: gpio
    name: "Sensor 1"
    pin: GPIO32
  - platform: gpio
    name: "Sensor 2"
    pin: GPIO33
```

```yaml
# Correct: nested mapping after key
binary_sensor:
  - label:
      text: "Button"
```

### Reusing Configuration with Anchors

Use YAML anchors (`&name`) and aliases (`*name`) to avoid duplication, especially for repeated metadata:

```yaml
sensor:
  - &adc_common
    platform: adc
    device_class: temperature
    unit_of_measurement: "°C"
    accuracy_decimals: 1
  - <<: *adc_common
    id: sensor1
    pin: GPIO32
    name: "Temperature 1"
  - <<: *adc_common
    id: sensor2
    pin: GPIO33
    name: "Temperature 2"
```

### Multi-line Text

Use block strings for readable multi-line text:

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

## Common Configuration Reference

For detailed syntax reference, see:

- **YAML Features and Extensions**: See [yaml-syntax.md](references/yaml-syntax.md) for comments, scalars, sequences, mappings, anchors, multi-line strings, secrets, substitutions, !include, hidden items, and lambdas

- **Packages and Modularization**: See [packages.md](references/packages.md) for local packages, remote git packages, templating with variables, extending, and removing configurations

- **Configuration Types**: See [configuration-types.md](references/configuration-types.md) for ID naming rules, pin specifications, and time period formats

## Debugging Common Issues

**Indentation errors** - Ensure consistent space indentation (no tabs). Sequence items starting with `-` should be at the same indentation as their parent key.

**ID conflicts** - IDs must be unique across the configuration. ESPHome warns if you use the same ID twice.

**Invalid ID names** - IDs must start with a letter and contain only alphanumerics and underscores, not C++ keywords.

**Pin already in use** - By default, ESPHome flags if a pin is used by multiple components. Use `allow_other_uses: true` in the pin schema only if truly necessary.

**Merge conflicts in packages** - Remember that values in the main configuration override package values. Use `!extend` to modify included configuration rather than replacing it.

**Secrets not found** - Ensure the secrets file is in the same directory as the config file and uses simple `key: value` mappings only.
