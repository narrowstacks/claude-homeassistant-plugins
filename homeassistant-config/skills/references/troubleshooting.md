# Home Assistant Configuration Troubleshooting Guide

## YAML Syntax Errors

### Error: "found character '\t' that cannot start any token"
**Cause**: Tab character used instead of spaces

**Solution**: Replace all tabs with spaces (2 spaces per indentation level)

```yaml
# Wrong - uses tab
automation:
	- alias: "Test"

# Correct - uses 2 spaces
automation:
  - alias: "Test"
```

### Error: "mapping values are not allowed here"
**Cause**: Incorrect indentation or missing colon

**Solution**: Check indentation and ensure all keys have colons

```yaml
# Wrong - missing colon
automation
  - alias: "Test"

# Correct
automation:
  - alias: "Test"
```

### Error: "could not find expected ':'"
**Cause**: Missing colon after key

**Solution**: Add colon after the key

```yaml
# Wrong
automation
  alias "Test"

# Correct
automation:
  - alias: "Test"
```

### Error: "expected <block end>, but found '<scalar>'"
**Cause**: Incorrect indentation or list structure

**Solution**: Check indentation consistency

```yaml
# Wrong - inconsistent indentation
automation:
  - alias: "Test"
  triggers:
      - trigger: state

# Correct
automation:
  - alias: "Test"
    triggers:
      - trigger: state
```

### Boolean Interpretation Issues
**Problem**: Values like `on`, `off`, `yes`, `no` interpreted as booleans

**Solution**: Quote the values

```yaml
# Wrong - 'on' becomes boolean true
state: on

# Correct - 'on' stays as string
state: 'on'
state: "on"
```

## Automation Issues

### Automation Not Triggering

**Check 1: Verify entity ID exists**
```yaml
# In Developer Tools > States, search for the entity
# Make sure spelling is exact (case-sensitive)
```

**Check 2: Enable debug logging**
```yaml
logger:
  default: info
  logs:
    homeassistant.components.automation: debug
```

**Check 3: Verify trigger syntax**
```yaml
# Old syntax (deprecated)
trigger:
  platform: state
  entity_id: sensor.temp

# New syntax (current)
triggers:
  - trigger: state
    entity_id: sensor.temp
```

**Check 4: Verify conditions are met**
```yaml
# Add condition logging
conditions:
  - condition: template
    value_template: |
      {{ states('sensor.temp') | float > 20 }}
  # Check template in Developer Tools > Template
```

**Check 5: Check automation is enabled**
```yaml
# In UI: Settings > Automations & Scenes
# Ensure toggle is ON for the automation
```

### Automation Triggers Too Often

**Problem**: Automation triggers on every state change

**Solution**: Add `for` duration
```yaml
triggers:
  - trigger: state
    entity_id: sensor.temperature
    above: 25
    for:
      minutes: 5  # Only trigger if maintained for 5 minutes
```

**Problem**: Template triggers on every update

**Solution**: Use `from`/`to` or compare previous values
```yaml
triggers:
  - trigger: template
    value_template: "{{ states('sensor.temp') | float > 25 }}"
  # Add condition to check if actually changed
conditions:
  - condition: template
    value_template: >
      {{ trigger.from_state.state != trigger.to_state.state }}
```

### Automation Runs Multiple Times Simultaneously

**Problem**: Multiple triggers cause overlapping runs

**Solution**: Use appropriate mode
```yaml
mode: single  # Don't run if already running (default)
# or
mode: restart  # Restart if triggered again
# or
mode: queued  # Queue runs
max: 5
```

## Template Issues

### Template Syntax Error

**Error**: "TemplateSyntaxError"

**Solution**: Test template in Developer Tools > Template

```jinja
# Wrong - missing quotes
{{ states(sensor.temp) }}

# Correct
{{ states('sensor.temp') }}
```

### Entity Not Found Error

**Error**: Entity returns 'unknown' or 'unavailable'

