# Home Assistant Configuration Best Practices

## File Organization

### Split Your Configuration
Don't keep everything in configuration.yaml. Split into logical files:

```yaml
# Good structure
configuration.yaml      # Main file with includes
automations.yaml       # All automations or directory
scripts.yaml          # All scripts or directory
secrets.yaml          # Passwords and API keys
customize.yaml        # Entity customizations
groups.yaml           # Groups
scenes.yaml           # Scenes

# Directories
automations/          # Separate automations by room/function
scripts/              # Reusable scripts
sensors/              # Template and regular sensors
packages/             # Package-based organization
```

### Use Meaningful Names
```yaml
# Bad
- alias: "Automation 1"
  
# Good
- alias: "Front Door - Turn on Entry Lights at Night"
```

### Add Descriptions
```yaml
- alias: "Morning Routine"
  description: "Gradual wake-up with lights and music on weekdays"
  triggers:
    - trigger: time
      at: "07:00:00"
```

## YAML Syntax

### Indentation
- Always use 2 spaces (never tabs)
- Be consistent throughout your configuration

```yaml
# Correct
automation:
  - alias: "Test"
    triggers:
      - trigger: state
        entity_id: sensor.test
        
# Wrong - inconsistent indentation
automation:
- alias: "Test"
  triggers:
    - trigger: state
      entity_id: sensor.test
```

### Quoting Strings
Quote strings that might be interpreted as booleans or contain special characters:

```yaml
# Required quotes
state: 'on'     # Not on (boolean true)
state: 'off'    # Not off (boolean false)
state: 'yes'    # Not yes (boolean true)
state: 'no'     # Not no (boolean false)

# Good practice
name: "Living Room Light"
message: "Temperature: {{ states('sensor.temp') }}°C"
```

### Comments
Use comments to explain complex logic:

```yaml
- alias: "Adaptive Lighting"
  triggers:
    - trigger: state
      entity_id: binary_sensor.motion
  actions:
    # Adjust brightness based on time of day
    # Morning: 80%, Afternoon: 100%, Evening: 60%, Night: 20%
    - action: light.turn_on
      data:
        brightness_pct: >
          {% set hour = now().hour %}
          {% if hour < 12 %}80{% elif hour < 18 %}100{% elif hour < 22 %}60{% else %}20{% endif %}
```

## Secrets Management

### Never Commit Secrets
Use secrets.yaml for sensitive data:

```yaml
# secrets.yaml (add to .gitignore)
wifi_password: "MySecurePassword123"
api_key_openweather: "abc123def456"
latitude: 37.7749
longitude: -122.4194

# configuration.yaml
http:
  api_password: !secret wifi_password

weather:
  - platform: openweathermap
    api_key: !secret api_key_openweather
```

### Environment Variables
For Docker/container deployments:

```yaml
# Only works in Home Assistant Core installations
database:
  db_url: !env_var DATABASE_URL postgresql://user:pass@localhost/hass
```

## Automations

### Use Descriptive Aliases
```yaml
# Bad
- alias: "auto1"

# Good
- alias: "Bathroom Light - Motion Activated with 5min Timer"
```

### Choose Appropriate Modes
```yaml
# Default: single - don't run if already running
mode: single

# Restart timer when triggered again (motion sensors)
mode: restart

# Allow multiple simultaneous runs (notifications)
mode: parallel
max: 10

# Queue runs to execute sequentially
mode: queued
max: 5
```

### Handle Edge Cases
```yaml
# Use float filter with default for unavailable sensors
- condition: numeric_state
  entity_id: sensor.temperature
  above: "{{ states('sensor.threshold') | float(20) }}"

# Check if entity exists before using
- condition: template
  value_template: >
    {{ states('sensor.test') not in ['unavailable', 'unknown'] }}
```

### Use Variables for Reusability
```yaml
- alias: "Example with Variables"
  variables:
    notification_service: mobile_app_phone
    temperature_threshold: 25
  triggers:
    - trigger: numeric_state
      entity_id: sensor.temperature
      above: "{{ temperature_threshold }}"
  actions:
    - action: "notify.{{ notification_service }}"
      data:
        message: "Temperature exceeded {{ temperature_threshold }}°C"
```

## Scripts

### Design for Reusability
```yaml
# Good - accepts parameters
notify_with_priority:
  fields:
    message:
      description: "Message to send"
      required: true
    priority:
      description: "normal, high, or critical"
      default: "normal"
  sequence:
    - action: notify.mobile_app_phone
      data:
        message: "{{ message }}"
        data:
          priority: "{{ priority }}"
```

### Use Mode Appropriately
```yaml
# For notifications that can run simultaneously
mode: parallel
max: 10

# For routines that should restart if triggered again
mode: restart

# For sequences that must complete
mode: single
```

### Document Parameters
```yaml
set_light_scene:
  fields:
    entity_id:
      description: "Light entity to control"
      example: "light.living_room"
      required: true
      selector:
        entity:
          domain: light
    brightness:
      description: "Brightness percentage (0-100)"
      example: 80
      default: 100
```

## Templates

### Keep Templates Readable
```yaml
# Bad - hard to read
state: "{{ states.light | selectattr('state', 'eq', 'on') | list | count }}"

# Good - readable
state: >
  {{
    states.light
    | selectattr('state', 'eq', 'on')
    | list | count
  }}
```

### Handle Missing States
```yaml
# Use float() with default value
{{ states('sensor.temperature') | float(0) }}

# Check availability
{% if states('sensor.temperature') not in ['unavailable', 'unknown'] %}
  {{ states('sensor.temperature') }}
{% else %}
  N/A
{% endif %}
```

