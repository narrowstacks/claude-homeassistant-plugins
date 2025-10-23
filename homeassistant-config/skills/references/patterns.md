# Common Home Assistant Automation Patterns

This document contains proven patterns for common Home Assistant automation scenarios.

## Lighting Patterns

### Motion-Activated Lights with Timer
```yaml
- alias: "Hallway Motion Light"
  description: "Turn on light on motion, turn off 5 minutes after no motion"
  mode: restart  # Restart timer if motion detected again
  triggers:
    - trigger: state
      entity_id: binary_sensor.hallway_motion
      to: "on"
  actions:
    - action: light.turn_on
      target:
        entity_id: light.hallway
    - wait_for_trigger:
        - trigger: state
          entity_id: binary_sensor.hallway_motion
          to: "off"
    - delay:
        minutes: 5
    - action: light.turn_off
      target:
        entity_id: light.hallway
```

### Adaptive Lighting (Time-Based Brightness)
```yaml
- alias: "Adaptive Bedroom Lighting"
  triggers:
    - trigger: state
      entity_id: binary_sensor.bedroom_motion
      to: "on"
  variables:
    hour: "{{ now().hour }}"
    brightness: >
      {% if hour >= 6 and hour < 12 %}
        80
      {% elif hour >= 12 and hour < 18 %}
        100
      {% elif hour >= 18 and hour < 22 %}
        60
      {% else %}
        20
      {% endif %}
    color_temp: >
      {% if hour >= 6 and hour < 12 %}
        250
      {% elif hour >= 18 and hour < 22 %}
        350
      {% else %}
        454
      {% endif %}
  actions:
    - action: light.turn_on
      target:
        entity_id: light.bedroom
      data:
        brightness_pct: "{{ brightness }}"
        color_temp: "{{ color_temp }}"
```

### Circadian Lighting
```yaml
# Template sensor for dynamic color temperature
template:
  - sensor:
      - name: "Circadian Color Temperature"
        unit_of_measurement: "mired"
        state: >
          {% set sun_elevation = state_attr('sun.sun', 'elevation') | float %}
          {% if sun_elevation > 0 %}
            {{ 250 + (204 * (1 - (sun_elevation / 90))) | int }}
          {% else %}
            454
          {% endif %}

# Automation to apply
- alias: "Update Circadian Lighting"
  triggers:
    - trigger: state
      entity_id: sensor.circadian_color_temperature
  conditions:
    - condition: state
      entity_id: light.living_room
      state: "on"
  actions:
    - action: light.turn_on
      target:
        entity_id: light.living_room
      data:
        color_temp: "{{ states('sensor.circadian_color_temperature') }}"
        transition: 60
```

### Lights Off When Everyone Leaves
```yaml
- alias: "All Lights Off When Away"
  triggers:
    - trigger: state
      entity_id: group.all_persons
      to: "not_home"
      for:
        minutes: 5
  conditions:
    - condition: template
      value_template: >
        {{ states.light | selectattr('state', 'eq', 'on') | list | count > 0 }}
  actions:
    - action: light.turn_off
      target:
        entity_id: all
    - action: notify.mobile_app_phone
      data:
        message: "All lights turned off as everyone left"
```

## Climate Control Patterns

### Smart Thermostat Based on Presence
```yaml
- alias: "Smart Thermostat Control"
  triggers:
    - trigger: state
      entity_id: binary_sensor.anyone_home
    - trigger: time_pattern
      hours: "/1"  # Check every hour
  actions:
    - choose:
        # Someone is home
        - conditions:
            - condition: state
              entity_id: binary_sensor.anyone_home
              state: "on"
          sequence:
            - action: climate.set_temperature
              target:
                entity_id: climate.thermostat
              data:
                temperature: 22
            - action: climate.set_hvac_mode
              target:
                entity_id: climate.thermostat
              data:
                hvac_mode: "auto"
        
        # No one home
        - conditions:
            - condition: state
              entity_id: binary_sensor.anyone_home
              state: "off"
          sequence:
            - action: climate.set_temperature
              target:
                entity_id: climate.thermostat
              data:
                temperature: 18
```

