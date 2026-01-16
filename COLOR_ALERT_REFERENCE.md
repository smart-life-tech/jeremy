# Professional Color Coding & Alert Reference

## Color System (for Display + HA)

### RGB Hex Values
```
🟢 GREEN (#00FF00)     = Excellent/Normal/Safe
🟡 YELLOW (#FFFF00)    = Elevated/Caution/Monitor
🟠 ORANGE (#FF8800)    = Poor/Warning/Concern
🔴 RED (#FF0000)       = Critical/Hazardous/Emergency
⚪ GRAY (#888888)       = No Data/Invalid/Offline
```

---

## Air Quality Score System

### Score → Alert Status → Color → Action

| Score | Status | Color | Color Hex | Action | Response Time |
|-------|--------|-------|-----------|--------|----------------|
| 75-100% | EXCELLENT | GREEN | #00FF00 | Monitor only | No action needed |
| 50-75% | GOOD | YELLOW | #FFFF00 | Monitor trends | Routine checks |
| 25-50% | FAIR | ORANGE | #FF8800 | Active monitoring | Increase ventilation |
| 10-25% | POOR | RED | #FF0000 | **Alert issued** | Immediate ventilation |
| 0-10% | HAZARDOUS | RED + BLINK | #FF0000 | **Critical alert** | Evacuate/Emergency |

---

## Individual Gas Thresholds

### Carbon Monoxide (CO)

| ppm Range | Status | Severity | Action | Color |
|-----------|--------|----------|--------|-------|
| 0-10 | NORMAL | — | Monitor | 🟢 GREEN |
| 10-35 | ELEVATED | Low | Ventilate | 🟡 YELLOW |
| 35-100 | WARNING | Medium | Strong ventilation | 🟠 ORANGE |
| 100-500 | ALERT | High | **Evacuate** | 🔴 RED |
| >500 | CRITICAL | EXTREME | **Emergency 911** | 🔴 RED BLINK |

**OSHA PEL**: 50 ppm (8-hour exposure)  
**IDLH**: 1200 ppm (life-threatening)

---

### Ammonia (NH₃)

| ppm Range | Status | Severity | Action | Color |
|-----------|--------|----------|--------|-------|
| 0-25 | NORMAL | — | Monitor | 🟢 GREEN |
| 25-50 | ELEVATED | Low | Ventilate | 🟡 YELLOW |
| 50-100 | WARNING | Medium | Strong ventilation | 🟠 ORANGE |
| 100-300 | ALERT | High | **Evacuate** | 🔴 RED |
| >300 | CRITICAL | EXTREME | **Emergency 911** | 🔴 RED BLINK |

**OSHA PEL**: 50 ppm  
**IDLH**: 300 ppm  
**Health effect**: Corrosive to respiratory tract

---

### Nitrogen Dioxide (NO₂)

| ppm Range | Status | Severity | Action | Color |
|-----------|--------|----------|--------|-------|
| 0-0.5 | NORMAL | — | Monitor | 🟢 GREEN |
| 0.5-2 | ELEVATED | Low | Ventilate | 🟡 YELLOW |
| 2-5 | WARNING | Medium | Strong ventilation | 🟠 ORANGE |
| 5-15 | ALERT | High | **Evacuate** | 🔴 RED |
| >15 | CRITICAL | EXTREME | **Emergency 911** | 🔴 RED BLINK |

**OSHA PEL**: 5 ppm (ceiling)  
**IDLH**: 20 ppm  
**Health effect**: Respiratory irritant, reactive gas

---

### Carbon Dioxide (CO₂)

| ppm Range | Status | Severity | Action | Color |
|-----------|--------|----------|--------|-------|
| <600 | EXCELLENT | — | No action | 🟢 GREEN |
| 600-1000 | GOOD | — | Monitor | 🟡 YELLOW |
| 1000-1500 | ACCEPTABLE | Low | Ventilate | 🟡 YELLOW |
| 1500-2000 | POOR | Medium | Active ventilation | 🟠 ORANGE |
| >2000 | CRITICAL | High | Full ventilation | 🔴 RED |

**Typical outdoor**: 400 ppm  
**OSHA PEL**: 5000 ppm  
**IDLH**: 40,000 ppm  
**Health effect**: Asphyxiation at high levels, cognitive impairment at 1000-2000ppm

---

### Volatile Organic Compounds (VOC/IAQ)

| IAQ Score | Status | Severity | Action | Color |
|-----------|--------|----------|--------|-------|
| 0-50 | EXCELLENT | — | No action | 🟢 GREEN |
| 50-150 | GOOD | — | Monitor | 🟡 YELLOW |
| 150-300 | ACCEPTABLE | Low | Ventilate | 🟡 YELLOW |
| 300-400 | POOR | Medium | Active ventilation | 🟠 ORANGE |
| >400 | HAZARDOUS | High | Emergency ventilation | 🔴 RED |

