# Waveshare ESP32-S3 Touch LCD 2.8" Type B - Complete ESPHome Guide

This repository documents the complete process of getting the Waveshare ESP32-S3 Touch LCD 2.8" Type B display working with ESPHome, including LVGL GUI, Material Design icons, and dynamic UI elements.

## 🎯 Display Specifications

- **Model**: ESP32-S3 2.8inch Display Development Board Type B
- **Display Controller**: ST7701S
- **Resolution**: 480×640 pixels (portrait)
- **Touch Controller**: GT911
- **Color Format**: BGR (important for correct colors!)
- **Interface**: RGB parallel (16-bit)

## 📋 Table of Contents

1. [Hardware Setup](#hardware-setup)
2. [The Core Problem](#the-core-problem)
3. [The Solution](#the-solution)
4. [Critical Configuration Points](#critical-configuration-points)
5. [LVGL GUI with Material Design](#lvgl-gui-with-material-design)
6. [BGR Color Conversion](#bgr-color-conversion)
7. [Complete Example](#complete-example)
8. [Troubleshooting](#troubleshooting)

---

## 🔧 Hardware Setup

### Pin Configuration

The display uses the following GPIO pins on the ESP32-S3:

```yaml
# I2C (for touchscreen and PCA9554)
SDA: GPIO15
SCL: GPIO07

# SPI (for display)
CLK: GPIO02
MOSI: GPIO01

# Display Control (via PCA9554 I/O expander)
CS: PCA9554 Pin 2
RESET: PCA9554 Pin 0
Touch RESET: PCA9554 Pin 1

# RGB Parallel Data
DE: GPIO40
HSYNC: GPIO38
VSYNC: GPIO39
PCLK: GPIO41

# RGB Data Pins
RED: [46, 3, 8, 18, 17]
GREEN: [14, 13, 12, 11, 10, 9]
BLUE: [5, 45, 48, 47, 21]

# Backlight
PWM: GPIO06

# Touch Interrupt
INT: GPIO16
```

---

## 🚨 The Core Problem

The ST7701S display controller requires a **vendor-specific initialization sequence**. Using generic or incomplete sequences results in:
- Display powers on but remains black/white
- Backlight works but no pixels display
- Touch may or may not work

### Why Generic Sequences Fail

The ST7701S is a complex controller requiring:
1. Multi-page register access (pages 0x10, 0x11, 0x13)
2. Precise gamma correction settings
3. Power control sequences
4. Clock and timing configurations
5. Specific VCOM voltage settings

Missing even one command or using wrong timing can prevent the display from working.

---

## ✅ The Solution

### 1. The Correct Init Sequence

The working initialization sequence was extracted from Waveshare's C driver for the 2.8" display. Key points:

```yaml
init_sequence:
  # Page switching and configuration
  - [ 0xFF, 0x77, 0x01, 0x00, 0x00, 0x13 ]  # Page 1
  - [ 0xEF, 0x08 ]
  
  # ... (full sequence in example YAML)
  
  # CRITICAL FINAL COMMANDS:
  - [ 0x11 ]      # Sleep Out
  - delay 200ms   # MUST wait for power stabilization
  - [ 0x29 ]      # Display ON
  - delay 100ms
  - [ 0x3A, 0x55 ] # 16-bit color
  - [ 0x36, 0x00 ] # Portrait orientation
```

### 2. The VCOM Power Delay

**This is the #1 reason for failure even with correct init sequence!**

The display needs time to charge internal voltage pumps before initialization:

```yaml
esphome:
  on_boot:
    - then: 
        - delay: 10s          # Wait for VCOM to stabilize
        - wifi.enable:        # Only then enable WiFi
        - delay: 1s
```

Without this delay:
- Display may flash and die
- Random crashes and reboots
- Guru Meditation errors

### 3. WiFi Must Be Disabled on Boot

```yaml
wifi:
  enable_on_boot: false  # CRITICAL!
```

This prevents WiFi from drawing power during the critical VCOM stabilization period.

---

## 🎨 Critical Configuration Points

### A. Color Order: BGR

This display uses **BGR color order**, not RGB. This is critical for LVGL:

```yaml
display:
  - platform: st7701s
    color_order: bgr  # NOT rgb!
```

### B. Portrait Orientation

For 480×640 portrait mode:

```yaml
dimensions:
  width: 480
  height: 640
rotation: 0  # Portrait
```

The MADCTL command must match:
```yaml
- [ 0x36, 0x00 ]  # Portrait: 0x00
```

For landscape (640×480), use:
```yaml
rotation: 90
- [ 0x36, 0xA0 ]  # Landscape: 0xA0
```

### C. PSRAM Configuration

The ESP32-S3 needs proper PSRAM settings for smooth LVGL operation:

```yaml
esp32:
  framework:
    type: esp-idf
    sdkconfig_options:
      CONFIG_SPIRAM: y
      CONFIG_SPIRAM_MODE_OCT: y
      CONFIG_SPIRAM_SPEED_80M: y
      CONFIG_SPIRAM_USE_MALLOC: y
      # ... (see full config)
```

### D. LVGL Buffer Size

```yaml
lvgl:
  buffer_size: 25%  # Good balance of memory and performance
  byte_order: little_endian
```

---

## 🎨 LVGL GUI with Material Design

### Material Design Icons

To use Material Design Icons in your UI:

```yaml
font:
  - file: https://github.com/Templarian/MaterialDesign-Webfont/raw/master/fonts/materialdesignicons-webfont.ttf
    id: icons_large
    size: 48
    bpp: 2
    glyphs: [
      "\U000F0594",  # mdi-weather-night
      "\U000F050F",  # mdi-thermometer
      "\U000F0335",  # mdi-lightbulb
    ]
```

Find icon codes at: https://pictogrammers.com/library/mdi/

### Dynamic Button Colors

Create buttons that change color based on state:

```yaml
binary_sensor:
  - platform: homeassistant
    id: light_state
    entity_id: light.your_light
    on_state:
      then:
        - if:
            condition:
              binary_sensor.is_on: light_state
            then:
              # Yellow when ON (remember BGR format!)
              - lvgl.widget.update:
                  id: btn_light
                  bg_color: 0x3BEBFF  # Yellow in BGR
              - lvgl.label.update:
                  id: lbl_status
                  text: "ON"
            else:
              # Gray when OFF
              - lvgl.widget.update:
                  id: btn_light
                  bg_color: 0x424242
              - lvgl.label.update:
                  id: lbl_status
                  text: "OFF"
```

---

## 🎨 BGR Color Conversion

**CRITICAL**: This display uses BGR, so RGB colors must be converted!

### Color Conversion Formula

RGB color `0xRRGGBB` becomes BGR `0xBBGGRR`

### Common Color Conversions

| Color | RGB Code | BGR Code | Usage |
|-------|----------|----------|-------|
| Red | `0xD32F2F` | `0x2F2FD3` | Header background |
| Orange | `0xFF8A65` | `0x658AFF` | Thermometer icon |
| Light Blue | `0x90CAF9` | `0xF9CA90` | Moon icon |
| Yellow | `0xFFEB3B` | `0x3BEBFF` | Button ON state |
| White | `0xFFFFFF` | `0xFFFFFF` | Text (symmetric) |
| Gray | `0x424242` | `0x424242` | Cards (symmetric) |
| Black | `0x000000` | `0x000000` | Background (symmetric) |

### Example: Red Header

**Wrong (will show as cyan):**
```yaml
bg_color: 0xD32F2F  # Red in RGB
```

**Correct (shows as red):**
```yaml
bg_color: 0x2F2FD3  # Red in BGR
```

---

## 📦 Complete Example

See `waveshare-material-design-anonymized.yaml` for a complete working example featuring:

- ✅ Material Design dark theme
- ✅ Material Design icons (home, clock, thermometer, lightbulb)
- ✅ Dynamic button that changes from gray to yellow when light turns on
- ✅ Proper BGR color handling
- ✅ Home Assistant integration
- ✅ Touch support
- ✅ Time display
- ✅ Temperature sensor display
- ✅ Light control with visual feedback

---

## 🔍 Troubleshooting

### Display is Black/White

**Symptoms**: Backlight works, but no image
**Causes**:
1. Wrong init sequence
2. Insufficient VCOM delay
3. WiFi enabled too early

**Solutions**:
- Use the exact init sequence from this repo
- Ensure 10s delay in `on_boot`
- Set `enable_on_boot: false` for WiFi

### Wrong Colors Displayed

**Symptoms**: Red shows as cyan, blue shows as yellow, etc.
**Cause**: Not using BGR color format

**Solution**: Convert all RGB colors to BGR using formula: `0xRRGGBB` → `0xBBGGRR`

### Touch Not Working

**Symptoms**: Display works but touch doesn't respond
**Causes**:
1. GT911 not properly initialized
2. Touch interrupt pin not configured
3. Touch reset pin not toggled

**Solutions**:
- Ensure I2C is working (`scan: true`)
- Check PCA9554 is detected
- Verify touch reset pin configuration

### Guru Meditation Errors / Crashes

**Symptoms**: Random crashes, memory errors, device reboots
**Causes**:
1. PSRAM not configured correctly
2. LVGL buffer too large
3. Memory allocation issues
4. WiFi interfering with display init

**Solutions**:
- Use recommended PSRAM settings
- Reduce buffer_size to 20-25%
- Ensure proper `on_boot` sequence
- Monitor logs for memory issues

### Icons Not Displaying

**Symptoms**: Boxes or wrong symbols instead of icons
**Causes**:
1. Wrong unicode code
2. Icon not in font glyphs list
3. Icon font not loading

**Solutions**:
- Verify unicode from https://pictogrammers.com/library/mdi/
- Add icon to glyphs list
- Check font file downloads successfully

### Layout Issues / Text Overlapping

**Symptoms**: Text squashed together, elements overlapping
**Causes**:
1. Wrong positioning (x/y values)
2. Font size too large
3. Card height too small

**Solutions**:
- Use TOP_MID/BOTTOM_MID alignment with appropriate y offsets
- Reduce font sizes
- Increase card heights
- Test different pad_all values

---

## 🎓 Key Takeaways

1. **The init sequence is everything** - Use the exact sequence from Waveshare's C driver
2. **VCOM delay is critical** - Always wait 10s before WiFi
3. **BGR not RGB** - Convert all colors or they'll display wrong
4. **WiFi disable on boot** - Prevents power issues during init
5. **PSRAM configuration matters** - Use the recommended settings for smooth LVGL
6. **Portrait vs Landscape** - MADCTL byte must match rotation setting
7. **Touch needs reset** - GT911 requires proper PCA9554 pin toggling

---

## 📚 Resources

- **ESPHome Documentation**: https://esphome.io/components/display/st7701s
- **LVGL Documentation**: https://docs.lvgl.io/
- **Material Design Icons**: https://pictogrammers.com/library/mdi/
- **Waveshare Wiki**: https://www.waveshare.com/wiki/ESP32-S3-Touch-LCD-2.8B

---

## 🙏 Credits

This configuration was developed through extensive testing and debugging to create a fully working Material Design interface for the Waveshare ESP32-S3 Touch LCD 2.8" Type B display.

Special thanks to:
- Waveshare for the C driver source code
- ESPHome community for the ST7701S component
- Material Design Icons project

---

## 📝 License

This configuration is provided as-is for educational purposes. Feel free to use and modify for your own projects.

---

## ⭐ Contributing

Found an issue or improvement? Please open an issue or pull request!
