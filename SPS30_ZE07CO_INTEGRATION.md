# V2 Sensor Integration: SPS30 & ZE07-CO Guide

## Quick Start

### SPS30 (I2C Particulate Matter Sensor)
```
Library: sps30 (built-in ESPHome support)
Address: 0x69 (default, check datasheet if different)
Bus: Main I2C (GPIO8/GPIO9)
Update Interval: 30 seconds (recommended)
No calibration needed - factory calibrated
```

### ZE07-CO (UART CO Sensor)
```
Interface: UART at 9600 baud
RX Pin: GPIO18
TX Pin: GPIO17 (usually not used)
Protocol: AT commands or Modbus
Update Interval: 5 seconds
No calibration needed - factory calibrated
```

---

## SPS30 Detailed Integration

### Physical Specifications
```
Dimensions: 41.3mm (L) × 41.3mm (W) × 12.2mm (H)
Weight: ~7g
Power: 4.5-5.5V DC
Current: 
  - Idle: ~3mA
  - Active measurement: ~40mA (with fan)
  - Fan cleaning: Can spike to 80mA briefly

Operating Conditions:
  - Temperature: 0 to 50°C
  - Humidity: 0-95% RH (non-condensing)
  - Altitude: 0-2000m
```

### I2C Wiring (SPS30)
```
      SPS30          ESP32-S3
      ─────          ────────
      VDD    ───────→ 5V or 3.3V
      GND    ───────→ GND
      SDA    ───────→ GPIO8 (via 4.7kΩ pull-up)
      SCL    ───────→ GPIO9 (via 4.7kΩ pull-up)
      
Note: Some SPS30 modules include on-board pullups
Check your specific module
```

### Measurement Principles
```
Technology: Laser scattering optical counter
Wavelength: 650nm red laser
Particle Detection: 0.3 to 40 µm
Output Modes:
  - Mass concentration (µg/m³) - standard
  - Number concentration (particles/cm³) - optional

Accuracy at Different PM Levels:
  PM1.0:  ±10% @ 10-100 µg/m³
  PM2.5:  ±10% @ 10-100 µg/m³
  PM4.0:  ±10% @ 10-100 µg/m³
  PM10:   ±10% @ 10-100 µg/m³
```

### Fan Cleaning
```
SPS30 has automatic self-cleaning mechanism:

When: Every 168 hours (once per week)
Duration: ~10 seconds
What happens:
  - Fan speed increases to ~12,000 RPM
  - Dislodges particle buildup
  - Current draw increases briefly
  
Effect on readings:
  - May show temporary spikes
  - Normal and expected
  - Improves long-term accuracy

Can be triggered manually or adjusted if needed.
```

### Health Thresholds (WHO/EPA)

**PM2.5 (Most Important)**
```
WHO 2021 Air Quality Guideline
  - 24-hour mean: <15 µg/m³ (new guidance)
  - Annual mean: <5 µg/m³ (very strict)

EPA Standard
  - 24-hour mean: <35 µg/m³
  - Annual mean: <12 µg/m³

Our Alert Thresholds
  - Good:        <10 µg/m³     (GREEN)
  - Acceptable:  10-15 µg/m³   (YELLOW)
  - Elevated:    15-25 µg/m³   (ORANGE)
  - Poor:        25-35 µg/m³   (ORANGE)
  - Hazardous:   >35 µg/m³     (RED)
```

**PM10 (Coarser Particles)**
```
EPA 24-hour standard: 150 µg/m³
WHO guideline: 50 µg/m³
Mainly dust and pollen (less health-critical)
Still important for respiratory sensitivity
```

### ESPHome Implementation
```yaml
sensor:
  - platform: sps30
    i2c_id: bus_a
    pm_1_0:
      name: "SPS30 PM1.0"
      id: sps30_pm1_0
    pm_2_5:
      name: "SPS30 PM2.5"
      id: sps30_pm2_5
      device_class: "pm25"        # HA device class
    pm_4_0:
      name: "SPS30 PM4.0"
      id: sps30_pm4_0
    pm_10_0:
      name: "SPS30 PM10"
      id: sps30_pm10
      device_class: "pm10"        # HA device class
    update_interval: 30s           # Don't query too fast
```

### Common Issues & Solutions

**Issue: SPS30 not found on I2C bus**
```
Solution 1: Check wiring
  - Verify GPIO8 (SDA), GPIO9 (SCL)
  - Check power supply (5V or 3.3V?)
  - Verify I2C pullups present

Solution 2: Address mismatch
  - Default address: 0x69
  - Can be changed via I2C
  - Add debug logging to see what's being found

Solution 3: Library issue
  - Ensure ESPHome version >= 2024.1
  - SPS30 support added in recent versions
```

**Issue: PM values always zero**
```
Possible Causes:
  1. Sensor warming up (takes 30-60 seconds)
  2. Sensor in standby mode
  3. Fan not running
  4. No particles in air (could be normal!)

Verification:
  - Check logs for SPS30 initialization
  - Monitor fan current draw spike
  - Test with candle/incense nearby
```