**IAQ Scale**: 0 (excellent) to 500 (hazardous)  
**BME680 Algorithm**: Proprietary Bosch BSEC library  
**Detects**: Cooking odors, cleaning chemicals, perfumes, off-gassing

---

## Alert Classification System

### Level 0: NORMAL (Green)
```
Condition: All readings within safe ranges
Display: Green color, "EXCELLENT" or "GOOD"
Notification: None
HA Action: None
Action Required: Routine monitoring
```

### Level 1: WARNING (Yellow)
```
Condition: CO₂ 1000-1500 OR VOC 150-300 OR elevated gas ratios
Display: Yellow color, "FAIR"
Notification: Subtle notification on device
HA Action: Optional: gentle notification
Action Required: Increase ventilation, monitor trends
```

### Level 2: ALERT (Orange)
```
Condition: CO₂ >1500 OR VOC >300 OR gases at warning levels
Display: Orange color, "POOR"
Notification: Clear notification on device
HA Action: Turn on HVAC/fans, optional mobile notification
Action Required: Active ventilation immediately
```

### Level 3: CRITICAL (Red + Flashing)
```
Condition: CO >100 OR NH₃ >150 OR NO₂ >10 OR AQ <10%
Display: Red color, FLASHING
Notification: LOUD alarm on device
HA Action: All ventilation ON, mobile emergency notification
Action Required: EVACUATE, call emergency services if CO
```

---

## HA Automation Trigger Examples

### Gentle Reminders (Level 1)
```yaml
trigger: numeric_state
entity: sensor.scd41_co2
above: 1200
# Result: "Gentle notification" + increase ventilation to 50%
```

### Strong Warnings (Level 2)
```yaml
trigger: numeric_state
entity: sensor.scd41_co2
above: 1800
# Result: "Strong notification" + increase ventilation to 100%
```

### Emergency Response (Level 3)
```yaml
trigger: state
entity: binary_sensor.co_alert
to: "on"
# Result: SIREN + flashing lights + mobile emergency notification
```

---

## Display Colors in LVGL Code

```yaml
# Color values for your display.yaml
colors:
  excellent_green: 0x00FF00      # (0, 255, 0)
  good_yellow: 0xFFFF00          # (255, 255, 0)
  poor_orange: 0xFF8800          # (255, 136, 0)
  critical_red: 0xFF0000         # (255, 0, 0)
  warning_red: 0xFF0000          # Same as critical
  neutral_gray: 0x888888         # (136, 136, 136)
  background_dark: 0x0a0a14      # Dark blue-black
  text_bright: 0xFFFFFF          # White
  text_dim: 0x666666             # Dark gray
```

### Example LVGL Update
```yaml
text_color: !lambda |-
  if (!id(scd41_co2).has_state()) {
    return lv_color_hex(0x888888);  // Gray - no data
  }
  float co2 = id(scd41_co2).state;
  if (co2 < 600) return lv_color_hex(0x00FF00);    // Green
  if (co2 < 1000) return lv_color_hex(0xFFFF00);   // Yellow
  if (co2 < 1500) return lv_color_hex(0xFFFF00);   // Yellow
  if (co2 < 2000) return lv_color_hex(0xFF8800);   // Orange
  return lv_color_hex(0xFF0000);                    // Red
```

---

## Data Quality Indicators

### Quality Score Colors

| Score | Status | Color | Meaning |
|-------|--------|-------|---------|
| 80-100% | EXCELLENT | 🟢 | All sensors healthy, baseline valid |
| 60-80% | GOOD | 🟡 | Minor issue (age, missing sensor) |
| 40-60% | FAIR | 🟠 | Baseline not set or stale (>7 days) |
| 20-40% | POOR | 🔴 | Multiple sensors offline |
| 0-20% | CRITICAL | 🔴 BLINK | System unreliable, recalibrate |

### Quality Deductions
- **-40%**: Gas baseline not set
- **-15%**: SCD41 CO₂ sensor offline
- **-15%**: BME680 VOC sensor offline
- **-10%**: Calibration >7 days old
- **0-5%**: WiFi weak (<50% signal)

---

## Professional Health Impact Summary

### At These Levels, People Experience:

**CO₂ (1000-1500 ppm)**
- Reduced cognitive function
- Decreased decision-making ability
- Increased drowsiness
- Classroom/office productivity drops 50%

**CO₂ (>1500 ppm)**
- Headaches
- Difficulty concentrating
- Fatigue
- "Stale air" feeling (even if not dangerous)

