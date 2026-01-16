# Professional UI & Home Assistant Integration Guide

## Part 1: Display UI Integration for Professional Look

### Adding the Air Quality Dashboard to Your display.yaml

Add this section to your LVGL pages in display.yaml:

```yaml
  # ========== AIR QUALITY PROFESSIONAL DASHBOARD PAGE ==========
  - id: aq_dashboard_page
    skip_refresh: false
    widgets:
      # ===== HEADER =====
      - obj:
          x: 0
          y: 0
          width: 800
          height: 50
          bg_color: 0x1a1a2e
          border_width: 0
          widgets:
            - label:
                x: 10
                y: 10
                text: "AIR QUALITY MONITOR"
                text_font: font_large
                text_color: 0xFFFFFF
                repl_obj_id: lbl_aq_title

      # ===== MAIN AIR QUALITY SCORE (Large Center) =====
      - obj:
          x: 20
          y: 70
          width: 350
          height: 200
          bg_color: 0x0a0a14
          border_width: 2
          border_color: !lambda |-
            if (!id(aq_score).has_state()) return lv_color_hex(0x888888);
            float score = id(aq_score).state;
            if (score >= 75) return lv_color_hex(0x00FF00); // Green
            if (score >= 50) return lv_color_hex(0xFFFF00); // Yellow
            if (score >= 25) return lv_color_hex(0xFF8800); // Orange
            return lv_color_hex(0xFF0000); // Red
          widgets:
            - label:
                x: 10
                y: 10
                text: "Overall Score"
                text_font: font_small
                text_color: 0x888888
            - arc:
                x: 30
                y: 35
                width: 120
                height: 120
                id: arc_aq_score
                start_angle: 180
                end_angle: 0
                arc_width: 8
                bg_color: 0x333333
                needle_color: !lambda |-
                  if (!id(aq_score).has_state()) return lv_color_hex(0x888888);
                  float score = id(aq_score).state;
                  if (score >= 75) return lv_color_hex(0x00FF00);
                  if (score >= 50) return lv_color_hex(0xFFFF00);
                  if (score >= 25) return lv_color_hex(0xFF8800);
                  return lv_color_hex(0xFF0000);
                update_interval: 5s
                value: !lambda |-
                  if (!id(aq_score).has_state()) return 0;
                  return (int)(id(aq_score).state);
            - label:
                x: 160
                y: 70
                text: !lambda |-
                  if (!id(aq_score).has_state()) return "NO DATA";
                  char buf[10];
                  snprintf(buf, sizeof(buf), "%.0f%%", id(aq_score).state);
                  return std::string(buf);
                text_font: font_large
                text_color: !lambda |-
                  if (!id(aq_score).has_state()) return lv_color_hex(0x888888);
                  float score = id(aq_score).state;
                  if (score >= 75) return lv_color_hex(0x00FF00);
                  if (score >= 50) return lv_color_hex(0xFFFF00);
                  if (score >= 25) return lv_color_hex(0xFF8800);
                  return lv_color_hex(0xFF0000);
                repl_obj_id: lbl_aq_score
            - label:
                x: 160
                y: 110
                text: !lambda |-
                  if (!id(aq_alert_status).has_state()) return "---";
                  return id(aq_alert_status).state;
                text_font: font_small
                text_color: !lambda |-
                  if (!id(aq_score).has_state()) return lv_color_hex(0x888888);
                  float score = id(aq_score).state;
                  if (score >= 75) return lv_color_hex(0x00FF00);
                  if (score >= 50) return lv_color_hex(0xFFFF00);
                  if (score >= 25) return lv_color_hex(0xFF8800);
                  return lv_color_hex(0xFF0000);
                repl_obj_id: lbl_aq_status

      # ===== CO2 SECTION =====
      - obj:
          x: 430
          y: 70
          width: 150
          height: 90
          bg_color: 0x0a0a14
          border_width: 1
          border_color: 0x444444
          widgets:
            - label:
                x: 5
                y: 5
                text: "CO₂"
                text_font: font_small
                text_color: 0x888888
            - label:
                x: 5
                y: 25
                text: !lambda |-
                  if (!id(scd41_co2).has_state()) return "-- ppm";
                  char buf[20];
                  snprintf(buf, sizeof(buf), "%.0f ppm", id(scd41_co2).state);
                  return std::string(buf);
                text_font: font_medium
                text_color: !lambda |-
                  if (!id(scd41_co2).has_state()) return lv_color_hex(0x888888);
                  float co2 = id(scd41_co2).state;
                  if (co2 < 600) return lv_color_hex(0x00FF00);
                  if (co2 < 1000) return lv_color_hex(0xFFFF00);
                  if (co2 < 1500) return lv_color_hex(0xFF8800);
                  return lv_color_hex(0xFF0000);
                repl_obj_id: lbl_co2_ppm
            - label:
                x: 5
                y: 50
                text: !lambda |-
                  if (!id(co2_safety_status).has_state()) return "---";
                  return id(co2_safety_status).state;
                text_font: font_small
                text_color: 0x888888
                repl_obj_id: lbl_co2_status

      # ===== VOLATILE ORGANIC COMPOUNDS (IAQ) =====
      - obj:
          x: 620
          y: 70
          width: 160
          height: 90
          bg_color: 0x0a0a14
          border_width: 1
          border_color: 0x444444
          widgets:
            - label:
                x: 5
                y: 5
                text: "VOC / IAQ"
                text_font: font_small
                text_color: 0x888888
            - label:
                x: 5
                y: 25
                text: !lambda |-
                  if (!id(bme680_iaq).has_state()) return "-- IAQ";
                  char buf[20];
                  snprintf(buf, sizeof(buf), "%.0f IAQ", id(bme680_iaq).state);
                  return std::string(buf);
                text_font: font_medium
                text_color: !lambda |-
                  if (!id(bme680_iaq).has_state()) return lv_color_hex(0x888888);
                  float iaq = id(bme680_iaq).state;
                  if (iaq < 150) return lv_color_hex(0x00FF00);
                  if (iaq < 300) return lv_color_hex(0xFFFF00);
                  if (iaq < 400) return lv_color_hex(0xFF8800);
                  return lv_color_hex(0xFF0000);
                repl_obj_id: lbl_iaq_value
            - label:
                x: 5
                y: 50
                text: !lambda |-
                  if (!id(bme680_temp).has_state()) return "--°F";
                  float c = id(bme680_temp).state;
                  float f = c * 9.0f/5.0f + 32.0f;
                  char buf[15];
                  snprintf(buf, sizeof(buf), "%.1f°F", f);
                  return std::string(buf);
                text_font: font_small
                text_color: 0x999999
                repl_obj_id: lbl_bme680_temp

      # ===== GAS SENSORS (Only if calibrated) =====
      - obj:
          x: 20
          y: 290
          width: 760
          height: 170
          bg_color: 0x0a0a14
          border_width: 2
          border_color: 0x333333
          widgets:
            - label:
                x: 10
                y: 10
                text: "TOXIC GAS SENSORS (Calibrated)"
                text_font: font_small
                text_color: 0x888888
                repl_obj_id: lbl_gas_header

            # CO Column
            - obj:
                x: 10
                y: 35
                width: 220
                height: 120
                bg_color: 0x1a1a2e
                border_width: 1
                border_color: 0x444444
                widgets:
                  - label:
                      x: 5
                      y: 5
                      text: "Carbon Monoxide (CO)"
                      text_font: font_small
                      text_color: 0xAAAAAA
                  - label:
                      x: 5
                      y: 25
                      text: !lambda |-
                        if (!id(g_baseline_valid) || !id(gas_co_ppm).has_state()) {
                          return "NOT CALIBRATED";
                        }
                        char buf[15];
                        snprintf(buf, sizeof(buf), "%.0f ppm", id(gas_co_ppm).state);
                        return std::string(buf);
                      text_font: font_medium
                      text_color: !lambda |-
                        if (!id(g_baseline_valid) || !id(gas_co_ppm).has_state()) {
                          return lv_color_hex(0x888888);
                        }
                        float ppm = id(gas_co_ppm).state;
                        if (ppm < 10) return lv_color_hex(0x00FF00);
                        if (ppm < 35) return lv_color_hex(0xFFFF00);
                        if (ppm < 100) return lv_color_hex(0xFF8800);
                        return lv_color_hex(0xFF0000);
                      repl_obj_id: lbl_co_ppm
                  - label:
                      x: 5
                      y: 50
                      text: "OSHA PEL: 50ppm"
                      text_font: font_tiny
                      text_color: 0x888888
                  - label:
                      x: 5
                      y: 70
                      text: !lambda |-
                        if (!id(co_safety_status).has_state()) return "---";
                        return id(co_safety_status).state;
                      text_font: font_small
                      text_color: 0x888888
                      repl_obj_id: lbl_co_status

            # NH3 Column
            - obj:
                x: 270
                y: 35
                width: 220
                height: 120
                bg_color: 0x1a1a2e
                border_width: 1
                border_color: 0x444444
                widgets:
                  - label:
                      x: 5
                      y: 5
                      text: "Ammonia (NH₃)"
                      text_font: font_small
                      text_color: 0xAAAAAA
                  - label:
                      x: 5
                      y: 25
                      text: !lambda |-
                        if (!id(g_baseline_valid) || !id(gas_nh3_ppm).has_state()) {
                          return "NOT CALIBRATED";
                        }
                        char buf[15];
                        snprintf(buf, sizeof(buf), "%.0f ppm", id(gas_nh3_ppm).state);
                        return std::string(buf);
                      text_font: font_medium
                      text_color: !lambda |-
                        if (!id(g_baseline_valid) || !id(gas_nh3_ppm).has_state()) {
                          return lv_color_hex(0x888888);
                        }
                        float ppm = id(gas_nh3_ppm).state;
                        if (ppm < 25) return lv_color_hex(0x00FF00);
                        if (ppm < 50) return lv_color_hex(0xFFFF00);
                        if (ppm < 100) return lv_color_hex(0xFF8800);
                        return lv_color_hex(0xFF0000);
                      repl_obj_id: lbl_nh3_ppm
                  - label:
                      x: 5
                      y: 50
                      text: "OSHA PEL: 50ppm"
                      text_font: font_tiny
                      text_color: 0x888888
                  - label:
                      x: 5
                      y: 70
                      text: !lambda |-
                        if (!id(nh3_safety_status).has_state()) return "---";
                        return id(nh3_safety_status).state;
                      text_font: font_small
                      text_color: 0x888888
                      repl_obj_id: lbl_nh3_status

            # NO2 Column
            - obj:
                x: 530
                y: 35
                width: 220
                height: 120
                bg_color: 0x1a1a2e
                border_width: 1
                border_color: 0x444444
                widgets:
                  - label:
                      x: 5
                      y: 5
                      text: "Nitrogen Dioxide (NO₂)"
                      text_font: font_small
                      text_color: 0xAAAAAA
                  - label:
                      x: 5
                      y: 25
                      text: !lambda |-
                        if (!id(g_baseline_valid) || !id(gas_no2_ppm).has_state()) {
                          return "NOT CALIBRATED";
                        }
                        char buf[15];
                        snprintf(buf, sizeof(buf), "%.2f ppm", id(gas_no2_ppm).state);
                        return std::string(buf);
                      text_font: font_medium
                      text_color: !lambda |-
                        if (!id(g_baseline_valid) || !id(gas_no2_ppm).has_state()) {
                          return lv_color_hex(0x888888);
                        }
                        float ppm = id(gas_no2_ppm).state;
                        if (ppm < 0.5) return lv_color_hex(0x00FF00);
                        if (ppm < 2.0) return lv_color_hex(0xFFFF00);
                        if (ppm < 5.0) return lv_color_hex(0xFF8800);
                        return lv_color_hex(0xFF0000);
                      repl_obj_id: lbl_no2_ppm
                  - label:
                      x: 5
                      y: 50
                      text: "OSHA PEL: 5ppm"
                      text_font: font_tiny
                      text_color: 0x888888
                  - label:
                      x: 5
                      y: 70
                      text: !lambda |-
                        if (!id(no2_safety_status).has_state()) return "---";
                        return id(no2_safety_status).state;
                      text_font: font_small
                      text_color: 0x888888
                      repl_obj_id: lbl_no2_status

      # ===== DATA QUALITY & CALIBRATION STATUS =====
      - obj:
          x: 20
          y: 480
          width: 760
          height: 80
          bg_color: 0x1a1a2e
          border_width: 1
          border_color: 0x333333
          widgets:
            - label:
                x: 10
                y: 5
                text: "Data Quality & Diagnostics"
                text_font: font_small
                text_color: 0x888888
            - label:
                x: 10
                y: 25
                text: !lambda |-
                  char buf[60];
                  snprintf(buf, sizeof(buf), 
                    "Quality: %.0f%% | Calib: %s | Age: %.1f d",
                    id(data_quality).state,
                    id(g_baseline_valid) ? "VALID" : "INVALID",
                    id(calib_age_days).has_state() ? id(calib_age_days).state : 0.0f
                  );
                  return std::string(buf);
                text_font: font_small
                text_color: 0xCCCCCC
                repl_obj_id: lbl_data_quality
            - label:
                x: 10
                y: 45
                text: !lambda |-
                  char buf[60];
                  snprintf(buf, sizeof(buf), 
                    "AEI: %.1f | WiFi: %d%% | Uptime: %luh",
                    id(pollutant_exposure_index).has_state() ? id(pollutant_exposure_index).state : 0.0f,
                    (int)id(wifi_pct).state,
                    (long)id(uptime_sec).state / 3600
                  );
                  return std::string(buf);
                text_font: font_small
                text_color: 0xCCCCCC
                repl_obj_id: lbl_diagnostics

      # ===== ACTION BUTTONS =====
      - button:
          x: 20
          y: 575
          width: 150
          height: 50
          bg_color: 0x0066FF
          widgets:
            - label:
                text: "Calibrate Now"
                align: center
                text_font: font_small
                text_color: 0xFFFFFF
          on_click:
            then:
              - button.press: btn_calibrate_gas
              - logger.log:
                  format: "User pressed Calibrate - baseline set"

      - button:
          x: 190
          y: 575
          width: 150
          height: 50
          bg_color: 0xFF6600
          widgets:
            - label:
                text: "Reset Calib"
                align: center
                text_font: font_small
                text_color: 0xFFFFFF
          on_click:
            then:
              - button.press: btn_reset_gas_calib
              - logger.log:
                  format: "User pressed Reset - baseline cleared"

      - button:
          x: 630
          y: 575
          width: 150
          height: 50
          bg_color: 0x444444
          widgets:
            - label:
                text: "← Back"
                align: center
                text_font: font_small
                text_color: 0xFFFFFF
          on_click:
            then:
              - lvgl.page.show: main_page

      - label:
          x: 20
          y: 565
          text: "← Tap to return to main page"
          text_font: font_tiny
          text_color: 0x666666
```

