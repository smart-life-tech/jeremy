# Version 2 vs Version 1 Comparison & Migration Guide

## Side-by-Side Hardware Comparison

```
┌──────────────────────────────────────────────────────────────────┐
│                    FEATURE COMPARISON                             │
├──────────────────────────────────────────┬──────────────┬─────────┤
│ Feature                                  │ V1 (4.3")    │ V2 (5") │
├──────────────────────────────────────────┼──────────────┼─────────┤
│ Microcontroller                          │ ESP32-S3     │ ESP32-S3│
│ Clock Speed                              │ 240MHz       │ 240MHz  │
│ RAM / Flash                              │ 8MB / 8MB    │ 8MB/16MB│
├──────────────────────────────────────────┼──────────────┼─────────┤
│ Display Size                             │ 4.3" 800x480 │ 5" 800x │
│ Display Type                             │ RGB Parallel │ ST7262  │
│ Touch Controller                         │ GT911        │ GT911   │
│ Brightness                               │ ~300 cd/m²   │ 320 cd/m│
├──────────────────────────────────────────┼──────────────┼─────────┤
│ Backlight Control                        │ CH422G (I2C) │ GPIO    │
│ SD Card Slot                             │ NO           │ YES ✓   │
│ Speaker Interface                        │ NO           │ YES ✓   │
│ Battery Interface                        │ Custom       │ Built-in│
├──────────────────────────────────────────┼──────────────┼─────────┤
│ I2C Bus                                  │ 1            │ 1       │
│ UART Available                           │ 1-2          │ 2-3     │
│ GPIO Pins Exposed                        │ Limited      │ More    │
├──────────────────────────────────────────┼──────────────┼─────────┤
│ SENSORS - CO2                            │ SCD41 ✓      │ SCD41 ✓ │
│ SENSORS - Gas (CO/NH3/NO2)               │ MiCS-6814 ✓  │ ZE07-CO │
│ SENSORS - VOC/IAQ                        │ BME680 ✓     │ BME680 │
│ SENSORS - Particulates (NEW)             │ NO ✗         │ SPS30 ✓ │
├──────────────────────────────────────────┼──────────────┼─────────┤
│ Data Logging                             │ HA only      │ SD card │
│ Offline Operation                        │ Yes          │ Yes + SD│
│ Professional Enclosure                   │ NO           │ Planned │
├──────────────────────────────────────────┼──────────────┼─────────┤
│ Power Consumption (idle)                 │ ~200mA       │ ~250mA  │
│ Power Consumption (active)               │ ~1.2A        │ ~1.4A   │
│ Battery Runtime (active)                 │ 8 hours      │ 7 hours │
│ Battery Runtime (screen saver)           │ 18 hours     │ 16 hours│
├──────────────────────────────────────────┼──────────────┼─────────┤
│ Cost (Display Module)                    │ $45-50       │ $42     │
│ Cost (Sensors V1)                        │ ~$150        │ -       │
│ Cost (Sensors V2 - additional)           │ -            │ +$30-40 │
│ Total BOM Cost                           │ ~$195-200    │ ~$220   │
└──────────────────────────────────────────┴──────────────┴─────────┘
```

---

## New Sensors in V2

### SPS30 - Particulate Matter Sensor
```
Technology:  Laser scattering optical particle counter
Interface:   I2C (very easy to integrate)
Measures:    PM1.0, PM2.5, PM4.0, PM10 (µg/m³)

Output Resolution: 
  - PM1.0:  1 µg/m³
  - PM2.5:  1 µg/m³
  - PM4.0:  1 µg/m³
  - PM10:   1 µg/m³

Accuracy:
  - PM2.5:  ±10% @ 100µg/m³ (typical)
  - PM10:   ±10% @ 100µg/m³

Key Features:
✓ No calibration needed (factory calibrated)
✓ Automatic fan cleaning (removes particle buildup)
✓ Very stable baseline (durable sensor)
✓ WHO air quality guidelines alignment
✓ Ideal for indoor/outdoor monitoring

Health Standards:
  - WHO 2021 guideline: PM2.5 <15 µg/m³ (24-hour mean)
  - EPA standard: PM2.5 <35 µg/m³ (24-hour mean)
  - Hazardous: >300 µg/m³
```

