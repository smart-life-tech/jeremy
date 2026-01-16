# Version 2 Implementation & Deployment Guide

## Hardware Assembly Checklist

### Display Module (UEAIDISP 5")
```
✓ Unpack display carefully (ESD sensitive)
✓ Identify all connectors:
  - Power (USB-C or battery)
  - TF card slot
  - Speaker connector
  - GPIO/UART headers
  
✓ Test display with factory firmware first
  - Verify touchscreen works
  - Check brightness
  - Confirm WiFi connection

✓ Locate GPIO pins for your sensors:
  - GPIO8, GPIO9 (I2C for sensors)
  - GPIO17, GPIO18 (UART for ZE07)
  - GND, 5V, 3.3V
```

### I2C Sensor Assembly
```
Parts needed:
  ✓ SCD41 (CO₂) module
  ✓ BME680 (VOC) module
  ✓ SPS30 (Particulates) module
  ✓ Jumper wires (22 AWG recommended)
  ✓ 2× 4.7kΩ resistors (pullups, often included)
  ✓ Breadboard or perfboard for connections

Wiring (all in parallel on I2C bus):
```
┌─── 5V (regulated)
│
├── SCD41 VDD
├── BME680 VCC
└── SPS30 VDD

┌─── GPIO8 (SDA)
├── 4.7kΩ pullup to 3.3V or 5V
├── SCD41 SDA
├── BME680 SDA
└── SPS30 SDA

┌─── GPIO9 (SCL)
├── 4.7kΩ pullup to 3.3V or 5V
├── SCD41 SCL
├── BME680 SCL
└── SPS30 SCL

┌─── GND
├── SCD41 GND
├── BME680 GND
└── SPS30 GND
```

### ZE07-CO UART Connection
```
5V Regulated Supply (separate from I2C sensors!)
  │
  └── ZE07-CO VCC
      ZE07-CO GND ─── ESP32 GND (common)

UART Connection:
  GPIO18 (RX) ──── ZE07-CO TX (output from sensor)
  GPIO17 (TX) ──── ZE07-CO RX (optional, usually unused)

Power Supply Important:
  ✗ Do NOT use 3.3V GPIO voltage
  ✓ Use dedicated 5V regulated supply
  ✓ Separate from I2C sensors if possible
  ✓ Use separate GND if possible (same star point)
```

### Power Supply Architecture
```
Recommended Setup:
┌─ USB-C Power (5V/2A minimum)
│  ├─ Display power (1.5A typical)
│  ├─ I2C sensors power (100-150mA)
│  └─ ZE07-CO power (30mA)
│
└─ OR Battery Management
   ├─ 10000mAh LiPo (3.7V)
   ├─ Charge circuit (USB-C, built-in on board)
   └─ 5V boost converter for ZE07/SPS30

For V2, recommend LiFePO4 charging (newer safer chemistry).
```

---

## Firmware Installation

### Step 1: Install ESPHome Add-on (if needed)
```bash
# In Home Assistant
Settings → Add-ons → Create Add-on → ESPHome

# Or use command line
pip install esphome
```

### Step 2: Prepare Configuration
```bash
# Copy air_quality_pro_v2.yaml to your esphome directory
cp air_quality_pro_v2.yaml /config/esphome/

# Create device config
cd /config/esphome
```

### Step 3: Create/Edit Device YAML

**Create `v2_device.yaml`:**
```yaml
esphome:
  name: air-quality-detector-v2
  friendly_name: "Air Quality Monitor V2"

esp32:
  board: esp32-s3-devkitc-1
  variant: esp32s3
  flash_size: 16MB
  psram:
    mode: octal
    speed: 80MHz

wifi:
  ssid: "YOUR_SSID"
  password: "YOUR_PASSWORD"
  ap:
    ssid: "AQ-Detector-V2"
    password: "fallback123"

api:
  encryption:
    key: !secret api_encryption_key