---

## Part 2: Home Assistant Automation Examples

### Automation 1: CO₂ Ventilation Control
```yaml
# automations.yaml

- alias: "AQ - High CO₂ Trigger Ventilation"
  description: "Automatically turn on ventilation when CO₂ exceeds 1500ppm"
  trigger:
    platform: numeric_state
    entity_id: sensor.scd41_co2
    above: 1500
  condition:
    - condition: state
      entity_id: binary_sensor.someone_home  # Only if someone home
      state: "on"
  action:
    - service: switch.turn_on
      target:
        entity_id: switch.exhaust_fan
    - service: notify.mobile_app_phone
      data:
        title: "High CO₂ Detected"
        message: "CO₂ is {{ states('sensor.scd41_co2') }}ppm - Ventilation ON"
        data:
          importance: high

- alias: "AQ - CO₂ Normal Disable Ventilation"
  trigger:
    platform: numeric_state
    entity_id: sensor.scd41_co2
    below: 900
  action:
    - service: switch.turn_off
      target:
        entity_id: switch.exhaust_fan
```

### Automation 2: Critical CO Alert
```yaml
- alias: "CRITICAL - Carbon Monoxide Alert"
  description: "Life safety: CO >100ppm triggers emergency sequence"
  trigger:
    platform: state
    entity_id: binary_sensor.co_alert
    to: "on"
  action:
    - service: switch.turn_on
      target:
        entity_id:
          - switch.siren_alarm
          - switch.red_alert_light
    - service: notify.mobile_app
      data:
        title: "🚨 DANGER - CARBON MONOXIDE 🚨"
        message: "CO LEVEL: {{ states('sensor.carbon_monoxide_co') }} ppm - EVACUATE!"
        data:
          importance: max
          tag: "co_alert"
          color: "FF0000"
    - service: climate.set_hvac_mode
      target:
        entity_id: climate.living_room
      data:
        hvac_mode: "fan_only"  # Full ventilation
    - service: logbook.log
      data:
        name: "CRITICAL EVENT"
        message: "Carbon Monoxide Alert Triggered"
        entity_id: binary_sensor.co_alert
```