### ZE07-CO - Dedicated CO Sensor (vs MiCS)
```
Comparison with MiCS-6814:

FEATURE            │ ZE07-CO      │ MiCS-6814
───────────────────┼──────────────┼──────────
Sensor Type        │ Amperometric │ Metal Oxide
Range              │ 0-1000 ppm   │ 0-1000 ppm
Response Time      │ <60 sec      │ <120 sec
Accuracy           │ ±5% or ±10ppm│ ±10%
Calibration Needed │ NO ✓         │ YES ✗
Zero Drift         │ <1% per year │ Variable
Cross-sensitivity │ Minimal      │ High
Operating Temp     │ -10 to +60°C │ -30 to +70°C
Cost               │ $30-40       │ $20-30

Advantages of ZE07:
✓ No calibration required
✓ Truer amperometric measurement
✓ Industrial-grade stability
✓ Much better long-term reliability
✓ Minimal cross-sensitivity with other gases

ZE07 CO Thresholds:
  - Normal: <5 ppm (background)
  - Elevated: 5-35 ppm (ventilate)
  - Warning: 35-100 ppm (strong ventilation)
  - Alert: 100-500 ppm (evacuate)
  - Critical: >500 ppm (emergency 911)
```

---

## Air Quality Scoring: V2 Enhancement

### V1 Scoring (3 sensors)
```
CO₂:  25% weight
VOC:  25% weight
Gases: 50% weight (MiCS - requires calibration)
─────────────────
Result: Incomplete without gas calibration
```

### V2 Scoring (4 sensors, more complete)
```
CO₂:        25% weight (SCD41)
Particles:  25% weight (SPS30 PM2.5)
CO:         25% weight (ZE07-CO, no calibration!)
VOC/IAQ:    25% weight (BME680)
─────────────────
Result: Complete assessment, always valid!

Key Improvement: V2 doesn't require gas calibration
```

---

## Why V2 is Better for Market

### 1. **PM2.5 Detection (Major Selling Point)**
- **Market demand**: High - post-COVID air quality concern
- **Competitive advantage**: Most "air quality" monitors don't measure particles
- **Health impact**: PM2.5 is #1 air quality concern
- **Use case**: Wildfire smoke, urban pollution, indoor dust

### 2. **ZE07-CO Over MiCS**
- **No calibration needed** = Simpler user experience
- **Industrial-grade accuracy** = Professional positioning
- **Dedicated CO sensor** = Legal safety implications
- **Better stability** = Long-term reliability for commercial use

### 3. **SD Card Logging**
- **Data persistence**: Logs survive WiFi outages
- **Compliance**: Audit trail for professional use
- **Trends**: Users can analyze their own data
- **Offline operation**: Works without Home Assistant

### 4. **More Professional Form Factor**
- **5" display** vs 4.3" - better readability
- **Better GPIO/UART** - room for future expansion
- **Battery interface** - easier charging
- **Speaker option** - for audible alerts

---

## Migration Path: V1 → V2

### For V1 Users
```
Option 1: Keep V1 (already working)
  ✓ No cost, no effort
  ✓ Still monitoring air quality
  ✓ Continue with HA automations
  
Option 2: Upgrade to V2 (new hardware)
  ✓ Add PM2.5 monitoring
  ✓ Better CO measurement (ZE07)
  ✓ SD card data logging
  ✓ Larger display
  
Option 3: Keep Both (Dual Model Strategy)
  ✓ V1 for bedroom/living room
  ✓ V2 for kitchen/entry (particle monitoring)
  ✓ Complementary coverage
  ✓ Better market positioning
```