**Solution**: Handle missing states
```jinja
# Wrong - fails if entity missing
{{ states('sensor.temp') | float }}

# Correct - provides default value
{{ states('sensor.temp') | float(0) }}

# Or check if available
{% if states('sensor.temp') not in ['unknown', 'unavailable'] %}
  {{ states('sensor.temp') }}
{% else %}
  N/A
{% endif %}
```

### Type Conversion Issues

**Problem**: Cannot compare string and number

**Solution**: Convert types explicitly
```jinja
# Wrong - comparing string to number
{{ states('sensor.temp') > 20 }}

# Correct - convert to float
{{ states('sensor.temp') | float(0) > 20 }}
```

### Template Too Complex

**Problem**: Template timeout or error

**Solution**: Simplify template or use helper entities
```jinja
# Instead of complex template in automation
# Create template sensor first

template:
  - sensor:
      - name: "Complex Calculation"
        state: >
          {{ complex_calculation_here }}

# Then use in automation
triggers:
  - trigger: state
    entity_id: sensor.complex_calculation
```

## Script Issues

### Script Not Running

**Check 1: Verify script exists**
```yaml
# Check in Developer Tools > Actions
# Look for script.script_name
```

**Check 2: Check script mode**
```yaml
# If script is already running
mode: single  # Won't run again
mode: restart  # Will restart
mode: parallel  # Will run in parallel
```

**Check 3: Verify parameters**
```yaml
# When calling script with parameters
action: script.my_script
data:
  message: "Test"  # Ensure parameter names match field names
```

### Script Fails Partway Through

**Check 1: Enable continue on error**
```yaml
sequence:
  - action: light.turn_on
    continue_on_error: true  # Don't stop on error
    target:
      entity_id: light.missing
  - action: notify.notify
    data:
      message: "This will still run"
```

**Check 2: Check entity states**
```yaml
# Add conditions before actions
sequence:
  - condition: state
    entity_id: light.bedroom
    state: 'on'
  - action: light.turn_off
    target:
      entity_id: light.bedroom
```

## Integration Issues

### Integration Not Loading

**Check configuration syntax**
```yaml
# Check Developer Tools > YAML
# Look for red X next to integration
```

**Check logs**
```yaml
# In Settings > System > Logs
# Look for errors related to integration
```

**Verify requirements**
```yaml
# Some integrations require additional setup
# Check integration documentation
```

### Entity Not Available

**Problem**: Entity shows as 'unavailable'

**Possible causes**:
1. Device offline or disconnected
2. Integration not configured properly
3. Network issues
4. Authentication failed

**Solution**: Check device connectivity and integration status

### State Not Updating

**Problem**: Entity state doesn't change

**Check polling interval**
```yaml
# Some integrations have scan_interval
sensor:
  - platform: some_platform
    scan_interval: 60  # Seconds between updates
```

**Force update**
```yaml
# In Developer Tools > Actions
# Call homeassistant.update_entity
# entity_id: sensor.your_sensor
```

## Performance Issues

### Home Assistant Slow

**Check 1: Review recorder settings**
```yaml
recorder:
  purge_keep_days: 7  # Don't keep too much history
  exclude:
    domains:
      - automation
      - weblink
      - updater
    entities:
      - sensor.high_frequency_sensor
```

**Check 2: Reduce automation frequency**
```yaml
# Wrong - runs every second
triggers:
  - trigger: time_pattern
    seconds: "*"

# Better - runs every 5 minutes
triggers:
  - trigger: time_pattern
    minutes: "/5"
```

**Check 3: Optimize templates**
```yaml
# Wrong - iterates all entities
{{ states | count }}

# Better - iterates specific domain
{{ states.sensor | count }}
```

**Check 4: Check database size**
```bash
# SSH into Home Assistant
du -h /config/home-assistant_v2.db
# If large, reduce purge_keep_days
```

### High CPU Usage

**Identify culprit**
```yaml
# Enable profiling
logger:
  default: info
  logs:
    homeassistant.helpers.script: debug
    homeassistant.helpers.template: debug
```