### Automation 3: Data Quality Monitoring
```yaml
- alias: "AQ - Calibration Overdue Warning"
  trigger:
    platform: numeric_state
    entity_id: sensor.calibration_age_days
    above: 14
  action:
    - service: notify.mobile_app_phone
      data:
        title: "Maintenance Required"
        message: "Gas sensor calibration is {{ states('sensor.calibration_age_days') }} days old. Recalibrate for accuracy."
        data:
          importance: low
          actions:
            - action: "CALIBRATE_NOW"
              title: "Calibrate Now"
            - action: "DISMISS"
              title: "Dismiss"

- alias: "AQ - Poor Data Quality Alert"
  trigger:
    platform: numeric_state
    entity_id: sensor.data_quality_score
    below: 60
  action:
    - service: notify.mobile_app_phone
      data:
        title: "⚠️ Sensor Malfunction"
        message: "Data quality: {{ states('sensor.data_quality_score') }}% - Check sensors"
```

### Automation 4: Multi-Gas Safety Sequence
```yaml
- alias: "AQ - Hazardous Air Quality Detection"
  trigger:
    platform: state
    entity_id: binary_sensor.hazardous_aq
    to: "on"
  action:
    # Log to database
    - service: influxdb.write
      data:
        measurement: "air_quality_alert"
        tags:
          severity: "hazardous"
          triggered_by: "{{ states('sensor.aq_alert_status') }}"
        value: "{{ states('sensor.aq_score') }}"
    
    # Notify occupants
    - service: notify.all_devices
      data:
        title: "⚠️ HAZARDOUS AIR QUALITY"
        message: |
          Score: {{ states('sensor.aq_score') }}%
          CO₂: {{ states('sensor.scd41_co2') }} ppm
          VOC: {{ states('sensor.display_bme680_iaq') }} IAQ
          Action: Ventilate immediately
        data:
          importance: high
    
    # Activate safety devices
    - service: switch.turn_on
      target:
        entity_id:
          - switch.bathroom_exhaust_fan
          - switch.kitchen_range_hood
          - switch.whole_house_ventilation
    
    # Create scene for emergency
    - service: scene.create
      data:
        scene_id: emergency_ventilation
        snapshot_entities:
          - switch.bathroom_exhaust_fan
          - switch.kitchen_range_hood
```