### Window Open, Turn Off Heating
```yaml
- alias: "Window Open - Pause Heating"
  triggers:
    - trigger: state
      entity_id: binary_sensor.bedroom_window
      to: "on"
      for:
        minutes: 2
  conditions:
    - condition: state
      entity_id: climate.bedroom
      state: "heat"
  actions:
    - action: climate.turn_off
      target:
        entity_id: climate.bedroom
    - action: notify.mobile_app_phone
      data:
        message: "Bedroom heating paused - window open"

- alias: "Window Closed - Resume Heating"
  triggers:
    - trigger: state
      entity_id: binary_sensor.bedroom_window
      to: "off"
  actions:
    - action: climate.set_hvac_mode
      target:
        entity_id: climate.bedroom
      data:
        hvac_mode: "auto"
```

## Security Patterns

### Door Left Open Alert
```yaml
- alias: "Front Door Left Open"
  triggers:
    - trigger: state
      entity_id: binary_sensor.front_door
      to: "on"
      for:
        minutes: 5
  actions:
    - action: notify.mobile_app_phone
      data:
        title: "Door Alert"
        message: "Front door has been open for 5 minutes"
        data:
          actions:
            - action: "remind_later"
              title: "Remind in 5 min"
    
    # Continue checking every 5 minutes
    - repeat:
        while:
          - condition: state
            entity_id: binary_sensor.front_door
            state: "on"
          - condition: template
            value_template: "{{ repeat.index <= 6 }}"  # Max 6 reminders
        sequence:
          - delay:
              minutes: 5
          - action: notify.mobile_app_phone
            data:
              message: "Front door still open ({{ repeat.index * 5 }} minutes)"
```

### Alarm When Away and Motion Detected
```yaml
- alias: "Security - Motion When Away"
  triggers:
    - trigger: state
      entity_id: binary_sensor.living_room_motion
      to: "on"
  conditions:
    - condition: state
      entity_id: alarm_control_panel.home
      state: "armed_away"
  actions:
    - action: alarm_control_panel.alarm_trigger
      target:
        entity_id: alarm_control_panel.home
    - action: notify.mobile_app_phone
      data:
        title: "ALARM TRIGGERED"
        message: "Motion detected while armed away!"
        data:
          priority: critical
    - action: camera.snapshot
      target:
        entity_id: camera.living_room
      data:
        filename: "/config/www/snapshots/alarm_{{ now().strftime('%Y%m%d_%H%M%S') }}.jpg"
```

### Nighttime Security Check
```yaml
- alias: "Night Security Check"
  triggers:
    - trigger: time
      at: "22:00:00"
  actions:
    - variables:
        unlocked_doors: >
          {{ states.lock | selectattr('state', 'eq', 'unlocked') | map(attribute='name') | list }}
        open_windows: >
          {{ states.binary_sensor 
             | selectattr('attributes.device_class', 'eq', 'window')
             | selectattr('state', 'eq', 'on')
             | map(attribute='name') | list }}
    
    - if:
        - condition: template
          value_template: "{{ unlocked_doors | count > 0 or open_windows | count > 0 }}"
      then:
        - action: notify.mobile_app_phone
          data:
            title: "Security Check"
            message: >
              {% if unlocked_doors | count > 0 %}
              Unlocked: {{ unlocked_doors | join(', ') }}
              {% endif %}
              {% if open_windows | count > 0 %}
              Open windows: {{ open_windows | join(', ') }}
              {% endif %}
            data:
              actions:
                - action: "lock_all"
                  title: "Lock All Doors"
```

## Presence Detection Patterns

### Welcome Home Routine
```yaml
- alias: "Welcome Home"
  triggers:
    - trigger: state
      entity_id:
        - person.john
        - person.jane
      from: "not_home"
      to: "home"
  conditions:
    - condition: sun
      after: sunset
  actions:
    - action: scene.turn_on
      target:
        entity_id: scene.welcome_home
    - action: notify.mobile_app_phone
      data:
        message: "Welcome home, {{ trigger.to_state.attributes.friendly_name }}!"
    - action: climate.set_temperature
      target:
        entity_id: climate.thermostat
      data:
        temperature: 22
```