**Common causes**:
1. Too many template sensors updating frequently
2. Complex templates in automations
3. Integrations polling too often
4. Too many automations triggering

## Network Issues

### Cannot Access Home Assistant

**Check 1: Verify IP address**
```bash
# SSH into system
hostname -I
```

**Check 2: Check port**
```yaml
# In configuration.yaml
http:
  server_port: 8123  # Default port
```

**Check 3: Check SSL settings**
```yaml
http:
  ssl_certificate: /ssl/fullchain.pem
  ssl_key: /ssl/privkey.pem
# Ensure certificates are valid
```

### API Integration Not Working

**Check authentication**
```yaml
# Long-lived access token required
# Generate in: Profile > Security > Long-Lived Access Tokens
```

**Check URL format**
```yaml
# Correct format
http://homeassistant.local:8123/api/
https://yourdomain.com/api/

# Include /api/ path
```

## Common Error Messages

### "Error rendering template"
**Cause**: Template syntax error or missing entity

**Solution**: Test template in Developer Tools > Template

### "Entity not found"
**Cause**: Typo in entity ID or entity doesn't exist

**Solution**: Check entity ID in Developer Tools > States

### "Service not found"
**Cause**: Using old service name or typo

**Solution**: Check available services in Developer Tools > Actions

```yaml
# Old
service: light.turn_on

# New
action: light.turn_on
```

### "Invalid config"
**Cause**: YAML syntax error or invalid configuration

**Solution**: 
1. Run configuration check (Developer Tools > YAML)
2. Review error message for specific line
3. Check indentation and syntax

### "Template variable warning"
**Cause**: Using undefined variable in template

**Solution**: Define variable or check existence
```jinja
# Wrong
{{ my_variable }}

# Correct - check if defined
{% if my_variable is defined %}
  {{ my_variable }}
{% endif %}

# Or provide default
{{ my_variable | default('default_value') }}
```

## Backup and Recovery

### Restore from Backup

**If Home Assistant won't start**:
1. Boot into recovery mode
2. Restore from snapshot
3. Check logs for errors

**If specific automation broken**:
1. Check git history if using version control
2. Restore from backup
3. Disable automation and troubleshoot

### Configuration Validation Failed

**Cannot restart after change**:
1. SSH into Home Assistant
2. Check configuration manually:
   ```bash
   hass --script check_config -c /config
   ```
3. Review error messages
4. Fix issues or restore backup

## Getting Help

### Information to Provide

When asking for help, include:
1. Home Assistant version
2. Installation type (OS, Container, Core)
3. Relevant configuration (with secrets removed)
4. Error messages from logs
5. Steps to reproduce issue

### Where to Get Help

- Home Assistant Community: https://community.home-assistant.io/
- Home Assistant Discord: https://discord.gg/home-assistant
- Home Assistant Reddit: https://reddit.com/r/homeassistant
- GitHub Issues: https://github.com/home-assistant/core/issues

### Best Practices for Troubleshooting

1. **Change one thing at a time**: Don't make multiple changes simultaneously
2. **Check configuration**: Always validate configuration before restarting
3. **Review logs**: Check home-assistant.log for errors
4. **Test in isolation**: Create test automation to isolate issues
5. **Use version control**: Commit working configuration
6. **Document changes**: Keep notes of what you changed
7. **Search first**: Many issues already documented and solved
8. **Provide details**: When asking for help, include all relevant information

## Diagnostic Tools

### Configuration Check
```yaml
# Developer Tools > YAML > Check Configuration
# Shows errors before restarting
```

### Template Editor
```yaml
# Developer Tools > Template
# Test Jinja2 templates with live data
```

### State Inspection
```yaml
# Developer Tools > States
# View all entity states and attributes
```

### Service Testing
```yaml
# Developer Tools > Actions
# Test service calls before adding to automations
```

### Event Monitoring
```yaml
# Developer Tools > Events
# Monitor events in real-time
```

### Logs
```yaml
# Settings > System > Logs
# View error and warning messages
# Filter by integration or severity
```