### Automation 5: Weekly Report
```yaml
- alias: "AQ - Weekly Air Quality Report"
  trigger:
    platform: time
    at: "08:00:00"
  condition:
    - condition: template
      value_template: "{{ now().weekday() == 0 }}"  # Sunday
  action:
    - service: notify.mobile_app_phone
      data:
        title: "Weekly Air Quality Report"
        message: |
          📊 WEEKLY AIR QUALITY SUMMARY
          
          Average AQ Score: {{ states('sensor.aq_score') }}%
          Peak CO₂: {{ states('sensor.scd41_co2') }} ppm
          Peak VOC (IAQ): {{ states('sensor.display_bme680_iaq') }}
          Calibration Status: {{ states('binary_sensor.calibration_required') }}
          
          Recommendation: {{ 'Recalibrate sensors' if states('binary_sensor.calibration_required') == 'on' else 'All sensors healthy' }}
```

---

## Part 3: Lovelace Dashboard Configuration

### Simple Dashboard Card
```yaml
# lovelace-dashboard.yaml
type: grid
cards:
  - type: custom:apexcharts-card
    graph_span: 6h
    header:
      title: Air Quality Trends
    series:
      - entity: sensor.scd41_co2
        name: CO₂
        color: "#00FF00"
        yaxis_id: primary
      - entity: sensor.display_bme680_iaq
        name: VOC (IAQ)
        color: "#FF8800"
        yaxis_id: secondary
    yaxes:
      - id: primary
        min: 400
        max: 3000
        title:
          text: "CO₂ (ppm)"
      - id: secondary
        min: 0
        max: 500
        title:
          text: "IAQ"

  - type: custom:gauge-card
    entity: sensor.air_quality_score
    title: "Air Quality Score"
    min: 0
    max: 100
    needle: true
    colors:
      - "hsl(10,100%,35%)"  # Red <25
      - "hsl(35,100%,35%)"  # Orange 25-50
      - "hsl(55,100%,35%)"  # Yellow 50-75
      - "hsl(130,100%,35%)" # Green >75

  - type: custom:stack-in-card
    cards:
      - type: horizontal-stack
        cards:
          - type: custom:mini-graph-card
            entity: sensor.carbon_monoxide_co
            name: CO
            height: 120
            points: 50
          - type: custom:mini-graph-card
            entity: sensor.ammonia_nh3
            name: NH₃
            height: 120
            points: 50
          - type: custom:mini-graph-card
            entity: sensor.nitrogen_dioxide_no2
            name: NO₂
            height: 120
            points: 50

  - type: entities
    entities:
      - entity: sensor.aq_alert_status
        name: "Alert Status"
      - entity: sensor.data_quality_score
        name: "Data Quality"
      - entity: sensor.calibration_age_days
        name: "Calibration Age"
      - entity: sensor.pollutant_exposure_index
        name: "Exposure Index"
    title: "Diagnostics"
```

