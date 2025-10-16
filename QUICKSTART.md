# Quick Start Guide

Get your Waveshare ESP32-S3 Touch LCD 2.8" Type B up and running in minutes!

## ⚡ Prerequisites

- ESPHome installed
- Home Assistant (optional, for sensor/control integration)
- USB-C cable for programming
- The display hardware

## 🚀 5-Minute Setup

### Step 1: Copy the YAML

1. Download `waveshare-material-design-anonymized.yaml`
2. Rename it to your device name (e.g., `living-room-display.yaml`)
3. Place it in your ESPHome config directory

### Step 2: Update Secrets

Edit your `secrets.yaml`:

```yaml
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"
```

### Step 3: Customize the YAML

Replace these placeholders:

```yaml
# Line 51: API Key
key: "YOUR_API_KEY_HERE"

# Line 55: OTA Password  
password: "YOUR_OTA_PASSWORD_HERE"

# Line 148: Temperature Sensor
entity_id: sensor.YOUR_TEMPERATURE_SENSOR

# Line 163: Light Entity
entity_id: light.YOUR_LIGHT_ENTITY

# Line 456: Light Entity (in button action)
entity_id: light.YOUR_LIGHT_ENTITY
```

### Step 4: Generate Keys (if needed)

If you don't have an API key:

```bash
esphome run your-display.yaml
# When prompted, ESPHome will generate keys automatically
```

### Step 5: Upload

```bash
esphome run your-display.yaml
```

First upload must be via USB. Subsequent updates can use OTA.

### Step 6: Wait for Boot

⏱️ **Important**: The display takes ~15 seconds to fully boot:
- 10s VCOM stabilization
- 2-3s display initialization  
- 1-2s WiFi connection

Don't panic if it seems slow - this is normal and necessary!

---

## 🎨 Customization Guide

### Change Header Color

Current: Red (`0x2F2FD3` in BGR)

To change to blue:
```yaml
bg_color: 0xFF5733  # Blue in BGR (0x3357FF in RGB)
```

Remember: Colors must be in BGR format!

### Change Header Text

Line 339:
```yaml
text: "Smart Home"  # Change to whatever you want
```

### Adjust Card Heights

Current heights:
- Time card: 160px
- Temperature card: 160px  
- Button: 140px

To change:
```yaml
height: 180  # Make it taller
```

### Change Font Sizes

Current sizes:
- Small: 18px (labels)
- Medium: 26px (headers)
- Large: 56px (time, temperature)

To change:
```yaml
size: 24  # New size in pixels
```

### Add More Icons

1. Find icon at https://pictogrammers.com/library/mdi/
2. Copy unicode (e.g., `F0594`)
3. Add to glyphs list:
```yaml
glyphs: [
  "\U000F0594",
  "\U000F0YOUR_ICON",  # Add here
]
```

---

## 🎯 Common Customizations

### Display Temperature in Fahrenheit

Change line 152-156:
```yaml
text: !lambda |-
  char buf[32];
  snprintf(buf, sizeof(buf), "%.1f°F", x * 9.0/5.0 + 32);
  return std::string(buf);
```

### Show 12-Hour Time

Change line 449-455:
```yaml
t.strftime(buf, sizeof(buf), "%I:%M %p");  # 12-hour with AM/PM
```

### Add More Buttons

Copy the button section (lines 401-433) and modify:
- Change `id:` to unique name
- Change `entity_id:` to your entity
- Change `text:` to button label
- Adjust `y:` position

### Change Temperature Room Name

Line 387:
```yaml
text: "Living Room"  # Change to your room
```

---

## 📊 Color Reference (BGR Format)

Use these pre-converted colors:

| Color | BGR Code | Use Case |
|-------|----------|----------|
| Red | `0x2F2FD3` | Alerts, warnings |
| Blue | `0xFF5733` | Information, primary |
| Green | `0x00C853` | Success, on states |
| Orange | `0x658AFF` | Warm colors |
| Yellow | `0x3BEBFF` | Highlights, active |
| Purple | `0xFC86BB` | Accent |
| Gray | `0x424242` | Inactive, off |
| Light Gray | `0xBBBBBB` | Secondary text |
| White | `0xFFFFFF` | Primary text |
| Black | `0x121212` | Background |

### RGB to BGR Converter

Python one-liner:
```python
def rgb_to_bgr(rgb):
    return f"0x{rgb[4:6]}{rgb[2:4]}{rgb[0:2]}"

rgb_to_bgr("0xD32F2F")  # Returns: 0x2F2FD3
```

---

## 🔧 Troubleshooting Quick Fixes

### Display is Black
```yaml
# Check these lines:
on_boot: delay 10s present?
enable_on_boot: false set?
init_sequence: Complete?
```

### Wrong Colors
Convert RGB to BGR:
- Red RGB: `0xRRGGBB`
- Red BGR: `0xBBGGRR`

### Touch Not Working
```yaml
# Check I2C scan at boot:
i2c:
  scan: true  # Should see PCA9554 at 0x20
```

### Crashes/Reboots
```yaml
# Reduce memory usage:
buffer_size: 20%  # Try lower
# Remove unused fonts/icons
```

### Icons Not Showing
1. Check unicode is correct
2. Icon is in glyphs list
3. Font file downloaded successfully

---

## 📝 Testing Checklist

After uploading, verify:

- [ ] Display shows content (not black/white)
- [ ] Colors are correct (header is the color you intended)
- [ ] Icons display properly (not boxes)
- [ ] Time updates every 10 seconds
- [ ] Temperature shows current value
- [ ] Touch works (button responds)
- [ ] Button changes color when light toggles
- [ ] No crashes/reboots
- [ ] WiFi stays connected

---

## 🆘 Getting Help

If you're stuck:

1. **Check logs**: `esphome logs your-display.yaml`
2. **Common errors**: See main README troubleshooting section
3. **Verify wiring**: Compare with pin configuration above
4. **Test basics**: Start with minimal config, add features gradually

---

## 🎓 Next Steps

Once you have the basics working:

1. Read `JOURNEY.md` to understand the development process
2. Check `README.md` for detailed technical information
3. Customize colors and layout to your preference
4. Add more controls and sensors
5. Share your configuration with the community!

---

## 📚 Essential Links

- **Main README**: Comprehensive technical guide
- **Journey Doc**: Development process and lessons learned
- **ESPHome Docs**: https://esphome.io/components/display/st7701s
- **MDI Icons**: https://pictogrammers.com/library/mdi/
- **Color Picker**: Use any RGB picker, then convert to BGR

---

## ✅ Success!

If your display is showing:
- ✅ Correct colors
- ✅ Proper icons
- ✅ Updating time
- ✅ Responsive touch
- ✅ Working controls

**Congratulations! You're all set!** 🎉

Now customize it to your heart's content and enjoy your beautiful new display!