**Issue: Readings jumping erratically**
```
Likely: Fan cleaning cycle running
  - Expected every 168 hours
  - Causes brief current spike
  - Readings stabilize after 10 seconds

Solution:
  - Add median filter if needed
  - Increase update interval to 60s
  - Normal and not a problem
```

---

## ZE07-CO Detailed Integration

### Sensor Specifications
```
Measurement Principle: Electrochemical amperometric
Gas Measured: Carbon Monoxide (CO)
Range: 0-1000 ppm
Resolution: 1 ppm
Accuracy: ±5% of reading OR ±10 ppm (whichever is larger)
Response Time: <60 seconds

Operating Conditions:
  - Temperature: -10 to +60°C
  - Humidity: 10-90% RH (non-condensing)
  - Pressure: 86-106 kPa
  - Power: 4.5-5.5V DC
  - Current: ~30mA continuous

Calibration: Factory calibrated, NO user calibration needed!
Lifetime: ~2-5 years (electrolyte consumption)
```

### Why ZE07-CO Over MiCS-6814

**ZE07-CO (Electrochemical)**
```
Pros:
✓ No calibration needed (huge advantage!)
✓ True amperometric measurement
✓ Minimal baseline drift (<1%/year)
✓ Industrial-grade stability
✓ Immune to cross-sensitivity
✓ Better accuracy (±5% vs ±10%)

Cons:
✗ Slightly more expensive ($30-40)
✗ Shorter lifetime (2-5 years vs 5+ years)
✗ Temperature sensitive (need compensation)
✗ Requires 4.5-5.5V (not 3.3V)
```

**MiCS-6814 (Metal Oxide)**
```
Pros:
✓ Cheaper ($20-30)
✓ Longer lifetime
✓ Works on 3.3V

Cons:
✗ Requires calibration (user hassle)
✗ Baseline drift over time (0.5-5%/month)
✗ Cross-sensitivity to other gases
✗ Lower accuracy (±10% or more)
✗ Environmental dependence
```

### UART Wiring (ZE07-CO)
```
      ZE07-CO        ESP32-S3
      ──────────     ────────
      VCC    ───────→ 5V (requires regulated 5V!)
      GND    ───────→ GND
      TX     ───────→ GPIO18 (RX on S3)
      RX     ───────→ GPIO17 (TX on S3) [optional]
      
Note: ZE07 is OUTPUT-ONLY on TX pin
      GPIO17 not actually needed but can leave connected
```

### Protocol: AT Commands vs Modbus

**AT Command Mode (Simpler)**
```
ZE07 can send unsolicited readings via UART:
  - Typical format: AABBCCDD... (hex encoded)
  - Timestamp: Included in some versions
  - Parsing: Can be complex without documentation

ESPHome usage:
  - Use uart.write to send commands
  - Use logger to capture output
  - Parse with lambda functions
```

**Modbus RTU (More Structured)**
```
Some ZE07 modules support Modbus:
  - Standardized protocol
  - Easier parsing
  - Register-based access
  - Better for integration

ESPHome usage:
  - modbus_controller platform
  - Read from specific registers
  - More reliable
```

### Determining Your Version
```
Check your ZE07-CO documentation:
  - Look for "AT command" or "protocol"
  - Check if it mentions "Modbus"
  - Try sending AT command: "AT+INFO"
  - See if you get structured response

Most common versions use AT commands.
If unsure, test with serial monitor first.
```

### Temperature Compensation

**Why Important:**
```
Electrochemical sensors are temperature-sensitive:
  - 20°C (68°F): Reference temperature
  - <10°C: Readings ~10% low
  - >30°C: Readings ~10% high
  
Solution: Apply temperature compensation
```

**Implementation:**
```yaml
sensor:
  - platform: template
    name: "ZE07-CO Compensated"
    unit_of_measurement: "ppm"
    update_interval: 5s
    lambda: |-
      if (!id(ze07_co_raw).has_state()) return NAN;
      if (!id(scd41_temp_c).has_state()) return NAN;
      
      float raw_co = id(ze07_co_raw).state;
      float temp_c = id(scd41_temp_c).state;
      
      // Temperature compensation factor
      // Roughly: 0.5% per °C deviation from 20°C
      float compensation = 1.0f + (temp_c - 20.0f) * 0.005f;
      
      return raw_co / compensation;
```

### CO OSHA Standards & Thresholds