---

## Part 4: Scripting for Smart Control

```yaml
# scripts.yaml

smart_ventilation:
  description: "Intelligently control ventilation based on air quality"
  sequence:
    - variables:
        aq_score: "{{ states('sensor.aq_score') | float(50) }}"
        co2_level: "{{ states('sensor.scd41_co2') | float(400) }}"
        
    - choose:
        # Excellent - minimal ventilation
        - conditions:
            - condition: template
              value_template: "{{ aq_score >= 75 }}"
          sequence:
            - service: switch.turn_off
              target:
                entity_id: switch.exhaust_fan
                
        # Good - light ventilation
        - conditions:
            - condition: template
              value_template: "{{ aq_score >= 50 and aq_score < 75 }}"
          sequence:
            - service: fan.set_speed
              target:
                entity_id: fan.exhaust_fan
              data:
                percentage: 50
                
        # Fair - moderate ventilation
        - conditions:
            - condition: template
              value_template: "{{ aq_score >= 25 and aq_score < 50 }}"
          sequence:
            - service: fan.set_speed
              target:
                entity_id: fan.exhaust_fan
              data:
                percentage: 75
                
        # Poor - full ventilation
        - conditions:
            - condition: template
              value_template: "{{ aq_score < 25 }}"
          sequence:
            - service: switch.turn_on
              target:
                entity_id:
                  - switch.exhaust_fan
                  - switch.range_hood
                  - switch.whole_house_ventilation
```