### Goodbye Routine
```yaml
- alias: "Goodbye - Last Person Leaves"
  triggers:
    - trigger: state
      entity_id: group.all_persons
      from: "home"
      to: "not_home"
      for:
        minutes: 5
  actions:
    - action: light.turn_off
      target:
        entity_id: all
    - action: lock.lock
      target:
        entity_id: all
    - action: climate.set_hvac_mode
      target:
        entity_id: climate.thermostat
      data:
        hvac_mode: "off"
    - action: media_player.turn_off
      target:
        entity_id: all
```

### Vacation Mode Detection
```yaml
# Binary sensor to detect vacation
template:
  - binary_sensor:
      - name: "Vacation Mode"
        state: >
          {% set away_hours = (now() - states.group.all_persons.last_changed).total_seconds() / 3600 %}
          {{ away_hours > 72 and is_state('group.all_persons', 'not_home') }}

# Automation when vacation detected
- alias: "Vacation Mode Activated"
  triggers:
    - trigger: state
      entity_id: binary_sensor.vacation_mode
      to: "on"
  actions:
    - action: notify.mobile_app_phone
      data:
        title: "Vacation Mode"
        message: "Vacation mode activated. Security measures enabled."
    - action: script.vacation_mode_enable
```

## Notification Patterns

### Actionable Notifications
```yaml
- alias: "Garage Door Open - Actionable Notification"
  triggers:
    - trigger: state
      entity_id: cover.garage_door
      to: "open"
      for:
        minutes: 10
  actions:
    - action: notify.mobile_app_phone
      data:
        title: "Garage Door Open"
        message: "Garage door has been open for 10 minutes"
        data:
          actions:
            - action: "close_garage"
              title: "Close Garage"
            - action: "ignore_garage"
              title: "Ignore"

- alias: "Handle Garage Notification Action"
  triggers:
    - trigger: event
      event_type: mobile_app_notification_action
      event_data:
        action: "close_garage"
  actions:
    - action: cover.close_cover
      target:
        entity_id: cover.garage_door
```

### Daily Summary Notification
```yaml
- alias: "Morning Summary"
  triggers:
    - trigger: time
      at: "08:00:00"
  conditions:
    - condition: state
      entity_id: person.john
      state: "home"
  actions:
    - action: notify.mobile_app_phone
      data:
        title: "Good Morning!"
        message: >
          Weather: {{ states('weather.home') }}
          Temperature: {{ states('sensor.outdoor_temperature') }}°C
          
          Calendar: {{ state_attr('calendar.personal', 'message') }}
          
          Garbage: {{ states('sensor.garbage_day') }}
```

### Critical Alert with Repeat
```yaml
- alias: "Water Leak Alert"
  triggers:
    - trigger: state
      entity_id: binary_sensor.water_leak
      to: "on"
  actions:
    - repeat:
        while:
          - condition: state
            entity_id: binary_sensor.water_leak
            state: "on"
          - condition: template
            value_template: "{{ repeat.index <= 10 }}"
        sequence:
          - action: notify.mobile_app_phone
            data:
              title: "WATER LEAK DETECTED"
              message: "Water leak in {{ trigger.to_state.attributes.friendly_name }}!"
              data:
                priority: critical
          - delay:
              minutes: 2
```

## Energy Management Patterns

### High Power Usage Alert
```yaml
- alias: "High Power Usage"
  triggers:
    - trigger: numeric_state
      entity_id: sensor.total_power
      above: 3000
      for:
        minutes: 5
  actions:
    - action: notify.mobile_app_phone
      data:
        title: "High Power Usage"
        message: >
          Power usage is {{ states('sensor.total_power') }}W
          Top consumers:
          {% set devices = [
            ('Dryer', states('sensor.dryer_power')),
            ('AC', states('sensor.ac_power')),
            ('Oven', states('sensor.oven_power'))
          ] %}
          {% for name, power in devices | sort(attribute=1, reverse=true) %}
          {{ name }}: {{ power }}W
          {% endfor %}
```