ota:
  password: !secret ota_password

time:
  - platform: homeassistant

# Include V2 sensor module
<<: !include air_quality_pro_v2.yaml

# Include display LVGL config (if you have one)
# <<: !include v2_display.yaml
```

### Step 4: Generate Secrets
```bash
# Create secrets file
esphome run v2_device.yaml

# ESPHome will guide you through:
# - WiFi setup
# - API key generation
# - OTA password
```

### Step 5: Compile
```bash
esphome compile v2_device.yaml
# Wait 5-10 minutes (large binary with all sensors)
```

### Step 6: Upload
```bash
esphome upload v2_device.yaml
# Connect device via USB-C
# ESPHome handles flashing automatically
```

### Step 7: Monitor
```bash
esphome logs v2_device.yaml
# Watch initialization:
# - I2C devices found
# - UART data appearing
# - HA connection established
```

---

## First Boot Validation

### Check Logs
```
Expected output:
  [16:34:22][I][scd4x:208]: SCD4x initialized
  [16:34:23][I][bme680:205]: BME680 initialized
  [16:34:24][I][sps30:145]: SPS30 found at address 0x69
  [16:34:25][I][uart:012]: UART1 data available
  [16:34:26][I][wifi:492]: WiFi connected to BOVENS-MESH IP:192.168.1.100
  [16:34:27][I][api:138]: Home Assistant API ready
```

### Verify Home Assistant Integration
```
In HA, these entities should appear automatically:
✓ sensor.air_quality_score_v2 (%)
✓ sensor.scd41_co2 (ppm)
✓ sensor.scd41_temperature (°F)
✓ sensor.scd41_humidity (%)
✓ sensor.ze07_co_carbon_monoxide (ppm)
✓ sensor.sps30_pm1_0 (µg/m³)
✓ sensor.sps30_pm2_5 (µg/m³)
✓ sensor.sps30_pm4_0 (µg/m³)
✓ sensor.sps30_pm10 (µg/m³)
✓ sensor.bme680_temperature (°C)
✓ sensor.bme680_humidity (%)
✓ sensor.bme680_pressure (hPa)
✓ sensor.bme680_iaq_index (IAQ)
✓ sensor.co_safety_status_v2
✓ sensor.pm2_5_safety_status
✓ binary_sensor.co_alert_v2
✓ binary_sensor.pm2_5_alert
✓ binary_sensor.co2_alert_v2
✓ binary_sensor.hazardous_aq_v2
```

### Test Each Sensor
```
1. SCD41 (CO₂)
   Action: Breathe on sensor
   Expected: CO₂ rises from ~400 to 600-1000 ppm
   Time: 30-60 seconds to see change

2. SPS30 (Particulates)
   Action: Light a candle near sensor
   Expected: PM2.5 rises from <10 to 50-100+ µg/m³
   Time: 10-30 seconds to detect

3. ZE07-CO (CO)
   Action: Blow vehicle exhaust near sensor
   Expected: CO rises from <5 to 50-500 ppm
   Time: 5-30 seconds to detect
   ⚠️  Do NOT do indoors - use small external source

4. BME680 (VOC/IAQ)
   Action: No action needed - continuously measures
   Expected: IAQ 50-150 in clean room
   Time: Ongoing monitoring
```

---

## Home Assistant Automation Setup

### Create Automations for V2

**Automation 1: PM2.5 Too High**
```yaml
automation:
  - id: "aqu_pm25_alert"
    alias: "AQ V2 - PM2.5 Alert"
    trigger:
      platform: state
      entity_id: binary_sensor.pm2_5_alert
      to: "on"
    action:
      - service: notify.mobile_app_phone
        data:
          title: "⚠️ High Particulates"
          message: "PM2.5: {{ states('sensor.sps30_pm2_5') }}µg/m³ - Close windows, use air filter"
          data:
            importance: high
      - service: switch.turn_on
        entity_id: switch.bedroom_air_filter  # if you have one