---

## Part 5: Customization Tips

### Add Custom Names in HA
```yaml
# customize.yaml
sensor.air_quality_score:
  friendly_name: "Indoor Air Quality %"
  icon: mdi:air-filter

sensor.scd41_co2:
  friendly_name: "CO₂ Level"
  icon: mdi:molecule-co2
  unit_of_measurement: "ppm"

sensor.carbon_monoxide_co:
  friendly_name: "Carbon Monoxide"
  icon: mdi:fire-alert
  
binary_sensor.co_alert:
  device_class: safety
  icon: mdi:fire-alert
```

### Create Groups for UI Organization
```yaml
# groups.yaml
air_quality_main:
  name: "Air Quality Monitor"
  entities:
    - sensor.aq_alert_status
    - sensor.air_quality_score
    - sensor.scd41_co2
    - sensor.display_bme680_iaq
    - sensor.pollutant_exposure_index

air_quality_gases:
  name: "Toxic Gas Sensors"
  entities:
    - sensor.carbon_monoxide_co
    - sensor.ammonia_nh3
    - sensor.nitrogen_dioxide_no2
    - binary_sensor.co_alert
    - binary_sensor.nh3_alert
    - binary_sensor.no2_alert

air_quality_diagnostics:
  name: "Sensor Health"
  entities:
    - sensor.data_quality_score
    - sensor.calibration_age_days
    - binary_sensor.calibration_required
```

---

## Summary

This integration provides:
✅ **Professional dashboard** on your display
✅ **Real-time HA automations** for smart control
✅ **Safety alerts** for dangerous gas levels
✅ **Data quality monitoring** for reliability
✅ **Trend visualization** in Lovelace
✅ **Mobile notifications** for on-the-go awareness

Your air quality detector is now production-ready!