### Off-Peak Usage Reminders
```yaml
- alias: "Off-Peak Usage Reminder"
  triggers:
    - trigger: time
      at: "21:00:00"  # Off-peak starts at 21:00
  actions:
    - action: notify.mobile_app_phone
      data:
        message: "Off-peak electricity rates now active. Good time to run dishwasher, laundry, etc."
```

## Media Patterns

### Pause Media When Doorbell Rings
```yaml
- alias: "Doorbell - Pause Media"
  triggers:
    - trigger: state
      entity_id: binary_sensor.doorbell
      to: "on"
  actions:
    - action: media_player.media_pause
      target:
        entity_id: media_player.living_room_tv
    - action: tts.google_say
      target:
        entity_id: media_player.google_home
      data:
        message: "Someone is at the door"
```

### Resume Media After Phone Call
```yaml
# Store state when call starts
- alias: "Phone Call - Pause Media"
  triggers:
    - trigger: state
      entity_id: sensor.phone_state
      to: "ringing"
  actions:
    - action: media_player.media_pause
      target:
        entity_id: media_player.living_room_speaker
    - action: input_boolean.turn_on
      target:
        entity_id: input_boolean.media_was_playing

# Resume when call ends
- alias: "Phone Call End - Resume Media"
  triggers:
    - trigger: state
      entity_id: sensor.phone_state
      from: "offhook"
      to: "idle"
  conditions:
    - condition: state
      entity_id: input_boolean.media_was_playing
      state: "on"
  actions:
    - action: media_player.media_play
      target:
        entity_id: media_player.living_room_speaker
    - action: input_boolean.turn_off
      target:
        entity_id: input_boolean.media_was_playing
```

## Time-Based Patterns

### Weekday vs Weekend Routines
```yaml
- alias: "Morning Alarm"
  triggers:
    - trigger: time
      at: input_datetime.alarm_time
  conditions:
    - condition: state
      entity_id: input_boolean.alarm_enabled
      state: "on"
    - condition: template
      value_template: >
        {% set day = now().weekday() %}
        {% if is_state('input_boolean.weekday_only', 'on') %}
          {{ day < 5 }}
        {% else %}
          true
        {% endif %}
  actions:
    - action: script.morning_routine
```

### Seasonal Adjustments
```yaml
- alias: "Seasonal Light Adjustment"
  triggers:
    - trigger: sun
      event: sunset
  variables:
    month: "{{ now().month }}"
    brightness: >
      {% if month in [12, 1, 2] %}
        100
      {% elif month in [3, 4, 5, 9, 10, 11] %}
        80
      {% else %}
        60
      {% endif %}
  actions:
    - action: light.turn_on
      target:
        entity_id: light.outdoor
      data:
        brightness_pct: "{{ brightness }}"
```

## Multi-Device Coordination

### Synchronize Lights in Multiple Rooms
```yaml
- alias: "Sync Bedroom Lights"
  triggers:
    - trigger: state
      entity_id: light.bedroom_main
      attribute: brightness
  actions:
    - action: light.turn_on
      target:
        entity_id:
          - light.bedroom_lamp_1
          - light.bedroom_lamp_2
      data:
        brightness: "{{ state_attr('light.bedroom_main', 'brightness') }}"
```

### Group Control with Individual Override
```yaml
# Use timer to track manual override
- alias: "Group Light Manual Override"
  triggers:
    - trigger: state
      entity_id:
        - light.bedroom_lamp_1
        - light.bedroom_lamp_2
  conditions:
    - condition: template
      value_template: "{{ trigger.from_state.state != trigger.to_state.state }}"
  actions:
    - action: timer.start
      target:
        entity_id: "timer.{{ trigger.entity_id | replace('.', '_') }}_override"
      data:
        duration: "01:00:00"

# Automation respects override timer
- alias: "Group Control - Check Override"
  triggers:
    - trigger: state
      entity_id: sensor.motion
  conditions:
    - condition: state
      entity_id: timer.light_bedroom_lamp_1_override
      state: "idle"
  actions:
    - action: light.turn_on
      target:
        entity_id: light.bedroom_lamp_1
```