```
OSHA Standards:
  - PEL (Permissible Exposure Limit): 50 ppm (8-hour TWA)
  - IDLH (Immediately Dangerous to Life/Health): 1200 ppm
  - Action Level: 30 ppm (enhanced monitoring)

Our Thresholds:
  - Background: 0-2 ppm (normal)
  - Elevated: 2-10 ppm (monitor, might be local source)
  - Warning: 10-35 ppm (investigate, maybe defective appliance)
  - Alert: 35-100 ppm (evacuate, ventilate immediately)
  - Critical: >100 ppm (EMERGENCY - call 911)

Health Effects by Level:
  10 ppm:  Safe for 40-hour/week exposure
  35 ppm:  4-hour TWA limit (OSHA action level)
  100 ppm: 30-minute exposure limit
  400 ppm: Headache/nausea in 2-3 hours
  800 ppm: Dizziness in 45 minutes
  1600 ppm: Headache in 2 hours, death in 4 hours
  3200 ppm: Headache/dizziness in 5 minutes, death in 30 min
```

### ESPHome Implementation

**Simple UART Reading:**
```yaml
uart:
  - id: uart_ze07_co
    tx_pin: GPIO17
    rx_pin: GPIO18
    baud_rate: 9600

sensor:
  - platform: modbus_controller
    modbus_controller_id: modbus_ze07
    id: ze07_co_raw
    register_type: holding
    address: 0x0000
    unit_of_measurement: "ppm"
    value_type: U_WORD
    update_interval: 5s
```

**Or Manual UART Parsing (if needed):**
```yaml
sensor:
  - platform: template
    name: "ZE07-CO"
    id: ze07_co_ppm
    update_interval: 5s
    lambda: |-
      // Manual parsing if no Modbus support
      // This depends on your specific ZE07 protocol
      // Consult your module's documentation
      
      if (!id(ze07_co_raw).has_state()) return NAN;
      return id(ze07_co_raw).state;
```

### Diagnostics

**Check Sensor Status:**
```yaml
text_sensor:
  - platform: template
    name: "ZE07-CO Status"
    update_interval: 60s
    lambda: |-
      if (!id(ze07_co_raw).has_state()) {
        return std::string("NO DATA");
      }
      
      float co = id(ze07_co_raw).state;
      if (co == 0) return std::string("WARMING UP");
      if (co > 0 && co < 10) return std::string("NORMAL");
      if (co >= 10 && co < 35) return std::string("ELEVATED");
      if (co >= 35) return std::string("CRITICAL");
      
      return std::string("UNKNOWN");
```

### Troubleshooting ZE07-CO

**Issue: No data from ZE07**
```
Solution 1: Check power supply
  - ZE07 requires 4.5-5.5V (use regulated supply!)
  - Not 3.3V GPIO voltage
  - Check with multimeter

Solution 2: UART configuration
  - Verify GPIO18 is RX (data IN to S3)
  - Verify GPIO17 is TX (optional)
  - Check baud rate: 9600
  - Test with serial monitor first

Solution 3: Hardware connection
  - Try different UART (UART0, UART1, UART2)
  - Check for loose wires
  - Verify cable integrity
```

**Issue: Readings always zero**
```
Possible causes:
  1. Sensor warming up (takes 2-5 minutes)
  2. Sensor not receiving power
  3. Protocol mismatch (AT vs Modbus)
  4. Register address wrong

Diagnostic:
  - Check logs for UART messages
  - Look for any data at all on RX pin
  - Monitor power supply voltage
  - Try AT+GADC command to get raw reading
```

**Issue: Values not updating**
```
Check:
  1. Update interval (set to 5s in code)
  2. UART not blocked by other processes
  3. Sensor needs to stabilize after power-on
  4. Try manual command: AT+GADC=? to test

Solution:
  - Increase update_interval to 10s if too frequent
  - Check for UART conflicts with other sensors
  - Wait 5 minutes after power-on before expecting data
```

---

## Integration Checklist for V2

### Before First Boot
- [ ] I2C pullups installed (4.7kΩ on SDA/SCL)
- [ ] SPS30 power supply (5V regulated)
- [ ] ZE07-CO power supply (5V regulated, separate from sensors)
- [ ] I2C addresses confirmed (SCD41=0x62, BME680=0x77, SPS30=0x69)
- [ ] UART pins assigned (GPIO17/18 for ZE07)
- [ ] Firmware compiled without errors

### First Boot Verification
- [ ] All I2C devices found (check logs)
- [ ] ZE07 UART receiving data
- [ ] SPS30 PM values appearing in HA
- [ ] All readings non-zero and stable after 2 minutes

### Performance Validation
- [ ] SPS30 update interval doesn't overload I2C
- [ ] ZE07 UART not blocking main loop
- [ ] AQ score calculating from all 4 sensors
- [ ] Alerts triggering at expected thresholds

### Data Logging (if enabled)
- [ ] SD card slot working
- [ ] CSV files being written
- [ ] Timestamps accurate
- [ ] Sufficient free space for 30+ days of data

---

## Next Steps

1. **Verify ZE07-CO protocol** - Confirm AT commands vs Modbus
2. **Test SPS30 I2C** - Run I2C scan to see address
3. **Compile V2 firmware** - Use air_quality_pro_v2.yaml
4. **Update HA automations** - Reference new sensor IDs
5. **Document differences** - For customer support

**Timeline:** 2-3 hours for full integration and testing