### Code Compatibility
- **V1 code**: `air_quality_pro.yaml` (MiCS-based, still works)
- **V2 code**: `air_quality_pro_v2.yaml` (SPS30 + ZE07-based)
- **Both in HA**: Can run simultaneously
- **Different entity IDs**: No conflicts
- **Shared HA automations**: Can reference both

---

## Dual Model Market Strategy

### Tier 1: "Basic" - V1
**Target**: Home automation enthusiasts, budget-conscious
```
Hardware:
  - 4.3" Waveshare display
  - SCD41 (CO₂)
  - BME680 (VOC/IAQ)
  - MiCS-6814 (gases)

Price: $200-250
Features: CO₂, VOC, gas detection, HA integration
Positioning: "Affordable air quality monitoring"
```

### Tier 2: "Professional" - V2
**Target**: Home safety buyers, commercial users
```
Hardware:
  - 5" UEAIDISP display
  - SCD41 (CO₂)
  - BME680 (VOC/IAQ)
  - ZE07-CO (dedicated CO)
  - SPS30 (particulate matter)

Price: $350-400
Features: Complete AQ monitoring + particles + logging
Positioning: "Professional-grade air quality detector"
```

### Marketing Positioning
```
V1: "Know your indoor air quality"
     └─ Good for home automation users

V2: "Know exactly what's in your air"
     └─ Better for families, professionals, commercial
     └─ Includes harmful particles
     └─ Professional CO measurement
     └─ Data logging for analysis
```

---

## Wiring for V2

### I2C Bus (Main)
```
GPIO8  (SDA) ─── SCD41, BME680, SPS30
GPIO9  (SCL) ─── SCD41, BME680, SPS30
GND           ─── All sensors

Pullups: 4.7kΩ (should be on-module)
Speed: 100kHz
```

### UART for ZE07-CO
```
GPIO17 (TX)  ─── Not used (receiver only)
GPIO18 (RX)  ─── ZE07-CO TX pin
GND          ─── ZE07-CO GND

Baud Rate: 9600
Protocol: Standard UART
```

### Display Connections
```
Display handles RGB parallel interface
Touch: GT911 on I2C (may conflict if not isolated)
Backlight: GPIO-based (better than V1's CH422G)
```

---

## Power Considerations

### Power Budget Analysis
```
Component          │ Current    │ Notes
─────────────────┼────────────┼──────────────────
ESP32-S3 (active) │ 160mA      │ 240MHz core
SCD41             │ 20mA       │ Measurement
BME680            │ 10mA       │ I2C polling
SPS30             │ 40mA       │ Fan (periodic)
ZE07-CO           │ 30mA       │ UART continuous
Display RGB       │ 800mA      │ LED backlighting
Display Ctrl      │ 150mA      │ ST7262 driver
─────────────────┼────────────┼──────────────────
TOTAL ACTIVE      │ ~1.4A      │ All running
────────────────────────────────────────────────

Display OFF       │ ~250mA     │ Sensors + WiFi idle
Screen Saver      │ ~500mA     │ Low brightness display

Battery Runtime (10000mAh LiPo):
  - Continuous active: ~7 hours
  - With screen saver: ~16-18 hours
  - WiFi OFF: 40+ hours

Optimization:
  ✓ SPS30 update interval: 30s (reduces current spikes)
  ✓ Screen saver after 5-10 min (major power saving)
  ✓ WiFi sleep when idle (if configured)
```

---

## SD Card Logging (New V2 Feature)

### Benefits
```
1. Data Persistence
   - Logs survive WiFi outages
   - Works offline entirely
   - Backup of HA data

2. Compliance
   - Audit trail for commercial use
   - CSV export for analysis
   - Timestamps on all readings

3. Advanced Analysis
   - Trend analysis over days/weeks
   - Peak detection algorithms
   - Spike correlation (what caused it?)
   
4. No Cloud Required
   - All data local
   - Privacy-preserving
   - Works in isolated environments
```

