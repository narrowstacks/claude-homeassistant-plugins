# ESPHome Packages System

## Overview

Packages enable configuration modularization by breaking configurations into reusable pieces. Instead of repeating common elements across many devices, define them once in a package and include them in multiple device configurations. All package definitions are merged non-destructively with the device's main configuration, allowing overrides.

## Merging Behavior

- **Dictionaries:** Merged key-by-key
- **Lists of components:** Merged by component ID (if specified)
- **Other lists:** Merged by concatenation
- **All other values:** Replaced with the later value
- **Substitutions:** Values in the main configuration override substitutions with the same name in packages

## Local Packages

Use `!include` to bring packages from local files. Packages can be a list or a mapping of references.

```yaml
# In config.yaml
packages:
  - !include common/wifi.yaml
  - !include common/device_base.yaml

api:
  actions:
    - action: start_laundry
      then:
        - switch.turn_on: relay
```

```yaml
# In common/wifi.yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

```yaml
# In common/device_base.yaml
esphome:
  name: ${node_name}
esp32:
  board: wemos_d1_mini32
logger:
api:
  encryption:
    key: !secret api_encryption_key
```

When a mapping is used, keys are for reference only and have no significance. Single package references can be used directly without a list.

```yaml
# As a mapping
packages:
  wifi: !include common/wifi.yaml
  base: !include common/device_base.yaml

# Single package (not in a list)
packages: !include common.yaml
```

## Remote/Git Packages

Load packages from a Git repository using URL-based configuration. Remote packages cannot use `!secret` lookups; use substitutions with optional defaults instead, allowing the local device YAML to set values from secrets.

```yaml
packages:
  # Shorthand form: github://username/repository/[folder/]file-path.yml[@branch-or-tag]
  remote_package_shorthand: github://esphome/example-repo/file1.yml@main
```

Full syntax with multiple files:

```yaml
packages:
  remote_package_files:
    url: https://github.com/esphome/example-repo
    files: [file1.yml, file2.yml]  # Optional; if not specified, all files included
    ref: main                        # Optional Git reference (branch or tag)
    refresh: 1d                      # Optional refresh interval
```

Advanced syntax with per-file variables:

```yaml
packages:
  remote_package_files:
    url: https://github.com/esphome/example-repo
    files:
      - path: file1.yml
        vars:
          a: 1
          b: 2
      - path: file1.yml  # Same file with different vars
        vars:
          a: 3
          b: 4
      - file2.yml  # No variables for this file
    ref: main
    refresh: 1d
```

## Packages as Templates

Packages can accept variables, making them act as templates for complex or repetitive configurations. Include the same package multiple times with different variable values. Packages may contain a `defaults` block providing substitutions for unspecified variables.

```yaml
# In config.yaml
packages:
  left_garage_door: !include
    file: garage-door.yaml
    vars:
      door_name: Left
  middle_garage_door: !include
    file: garage-door.yaml
    vars:
      door_name: Middle
  right_garage_door: !include
    file: garage-door.yaml
    vars:
      door_name: Right
```

```yaml
# In garage-door.yaml
switch:
  - name: ${door_name} Garage Door Switch
    platform: gpio
    # ... rest of configuration
```

## Extending Configurations

Use `!extend config_id` to modify included configurations where `config_id` is the ID of the component to modify.

```yaml
# In common.yaml
sensor:
  - platform: uptime
    id: uptime_sensor
    update_interval: 1min

# In config.yaml
packages: !include common.yaml

sensor:
  - id: !extend uptime_sensor
    update_interval: 10s  # Override the interval
```

## Removing Configurations

Use `!remove [config_id]` to remove existing entries from included configurations.

Remove a component by ID:

```yaml
packages: !include common.yaml

sensor:
  - id: !remove uptime_sensor
```

Remove an entire configuration block:

```yaml
packages: !include common.yaml

captive_portal: !remove
```

Remove a specific attribute:

```yaml
packages: !include common.yaml

sensor:
  - id: !extend uptime_sensor
    update_interval: !remove
```

## Best Practices

- Use packages for truly common elements shared across multiple devices
- Use meaningful package reference names when using mapping syntax (e.g., `base:`, `wifi:`, `sensors:`)
- For remote packages, prefer using substitutions over secrets to allow local override
- Group logically related configuration elements into single package files
- Use descriptive file names that indicate the configuration's purpose
- Leverage templating with variables for configurations that repeat with minor variations
- Combine `!extend` and `!remove` operations to adapt packages without duplicating configuration