### Use Appropriate Filters
```yaml
# Convert to number
{{ states('sensor.value') | float }}
{{ states('sensor.value') | int }}

# String manipulation
{{ states('sensor.name') | lower }}
{{ states('sensor.name') | upper }}
{{ states('sensor.name') | title }}

# Rounding
{{ value | round(2) }}

# Lists
{{ list | count }}
{{ list | sum }}
{{ list | average }}
{{ list | max }}
{{ list | min }}
```

## Performance

### Avoid Expensive Operations
```yaml
# Bad - iterates all states every update
state: >
  {% for state in states %}
    ...
  {% endfor %}

# Good - only iterate relevant domain
state: >
  {% for state in states.sensor %}
    ...
  {% endfor %}
```

### Limit Automation Frequency
```yaml
# Use 'for' to avoid rapid triggering
triggers:
  - trigger: state
    entity_id: sensor.temperature
    for:
      minutes: 5  # Must maintain state for 5 minutes

# Use time_pattern sparingly
triggers:
  - trigger: time_pattern
    minutes: "/15"  # Every 15 minutes, not every minute
```

### Optimize Recorder
```yaml
recorder:
  purge_keep_days: 7
  exclude:
    domains:
      - automation
      - updater
    entities:
      - sensor.very_frequent_sensor
```

## Testing and Validation

### Always Check Configuration
Before restarting:
1. Developer Tools > YAML > Check Configuration
2. Review any errors or warnings
3. Fix issues before restarting

### Test Templates
Use Developer Tools > Template to test before deploying:
```yaml
# Test in template editor first
{{ states('sensor.temperature') | float(0) > 25 }}
```

### Use Developer Tools
- **States**: View current entity states and attributes
- **Actions**: Test service calls
- **Template**: Test Jinja2 templates
- **Events**: Monitor events

### Enable Debug Logging
```yaml
logger:
  default: info
  logs:
    homeassistant.components.automation: debug
    homeassistant.components.script: debug
```

## Documentation

### Comment Complex Logic
```yaml
# Calculate dynamic brightness based on time and presence
# Morning (6-12): 80% if home, 20% if away
# Afternoon (12-18): 100% if home, 0% if away
# Evening (18-22): 60% if home, 0% if away
# Night (22-6): 20% if home, 0% if away
brightness_pct: >
  {% set hour = now().hour %}
  {% set home = is_state('binary_sensor.anyone_home', 'on') %}
  ...
```

### Document Dependencies
```yaml
# This automation requires:
# - binary_sensor.front_door
# - light.entry
# - light.hallway
# - sun.sun
- alias: "Front Door Lights"
```

### Keep a Changelog
Track major configuration changes in a CHANGELOG.md file

## Version Control

### Use Git
```bash
# Initialize git repository
git init

# Create .gitignore
echo "secrets.yaml" > .gitignore
echo "*.log" >> .gitignore
echo "*.db" >> .gitignore
echo "*.db-shm" >> .gitignore
echo "*.db-wal" >> .gitignore

# Commit changes
git add .
git commit -m "Initial configuration"
```

### Commit Frequently
- Commit after each working change
- Write descriptive commit messages
- Test before committing

## Backup

### Regular Backups
- Use Home Assistant Google Drive Backup add-on
- Or manual backups to external storage
- Include configuration files and database

### Test Restores
Periodically test that backups can be restored

## Naming Conventions

### Entity IDs
```yaml
# Use descriptive, hierarchical names
sensor.living_room_temperature
light.bedroom_ceiling
binary_sensor.front_door_contact
switch.coffee_maker
```

### Scripts and Automations
```yaml
# Scripts: action_description
script.notify_all
script.morning_routine
script.turn_off_downstairs

# Automations: location_trigger_action
automation.front_door_lights_on_open
automation.bedroom_lights_sunset
```

## Common Pitfalls to Avoid

### Don't Use Complex Templates Everywhere
```yaml
# Bad - template in every light action
- action: light.turn_on
  data:
    brightness: "{{ states('input_number.brightness') | int }}"

# Good - store in variable
variables:
  brightness: "{{ states('input_number.brightness') | int }}"
actions:
  - action: light.turn_on
    data:
      brightness: "{{ brightness }}"
```

### Don't Ignore Warnings
Configuration check warnings often indicate future problems

### Don't Hardcode Values
```yaml
# Bad
temperature: 22

# Good - use input_number for user control
temperature: "{{ states('input_number.target_temp') }}"
```

### Don't Create Infinite Loops
```yaml
# Bad - can create loop
- alias: "Loop Example"
  triggers:
    - trigger: state
      entity_id: light.bedroom
  actions:
    - action: light.toggle  # Can trigger itself!
      entity_id: light.bedroom

# Good - add condition to prevent loop
- alias: "Loop Prevention"
  triggers:
    - trigger: state
      entity_id: sensor.motion
  conditions:
    - condition: state
      entity_id: light.bedroom
      state: 'off'  # Only if currently off
  actions:
    - action: light.turn_on
      entity_id: light.bedroom
```

## Maintenance

### Regular Review
- Review automations quarterly
- Remove unused entities and automations
- Update deprecated syntax
- Check for new integration features

### Monitor Logs
- Review home-assistant.log regularly
- Address warnings and errors
- Remove debugging when not needed

### Stay Updated
- Follow Home Assistant release notes
- Update integrations and add-ons
- Test updates in a dev environment first

## Security

### Secure Your Instance
```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - ::1
  ip_ban_enabled: true
  login_attempts_threshold: 5
```

### Use Authentication
- Enable multi-factor authentication
- Use strong passwords
- Limit trusted networks

### Keep Software Updated
- Update Home Assistant regularly
- Update add-ons and integrations
- Monitor security advisories