**CO (10-35 ppm)**
- Elevated blood carboxyhemoglobin (COHb)
- Subtle cognitive effects
- Headache possible with prolonged exposure

**CO (35-100 ppm)**
- Headache, dizziness
- Difficulty breathing
- Heart palpitations possible

**CO (>100 ppm)**
- **DANGER**: Life-threatening
- Severe headache within minutes
- Loss of consciousness possible
- **ACTION: EVACUATE IMMEDIATELY**

**NH₃ (25-50 ppm)**
- Eye, nose, throat irritation
- Respiratory tract irritation
- Smell detectable

**NH₃ (>100 ppm)**
- **DANGER**: Severe respiratory tract burns
- Eye damage
- Systemic toxicity
- **ACTION: EVACUATE IMMEDIATELY**

**NO₂ (2-5 ppm)**
- Respiratory tract irritation
- Increased airway resistance
- Increased susceptibility to respiratory infection

**NO₂ (>10 ppm)**
- **DANGER**: Severe lung damage
- Pulmonary edema (fluid in lungs)
- Chronic respiratory damage
- **ACTION: EVACUATE IMMEDIATELY**

---

## Alert Escalation Flow

```
┌─────────────────────────────┐
│  NORMAL (Green)             │
│  Score: 75-100%             │
│  All gases within limits     │
│  Action: Monitor only        │
└────────────┬────────────────┘
             │
             ↓ (CO₂ >1000 OR VOC >150)
             │
┌─────────────────────────────┐
│  WARNING (Yellow)           │
│  Score: 50-75%              │
│  Elevated readings           │
│  Action: Increase ventilation│
└────────────┬────────────────┘
             │
             ↓ (CO₂ >1500 OR VOC >300)
             │
┌─────────────────────────────┐
│  ALERT (Orange)             │
│  Score: 25-50%              │
│  Poor air quality            │
│  Action: Full ventilation    │
└────────────┬────────────────┘
             │
             ↓ (CO >100 OR specific gas CRITICAL)
             │
┌─────────────────────────────┐
│  CRITICAL (Red, Flashing)   │
│  Score: 0-25%               │
│  HAZARDOUS CONDITIONS        │
│  Action: EVACUATE + 911      │
└─────────────────────────────┘
```

---

## Customer Communication Template

### For Website/Marketing
```
🟢 Green (Excellent): Perfect air quality, no action needed
🟡 Yellow (Good): Air quality acceptable, routine monitoring
🟠 Orange (Poor): Take action, increase ventilation
🔴 Red (Critical): Emergency condition, evacuate if needed

Your device monitors 3 critical gas types:
✓ CO₂ for comfort & cognitive function
✓ VOC/IAQ for odors, chemicals, off-gassing
✓ CO/NH₃/NO₂ for safety hazards

Professional calibration ensures accurate readings.
Real-time HA integration enables smart automation.
Mobile alerts keep you informed anywhere.
```

### For Support/Documentation
```
"The color indicates urgency, not danger necessarily:
- Green = no immediate action
- Yellow = awareness + light response
- Orange = active management
- Red = emergency response

Your response should match the color. Yellow means
'ventilate a bit more', not 'evacuate'."
```

---

## Emergency Response Checklist

When Level 3 (RED) Alert Triggers:

- [ ] **Immediate** (0-5 seconds):
  - Device sounds alarm/siren
  - Mobile notification received
  - All ventilation turns ON

- [ ] **First 30 seconds**:
  - Get fresh air/evacuate if CO alert
  - Open windows and doors
  - Shut off ignition sources if CO

- [ ] **Within 1 minute**:
  - Call 911 if CO >200 ppm
  - Leave building if not improving
  - Check for gas appliance issues

- [ ] **Post-incident**:
  - Professional inspection if CO source
  - Reset device once hazard cleared
  - Recalibrate baseline in clean air
  - Review automation logs in HA

---

## Professional Language for Integrators

When presenting to clients:

> "The system uses a **3-tier alert hierarchy** aligned with safety standards:
> - **ACTIVE MONITORING** (Green/Yellow) for normal variations
> - **RESPONSIVE CONTROL** (Orange) for automatic adjustments
> - **EMERGENCY PROTOCOLS** (Red) for critical conditions
>
> Each level triggers appropriate automations without false alarms.
> Thresholds based on OSHA, WHO, and EPA standards."

This positions your system as **professional and standards-based**, not just "another sensor."

---

## Final Reference

Keep this document handy when:
- Customizing alert thresholds for specific environments
- Creating customer documentation
- Training installers on alert responses
- Debugging sensor readings
- Presenting to professional clients
- Setting up automation responses in HA
