# IP Address Setup Page

## Overview
A new editable IP address configuration page has been added to your ESPHome display. Users can now edit all 8 plug IP addresses directly from the display using an on-screen keyboard, and changes are persisted across device reboots.

## What Was Added

### 1. Global Variables (Persistent Storage)
Added 8 new global variables that store IP addresses with persistent storage:
- `plug1_ip_str` through `plug8_ip_str`
- Each defaults to the current substitution values
- All changes are automatically saved to device flash memory across reboots

### 2. New Page: `ip_setup_page`
Located after the last timer page (plug8_timer_page) in the display configuration

**Layout:**
- **Title**: "IP Address Setup" at the top
- **Left Column** (Plugs 1-4):
  - Plug 1 IP input field
  - Plug 2 IP input field
  - Plug 3 IP input field
  - Plug 4 IP input field

- **Right Column** (Plugs 5-8):
  - Plug 5 IP input field
  - Plug 6 IP input field
  - Plug 7 IP input field
  - Plug 8 IP input field

- **On-Screen Keyboard**: Full alphanumeric keyboard with:
  - All letters (A-Z)
  - Numbers (0-9)
  - Decimal point (.)
  - Text editing (LV_KEYBOARD_MODE_TEXT_LOWER)

- **Control Buttons**:
  - **Save IPs** (Green) - Saves all changes and returns to main page
  - **Back** (Gray) - Discards changes and returns to main page

## How to Use

### Accessing the Page
To add a button on the main page to access this setup page, add the following button widget:

```yaml
- button:
    id: btn_ip_setup
    x: 650     # Bottom right of main page
    y: 420
    width: 110
    height: 45
    bg_color: 0x4444FF
    widgets: [{ label: { text: "IP Setup", align: center, text_font: font_small }}]
    on_click:
      then:
        - lvgl.page.show: ip_setup_page
```

### Editing IPs
1. Tap on any IP input field to select it
2. The on-screen keyboard will activate
3. Type the new IP address (e.g., `192.168.68.100`)
4. Tap another field to edit it, or tap "Save IPs" to save all changes
5. Changes are automatically persisted to device flash memory

## Keyboard Features
- **Text Mode**: Full QWERTY keyboard layout
- **Auto-focus**: Keyboard automatically switches to the field you tap
- **One-Line Input**: Each field accepts single-line IP addresses
- **Placeholder Text**: Shows default IP when field is empty

## Technical Details

### Global Variables Storage
Each IP is stored in a global string variable with `restore_value: yes`:
```yaml
- id: plug1_ip_str
  type: std::string
  restore_value: yes
  initial_value: '"192.168.68.76"'
```

### Value Persistence
When a user edits an IP and clicks outside the field, the `on_value` event:
1. Updates the global variable
2. Logs the change to the console
3. Automatically persists to flash memory

### Textarea Properties
- `one_line: true` - Restricts input to single line
- `on_click` - Updates keyboard to match current textarea
- `on_value` - Captures changes and stores in global variable

## Integration with Plug URLs
To use these dynamically stored IPs in your HTTP requests, you would need to modify the plug toggle buttons:

**Before** (using substitutions):
```yaml
- http_request.post:
    url: "http://${plug1_ip}/switch/kauf_plug/toggle"
```

**After** (using stored values):
```yaml
- http_request.post:
    url: !lambda |-
      return "http://" + id(plug1_ip_str) + "/switch/kauf_plug/toggle";
```

## Notes
- IP addresses are stored as strings for maximum flexibility
- Changes take effect immediately after saving
- No reboot required to apply new IPs
- Logging shows all IP changes for troubleshooting
- Empty fields revert to placeholder text

## Adding the Setup Button to Main Page
To make the IP setup page easily accessible, you can add a button to the main page. Search for the backlight control buttons and add this after them (around line 1420):

```yaml
- button:
    id: btn_settings
    x: 580
    y: 420
    width: 200
    height: 45
    bg_color: 0xFF8800
    widgets: [{ label: { text: "IP Settings", align: center, text_font: font_small }}]
    on_click:
      then:
        - lvgl.page.show: ip_setup_page
```