```

**Automation 2: Critical CO Alert**
```yaml
  - id: "aqu_co_critical"
    alias: "CRITICAL - CO Detected"
    trigger:
      platform: state
      entity_id: binary_sensor.co_alert_v2
      to: "on"
    action:
      - service: switch.turn_on
        entity_id:
          - switch.alarm_siren
          - switch.emergency_light
      - service: notify.mobile_app
        data:
          title: "🚨 DANGER - CARBON MONOXIDE 🚨"
          message: "CO: {{ states('sensor.ze07_co_carbon_monoxide') }}ppm - EVACUATE!"
          data:
            importance: max
```

**Automation 3: Good AQ Score**
```yaml
  - id: "aqu_v2_good"
    alias: "AQ V2 - Good Air Quality"
    trigger:
      platform: numeric_state
      entity_id: sensor.air_quality_score_v2
      above: 75
    action:
      - service: light.turn_on
        entity_id: light.status_light
        data:
          color_name: green
      - service: notify.mobile_app_phone
        data:
          message: "Air quality is excellent ({{ states('sensor.air_quality_score_v2') }}%)"
          data:
            importance: low
```

---

## Dashboard Setup

### Lovelace Card Configuration

**V2 Air Quality Card:**
```yaml
type: custom:stack-in-card
cards:
  - type: horizontal-stack
    cards:
      - type: custom:gauge-card
        entity: sensor.air_quality_score_v2
        title: "Overall AQ"
        min: 0
        max: 100
        needle: true
        colors:
          - "hsl(10,100%,35%)"    # Red <25
          - "hsl(35,100%,35%)"    # Orange 25-50
          - "hsl(55,100%,35%)"    # Yellow 50-75
          - "hsl(130,100%,35%)"   # Green >75

  - type: custom:gauge-card
    entity: sensor.sps30_pm2_5
    title: "PM2.5"
    min: 0
    max: 100
    unit_of_measurement: "µg/m³"
    needle: true
    colors:
      - "hsl(10,100%,35%)"        # Red
      - "hsl(35,100%,35%)"        # Orange
      - "hsl(55,100%,35%)"        # Yellow
      - "hsl(130,100%,35%)"       # Green

  - type: custom:gauge-card
    entity: sensor.ze07_co_carbon_monoxide
    title: "CO"
    min: 0
    max: 100
    unit_of_measurement: "ppm"
    needle: true
    colors:
      - "hsl(10,100%,35%)"        # Red
      - "hsl(35,100%,35%)"        # Orange
      - "hsl(55,100%,35%)"        # Yellow
      - "hsl(130,100%,35%)"       # Green

  - type: custom:mini-graph-card
    entity: sensor.scd41_co2
    name: "CO₂"
    hours_to_show: 6
    points_per_hour: 12
    line_color: "#0080FF"
```

---

## Testing & Validation

### Performance Benchmarks

**Expected Readings (clean indoor air):**
```
CO₂:    400-600 ppm (outdoor 400, indoor accumulates)
CO:     0-5 ppm (normal background)
PM2.5:  5-15 µg/m³ (clean room, higher if cooking/traffic)
VOC:    50-150 IAQ (clean air)
Temp:   68-75°F (room temperature)
Humidity: 40-60%
```

**Update Intervals:**
```
CO₂:     5 seconds (SCD41 native)
CO:      5 seconds (ZE07 polling)
PM2.5:   30 seconds (SPS30 takes time)
VOC:     5 seconds (BME680)
AQ Score: 10 seconds (composite calculation)
```

**Power Consumption:**
```
Idle (WiFi connected): ~250-300mA
Active (all sensors): ~1.4-1.6A
Display ON: adds ~800-1000mA
```

### Validation Tests

**Test 1: I2C Bus Integrity**
```
In HA, use esphome.scan_i2c_bus service
Expected: 4 devices found
  - 0x62: SCD41
  - 0x77: BME680
  - 0x69: SPS30
  - (Optional) 0x48: ADS1115 if temperature compensation