### Data Format
```csv
timestamp,co2_ppm,temp_f,humidity_%,voc_iaq,co_ppm,pm1_0,pm2_5,pm4_0,pm10,aq_score_%
2026-01-16T10:30:15Z,420,72.5,45,85,2.3,5,12,18,25,92
2026-01-16T10:31:15Z,425,72.4,45,86,2.4,5,12,18,26,91
...
```

### Implementation Options
1. **Simple**: ESPHome's built-in SD card logging
2. **Advanced**: HA system with InfluxDB + Grafana
3. **Hybrid**: Local SD card + cloud backup optional

---

## Testing Checklist for V2

### Before Deployment
- [ ] All 4 sensors appear in Home Assistant
- [ ] I2C bus scans correctly (SCD41, BME680, SPS30)
- [ ] ZE07-CO UART receiving data
- [ ] Display shows all sensor values
- [ ] AQ score calculating correctly
- [ ] Alerts triggering at expected thresholds
- [ ] SD card logging working
- [ ] Battery voltage accurate
- [ ] WiFi connecting reliably
- [ ] HA automations compatible with V2 entities

### Validation Tests
```
Test 1: PM2.5 Measurement
  Action: Run near candle/incense
  Expected: PM2.5 spikes, triggers alert
  Verify: Data logged to SD

Test 2: CO Detection
  Action: Run exhaust fan briefly (external CO source)
  Expected: ZE07-CO detects change
  Verify: AQ score drops, alert may trigger

Test 3: CO2 Measurement
  Action: Hold breath near sensor (breath CO2)
  Expected: SCD41 slowly increases
  Verify: Trend visible, no false spikes

Test 4: Power Drain
  Action: Run 8+ hours on battery
  Expected: See actual runtime
  Verify: Compare to power budget analysis
```

---

## Firmware Considerations

### ZE07-CO UART Configuration
```yaml
uart:
  - id: uart_ze07_co
    tx_pin: GPIO17      # Not used
    rx_pin: GPIO18      # Data reception
    baud_rate: 9600    # Standard
    stop_bits: 1
    parity: NONE
    debug: 
      direction: BOTH
      dummy_receiver_enabled: false
```

### Modbus Support
```
ZE07-CO can use either:
1. Plain UART with AT commands
2. Modbus protocol (more structured)

ESPHome has modbus_controller for this.
Need to determine which version your ZE07 is.
```

---

## Comparison to Market

```
PRODUCT          │ Price │ CO₂ │ CO  │ PM2.5│ HA    │ Battery
─────────────────┼───────┼─────┼─────┼──────┼───────┼─────────
Wyze Air Monitor │ $30   │ ✓   │ ✗   │ ✗    │ ✗     │ ✗
Nest Hub Max     │ $230  │ ✓   │ ✗   │ ✗    │ ✗ Prop│ ✗
Eve Room        │ $80   │ ✗   │ ✗   │ ✗    │ ✓     │ ✗
V1 "Basic"      │ $220  │ ✓   │ ✓   │ ✗    │ ✓     │ ✓
V2 "Professional"│ $380  │ ✓   │ ✓   │ ✓    │ ✓     │ ✓
─────────────────┴───────┴─────┴─────┴──────┴───────┴─────────
```

**Your V2 advantage:**
- Only device with CO + CO₂ + PM2.5 + VOC
- Dedicated CO sensor (not estimated)
- HA integration (others are cloud-locked)
- Battery powered (others are wall-mounted)
- Data logging (others don't have this)

---

## Next Steps

1. **Get ZE07-CO details**: Confirm UART protocol (AT vs Modbus)
2. **Test SPS30 integration**: Verify I2C address and driver
3. **Firmware update**: Use air_quality_pro_v2.yaml
4. **HA migration**: Update automation references (V1 vs V2 entities)
5. **Document processes**: Create customer setup guides

**Recommendation:** Build V2 as primary, keep V1 for budget tier.
