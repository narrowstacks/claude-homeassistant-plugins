# Template Sensor and Binary Sensor Examples
# These demonstrate various template patterns for Home Assistant

template:
  # Regular Sensors
  - sensor:
      # Simple state conversion
      - name: "Temperature in Fahrenheit"
        unique_id: temp_fahrenheit
        unit_of_measurement: "°F"
        device_class: temperature
        state_class: measurement
        state: >
          {{ (states('sensor.temperature_celsius') | float(0) * 1.8) + 32 | round(1) }}
      
      # Count entities
      - name: "Lights On Count"
        unique_id: lights_on_count
        unit_of_measurement: "lights"
        icon: mdi:lightbulb-on
        state: >
          {{ states.light | selectattr('state', 'eq', 'on') | list | count }}
      
      # Average of multiple sensors
      - name: "Average Home Temperature"
        unique_id: avg_home_temp
        unit_of_measurement: "°C"
        device_class: temperature
        state_class: measurement
        state: >
          {% set temps = [
            states('sensor.bedroom_temp') | float(0),
            states('sensor.living_room_temp') | float(0),
            states('sensor.kitchen_temp') | float(0)
          ] %}
          {{ (temps | sum / temps | length) | round(1) }}
      
      # Time-based calculation
      - name: "Time Until Sunset"
        unique_id: time_until_sunset
        icon: mdi:weather-sunset
        state: >
          {% set sunset = state_attr('sun.sun', 'next_setting') | as_datetime %}
          {% set now_time = now() %}
          {% set diff = (sunset - now_time).total_seconds() %}
          {% if diff > 0 %}
            {{ (diff / 3600) | round(1) }} hours
          {% else %}
            Sun has set
          {% endif %}
      
      # Battery level monitoring
      - name: "Low Battery Devices"
        unique_id: low_battery_devices
        icon: mdi:battery-alert
        state: >
          {% set ns = namespace(devices=[]) %}
          {% for state in states %}
            {% if state.attributes.battery_level is defined %}
              {% if state.attributes.battery_level | int < 20 %}
                {% set ns.devices = ns.devices + [state.name] %}
              {% endif %}
            {% endif %}
          {% endfor %}
          {{ ns.devices | length }}
        attributes:
          devices: >
            {% set ns = namespace(devices=[]) %}
            {% for state in states %}
              {% if state.attributes.battery_level is defined %}
                {% if state.attributes.battery_level | int < 20 %}
                  {% set ns.devices = ns.devices + [state.name ~ ' (' ~ state.attributes.battery_level ~ '%)'] %}
                {% endif %}
              {% endif %}
            {% endfor %}
            {{ ns.devices | join(', ') }}
      
      # Time since last motion
      - name: "Living Room Motion Last Seen"
        unique_id: living_room_motion_last
        icon: mdi:motion-sensor
        state: >
          {% set last = states.binary_sensor.living_room_motion.last_changed %}
          {{ relative_time(last) }}
      
      # Energy cost calculation
      - name: "Current Energy Cost"
        unique_id: current_energy_cost
        unit_of_measurement: "$/h"
        icon: mdi:currency-usd
        state: >
          {% set power = states('sensor.total_power') | float(0) %}
          {% set rate = 0.12 %}
          {{ (power / 1000 * rate) | round(2) }}
      
      # Distance calculation
      - name: "Distance from Home"
        unique_id: distance_from_home
        unit_of_measurement: "km"
        icon: mdi:map-marker-distance
        state: >
          {{ distance('device_tracker.phone', 'zone.home') | round(1) }}
      
      # Next alarm time
      - name: "Next Alarm Time"
        unique_id: next_alarm_time
        icon: mdi:alarm
        device_class: timestamp
        state: >
          {% set hour = states('input_number.alarm_hour') | int %}
          {% set minute = states('input_number.alarm_minute') | int %}
          {% set enabled = is_state('input_boolean.alarm_enabled', 'on') %}
          {% if enabled %}
            {{ today_at(hour ~ ':' ~ minute) }}
          {% else %}
            unavailable
          {% endif %}
      
      # Windows open count
      - name: "Windows Open"
        unique_id: windows_open_count
        unit_of_measurement: "windows"
        icon: mdi:window-open
        state: >
          {{
            states.binary_sensor
            | selectattr('attributes.device_class', 'eq', 'window')
            | selectattr('state', 'eq', 'on')
            | list | count
          }}
        attributes:
          open_windows: >
            {{
              states.binary_sensor
              | selectattr('attributes.device_class', 'eq', 'window')
              | selectattr('state', 'eq', 'on')
              | map(attribute='name')
              | list | join(', ')
            }}
      
      # Humidity change rate
      - name: "Bathroom Humidity Change"
        unique_id: bathroom_humidity_change
        unit_of_measurement: "%/min"
        icon: mdi:water-percent
        state: >
          {% set current = states('sensor.bathroom_humidity') | float(0) %}
          {% set history = state_attr('sensor.bathroom_humidity_change', 'previous') | float(0) %}
          {{ ((current - history) * 60) | round(1) }}
        attributes:
          previous: >
            {{ states('sensor.bathroom_humidity') | float(0) }}
      
      # Electricity rate (time-based)
      - name: "Current Electricity Rate"
        unique_id: current_electricity_rate
        unit_of_measurement: "$/kWh"
        icon: mdi:cash
        state: >
          {% set hour = now().hour %}
          {% if hour >= 7 and hour < 11 %}
            0.18
          {% elif hour >= 17 and hour < 21 %}
            0.22
          {% else %}
            0.12
          {% endif %}
      
      # Internet status
      - name: "Internet Connection"
        unique_id: internet_connection
        icon: mdi:web
        state: >
          {% set ping = states('sensor.speedtest_ping') | float(999) %}
          {% if ping < 50 %}
            Excellent
          {% elif ping < 100 %}
            Good
          {% elif ping < 200 %}
            Fair
          {% else %}
            Poor
          {% endif %}
        attributes:
          ping: "{{ states('sensor.speedtest_ping') }}"
          download: "{{ states('sensor.speedtest_download') }}"
          upload: "{{ states('sensor.speedtest_upload') }}"
      
      # Garbage day reminder
      - name: "Garbage Day"
        unique_id: garbage_day
        icon: mdi:delete
        state: >
          {% set day = now().weekday() %}
          {% if day == 0 %}
            Today is garbage day!
          {% elif day == 6 %}
            Garbage day tomorrow
          {% else %}
            {{ 7 - day }} days until garbage day
          {% endif %}
  
  # Binary Sensors
  - binary_sensor:
      # Anyone home
      - name: "Anyone Home"
        unique_id: anyone_home
        device_class: presence
        state: >
          {{
            states.person
            | selectattr('state', 'eq', 'home')
            | list | count > 0
          }}
      
      # All lights off
      - name: "All Lights Off"
        unique_id: all_lights_off
        device_class: light
        state: >
          {{
            states.light
            | selectattr('state', 'eq', 'on')
            | list | count == 0
          }}
      
      # High temperature alert
      - name: "High Temperature Alert"
        unique_id: high_temp_alert
        device_class: problem
        state: >
          {{ states('sensor.outdoor_temperature') | float(0) > 35 }}
      
      # Low temperature alert
      - name: "Low Temperature Alert"
        unique_id: low_temp_alert
        device_class: problem
        state: >
          {{ states('sensor.outdoor_temperature') | float(100) < 0 }}
      
      # Workday
      - name: "Workday"
        unique_id: is_workday
        icon: mdi:briefcase
        state: >
          {% set day = now().weekday() %}
          {{ day < 5 }}
      
      # Nighttime
      - name: "Nighttime"
        unique_id: is_nighttime
        device_class: light
        state: >
          {% set hour = now().hour %}
          {{ hour < 6 or hour >= 22 }}
      
      # Door or window open
      - name: "Any Door or Window Open"
        unique_id: any_door_window_open
        device_class: opening
        state: >
          {{
            states.binary_sensor
            | selectattr('attributes.device_class', 'in', ['door', 'window'])
            | selectattr('state', 'eq', 'on')
            | list | count > 0
          }}
        attributes:
          count: >
            {{
              states.binary_sensor
              | selectattr('attributes.device_class', 'in', ['door', 'window'])
              | selectattr('state', 'eq', 'on')
              | list | count
            }}
          open_items: >
            {{
              states.binary_sensor
              | selectattr('attributes.device_class', 'in', ['door', 'window'])
              | selectattr('state', 'eq', 'on')
              | map(attribute='name')
              | list | join(', ')
            }}
      
      # Motion detected anywhere
      - name: "Motion Detected"
        unique_id: any_motion_detected
        device_class: motion
        state: >
          {{
            states.binary_sensor
            | selectattr('attributes.device_class', 'eq', 'motion')
            | selectattr('state', 'eq', 'on')
            | list | count > 0
          }}
      
      # Rain forecast
      - name: "Rain Expected"
        unique_id: rain_expected
        device_class: moisture
        state: >
          {{ 'rain' in states('weather.home') | lower }}
      
      # Home occupied (presence + motion)
      - name: "Home Occupied"
        unique_id: home_occupied
        device_class: occupancy
        state: >
          {% set people_home = states.person | selectattr('state', 'eq', 'home') | list | count > 0 %}
          {% set recent_motion = states.binary_sensor 
             | selectattr('attributes.device_class', 'eq', 'motion')
             | selectattr('state', 'eq', 'on') 
             | list | count > 0 %}
          {{ people_home and recent_motion }}
      
      # Vacation mode
      - name: "Vacation Mode"
        unique_id: vacation_mode
        icon: mdi:airplane
        state: >
          {% set away_days = (now() - states.person.john.last_changed).days %}
          {{ away_days > 3 and is_state('person.john', 'not_home') }}
      
      # Device offline
      - name: "Critical Device Offline"
        unique_id: critical_device_offline
        device_class: problem
        state: >
          {% set critical = ['sensor.router', 'sensor.nas'] %}
          {% set offline = namespace(found=false) %}
          {% for entity in critical %}
            {% if is_state(entity, 'unavailable') or is_state(entity, 'unknown') %}
              {% set offline.found = true %}
            {% endif %}
          {% endfor %}
          {{ offline.found }}
        attributes:
          offline_devices: >
            {% set critical = ['sensor.router', 'sensor.nas'] %}
            {% set offline = namespace(devices=[]) %}
            {% for entity in critical %}
              {% if is_state(entity, 'unavailable') or is_state(entity, 'unknown') %}
                {% set offline.devices = offline.devices + [state_attr(entity, 'friendly_name')] %}
              {% endif %}
            {% endfor %}
            {{ offline.devices | join(', ') }}
      
      # Bedtime
      - name: "Bedtime"
        unique_id: is_bedtime
        icon: mdi:sleep
        state: >
          {% set hour = now().hour %}
          {% set bedtime_hour = states('input_number.bedtime_hour') | int %}
          {{ hour >= bedtime_hour or hour < 6 }}
      
      # All secure (locks and doors)
      - name: "Home Secured"
        unique_id: home_secured
        device_class: lock
        state: >
          {% set locks_locked = states.lock | selectattr('state', 'eq', 'unlocked') | list | count == 0 %}
          {% set doors_closed = states.binary_sensor 
             | selectattr('attributes.device_class', 'eq', 'door')
             | selectattr('state', 'eq', 'off')
             | list | count == states.binary_sensor 
                | selectattr('attributes.device_class', 'eq', 'door')
                | list | count %}
          {{ locks_locked and doors_closed }}