```

**Test 2: UART Connectivity**
```
Check logs for UART output
Should see continuous data like:
  [16:34:25][D][uart:012]: RX: ...
  
If nothing appears:
  - Check baud rate (9600)
  - Verify GPIO18 is actually receiving
  - Test with oscilloscope if available
```

**Test 3: Data Quality**
```
Monitor sensor.data_quality_v2
Expected: 100% when all sensors present
Drops indicate missing sensor
```

**Test 4: AQ Score Calculation**
```
Verify:
  - Score updates every 10 seconds
  - Changes when you introduce test conditions
  - Falls in range 0-100%
  - Status changes (EXCELLENT → GOOD → FAIR, etc.)
```

---

## Production Deployment Checklist

### Pre-Deployment
- [ ] All sensors tested individually
- [ ] Firmware compiles without errors/warnings
- [ ] Home Assistant integration verified
- [ ] Automations created and tested
- [ ] Dashboard displays all data
- [ ] Power management configured
- [ ] Temperature compensation working (ZE07)
- [ ] SD card logging working (if enabled)

### Deployment
- [ ] Device sealed in waterproof enclosure
- [ ] Sensor inlets properly ventilated
- [ ] Display protected but touchable
- [ ] Power supply rated for sustained use
- [ ] WiFi signal strength adequate (>-70dBm)
- [ ] Backup power/battery available

### Ongoing Maintenance
- [ ] Check for updates monthly
- [ ] Monitor data quality score
- [ ] Verify all sensors still responsive
- [ ] Clean SPS30 fan intake (dust trap if available)
- [ ] Replace battery if battery-powered
- [ ] Review HA logs for issues

---

## Troubleshooting Quick Reference

| Problem | Check First | Solution |
|---------|-------------|----------|
| No I2C devices | Wiring | Verify GPIO8/9, pullups, power |
| SPS30 not found | Address | Check if 0x69 (default) |
| ZE07 no data | UART | Check GPIO18, baud rate |
| CO always zero | Power | ZE07 needs 5V, not 3.3V |
| PM2.5 always zero | Sensor | Normal if no particles, test with candle |
| WiFi won't connect | Signal | Check SSID/password, WiFi strength |
| HA not finding device | API key | Regenerate encryption key |
| Readings unstable | Filtering | Increase update_interval |

---

## Comparison to V1 in Production

### V1 (Basic) Deployment
```
- 4.3" display (sufficient)
- Gas calibration needed (user task)
- MiCS requires clean-air setup
- Good for enthusiasts
- Works great with HA
```

### V2 (Professional) Deployment
```
- 5" display (better visibility)
- No gas calibration (ZE07 factory-cal)
- Particle measurement (major feature)
- Data logging to SD (compliance)
- Professional enclosure ready
- Better long-term reliability
```

**Recommendation:** 
- **Launch both** as product tiers
- V1 as budget option ($200-250)
- V2 as professional option ($350-400)
- Target different markets simultaneously

---

## Next Steps

1. **Build and test V2 prototype** (2-3 days)
2. **Validate all sensors** (1-2 days)
3. **Create HA automations** (1 day)
4. **Design professional enclosure** (1-2 weeks)
5. **Prepare customer documentation** (1 week)
6. **Launch both V1 & V2** (ready!)

**Timeline to market:** 4-6 weeks for complete V2 launch

---

## Support Resources

- **ESPHome Docs:** https://esphome.io
- **SPS30 Driver:** ESPHome built-in
- **ZE07-CO Protocol:** Check your module documentation
- **HA Automations:** https://www.home-assistant.io/docs/automation/
- **Sensor Datasheets:** Manufacturer documentation

You're ready to build V2! 🚀
