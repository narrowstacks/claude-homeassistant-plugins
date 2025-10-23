---
name: homeassistant-config
description: Create, modify, and troubleshoot Home Assistant configuration files including configuration.yaml, automations, scripts, templates, and YAML organization. Use when working with Home Assistant YAML configuration, writing automations with triggers/conditions/actions, creating scripts, using Jinja2 templates, splitting configuration files, or debugging Home Assistant setup.
---

# Home Assistant Configuration

Create and manage Home Assistant configuration files with proper YAML syntax, automations, scripts, and templates.

## When to Use This Skill

Use when working with:
- Main configuration.yaml files and includes
- Automations (triggers, conditions, actions)
- Scripts with parameters
- Jinja2 templates and template sensors
- File organization and splitting
- YAML syntax validation
- Home Assistant troubleshooting

## Core YAML Rules

**Critical syntax requirements:**
- Indentation: 2 spaces per level (never tabs)
- Lists: Start with `-` at same indentation
- Strings: Quote 'on', 'off', 'yes', 'no' to avoid boolean conversion
- Case sensitive: 'on' ≠ 'On' ≠ 'ON'
- Comments: Use `#`

**Example structure:**
```yaml
automation:
  - alias: "Descriptive Name"
    triggers:
      - trigger: state
        entity_id: sensor.example
    actions:
      - action: light.turn_on
```

## File Organization

### Basic Includes
```yaml
# configuration.yaml
automation: !include automations.yaml
script: !include scripts.yaml
sensor: !include sensors.yaml
```

### Directory Includes
```yaml
# Merge as list (files start with -)
automation: !include_dir_list automations/
sensor: !include_dir_merge_list sensors/

# Merge as named dict
script: !include_dir_merge_named scripts/
```

### Secrets Management
Store sensitive data in secrets.yaml (add to .gitignore):
```yaml
# secrets.yaml
api_key: "abc123"

# configuration.yaml
platform: service
api_key: !secret api_key
```

## Automations

### Current Syntax (2024+)
```yaml
- alias: "Name"
  triggers:
    - trigger: [type]
  conditions:
    - condition: [type]
  actions:
    - action: [service]
```

### Common Triggers

**State:** `trigger: state` with `entity_id`, `from`, `to`, `for`
**Time:** `trigger: time` with `at`
**Numeric:** `trigger: numeric_state` with `above`/`below`
**Sun:** `trigger: sun` with `event: sunset/sunrise`
**Template:** `trigger: template` with `value_template`

### Common Actions

**Service call:**
```yaml
- action: light.turn_on
  target:
    entity_id: light.bedroom
  data:
    brightness: 255
```

**Delay:** `delay: "00:05:00"` or `delay: {minutes: 5}`

**Conditional:**
```yaml
- if:
    - condition: state
      entity_id: light.bedroom
      state: "on"
  then:
    - action: light.turn_off
```

**Repeat:**
```yaml
- repeat:
    count: 5
    sequence:
      - action: light.toggle
```

### Modes
- `single`: Don't run if already running (default)
- `restart`: Restart when triggered again
- `parallel`: Run multiple instances (add `max: 10`)
- `queued`: Queue runs (add `max: 5`)

## Scripts

### Basic Structure
```yaml
script_name:
  alias: "Friendly Name"
  mode: single
  fields:
    param_name:
      description: "What this does"
      required: true
      selector:
        text:
  sequence:
    - action: service_name
      data:
        param: "{{ param_name }}"
```

### Calling Scripts
```yaml
# From automation
actions:
  - action: script.script_name
    data:
      param_name: "value"
```

## Jinja2 Templates

### Basic Syntax
```jinja
{{ states('sensor.temperature') }}
{{ state_attr('light.bedroom', 'brightness') }}
{{ is_state('light.bedroom', 'on') }}
```

### Common Functions
```jinja
{{ now() }}
{{ now().hour }}
{{ states('sensor.temp') | float(0) }}
{{ states('sensor.name') | lower }}
```

### Conditionals
```jinja
{% if is_state('light.bedroom', 'on') %}
  Light is on
{% else %}
  Light is off
{% endif %}
```

### Loops and Filters
```jinja
{{ states.light | selectattr('state', 'eq', 'on') | list | count }}
```

### Template Sensors
```yaml
template:
  - sensor:
      - name: "Sensor Name"
        state: >
          {{ states('sensor.input') | float * 2 }}
```

## Validation and Testing

**Before restarting:**
1. Developer Tools > YAML > Check Configuration
2. Review errors/warnings
3. Fix issues

**Test templates:**
Developer Tools > Template (live preview)

**Debug logging:**
```yaml
logger:
  default: info
  logs:
    homeassistant.components.automation: debug
```

## Common Patterns

For detailed automation patterns (motion lighting, presence detection, security, climate control, etc.), see [references/patterns.md](references/patterns.md)

For template sensor examples (calculations, averaging, counting, etc.), see [references/templates.md](references/templates.md)

For troubleshooting specific errors, see [references/troubleshooting.md](references/troubleshooting.md)

For best practices and optimization, see [references/best-practices.md](references/best-practices.md)

## Example Files

Complete working examples in examples/:
- `config.yaml` - Main configuration with includes
- `automations.yaml` - 25+ automation patterns
- `scripts.yaml` - 15+ script examples
- `templates.yaml` - 30+ template sensors
- `secrets.yaml` - Secrets file structure

## Quick Troubleshooting

**Tab character error:** Replace tabs with 2 spaces

**Boolean issue:** Quote 'on', 'off', 'yes', 'no' as strings

**Template error:** Test in Developer Tools > Template, use `| float(0)` for defaults

**Entity not found:** Check exact entity_id in Developer Tools > States

**Automation not triggering:** Enable debug logging, verify entity exists, check conditions

## Instructions for Claude

When helping with Home Assistant configuration:

1. Use current syntax (triggers/actions not trigger/service)
2. Always include proper indentation (2 spaces)
3. Quote boolean-like strings ('on', 'off', etc.)
4. Provide complete, working examples
5. Test templates for validity
6. Add comments for complex logic
7. Handle unavailable states with `| float(0)` or checks
8. Suggest validation before deployment
9. Reference example files for complete patterns
10. Load reference files as needed for detailed guidance
