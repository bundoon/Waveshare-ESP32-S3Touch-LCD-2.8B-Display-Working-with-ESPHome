Waveshare ESP32-S3 Touch LCD 2.8" Type B — Complete ESPHome Guide

This repository documents the complete process of getting the Waveshare ESP32-S3 Touch LCD 2.8" Type B working with ESPHome + LVGL, including:

✅ Stable RGB display

✅ GT911 touch

✅ Material Design UI

✅ Home Assistant integration

✅ Rock-solid timing (no tearing / flicker)

🎯 Display Specifications
Item	Value
Controller	ST7701S
Resolution	480 × 640 (portrait)
Interface	16-bit RGB parallel
Touch	GT911
I/O Expander	PCA9554 @ 0x20
Color Order	RGB
Driver	ESPHome mipi_rgb
📋 Table of Contents

Hardware Setup

The Core Problem

The Working Solution

Known-Stable Settings ⭐

ST7701 Init Sequence

LVGL GUI

Troubleshooting

🔧 Hardware Setup
Pin Configuration
I²C
SDA → GPIO15  
SCL → GPIO7

SPI (used for ST7701 init only)
CLK  → GPIO2  
MOSI → GPIO1

Control (via PCA9554)
LCD RESET   → P0  
TOUCH RESET → P1  
LCD CS      → P2

RGB Timing
DE    → GPIO40  
HSYNC → GPIO38  
VSYNC → GPIO39  
PCLK  → GPIO41

RGB Data
RED   → [46, 3, 8, 18, 17]  
GREEN → [14, 13, 12, 11, 10, 9]  
BLUE  → [5, 45, 48, 47, 21]

Backlight
GPIO6 (LEDC PWM @ 20 kHz)

Touch Interrupt
GPIO16

🚨 The Core Problem

The ST7701S:

Does NOT work with generic configs

Requires a vendor init sequence

Needs correct RGB timing

Is sensitive to pixel clock stability

Symptoms of incorrect setup:

Backlight on, no image

Rolling / flickering display

Random LVGL crashes

✅ The Working Solution

This display is 100 % stable using:

platform: mipi_rgb

PCA9554 for reset & CS

Correct vendor init sequence

Reduced pixel clock

⚠️ With this method:

❌ No 10-second VCOM delay required
❌ No Wi-Fi boot delay required

⭐ Known-Stable Settings

These are the magic numbers for a rock-solid display:

pclk_frequency: 16MHz
data_rate: 80MHz

lvgl:
  buffer_size: 35%

wifi:
  power_save_mode: none


Result:

No tearing

No flicker

No periodic glitches

📺 ST7701 Init Sequence (Waveshare ESP32-S3 2.8B)

This is the exact vendor sequence extracted from the Waveshare C driver.

<details> <summary>Click to expand</summary>
init_sequence:
  - [0xFF,0x77,0x01,0x00,0x00,0x13]
  - [0xEF,0x08]

  - [0xFF,0x77,0x01,0x00,0x00,0x10]
  - [0xC0,0x4F,0x00]
  - [0xC1,0x10,0x02]
  - [0xC2,0x07,0x02]
  - [0xCC,0x10]

  - [0xB0,0x00,0x10,0x17,0x0D,0x11,0x06,0x05,0x08,0x07,0x1F,0x04,0x11,0x0E,0x29,0x30,0x1F]
  - [0xB1,0x00,0x0D,0x14,0x0E,0x11,0x06,0x04,0x08,0x08,0x20,0x05,0x13,0x13,0x26,0x30,0x1F]

  - [0xFF,0x77,0x01,0x00,0x00,0x11]
  - [0xB0,0x65]
  - [0xB1,0x71]
  - [0xB2,0x82]
  - [0xB3,0x80]
  - [0xB5,0x42]
  - [0xB7,0x85]
  - [0xB8,0x20]
  - [0xC0,0x09]
  - [0xC1,0x78]
  - [0xC2,0x78]
  - [0xD0,0x88]
  - [0xEE,0x42]

  - [0x11]
  - delay 200ms
  - [0x35,0x00]

</details>
🎨 LVGL GUI

Material Design icons work normally:

font:
  - file: https://github.com/Templarian/MaterialDesign-Webfont/raw/master/fonts/materialdesignicons-webfont.ttf


Dynamic UI updates via Home Assistant sensors are fully supported.

🔍 Troubleshooting
Black Screen

✅ Check:

mipi_rgb (NOT st7701s / rpi_dpi_rgb)

PCA9554 detected at 0x20

Correct init sequence

Backlight GPIO6

Periodic Glitch / Tear

Lower pixel clock:

pclk_frequency: 16MHz

Random Crashes

Enable PSRAM

Use buffer ≤ 35 %

Touch Not Working

Confirm GT911 @ 0x5D

Verify interrupt pin = GPIO16

Ensure touch reset via PCA9554

🎓 Key Takeaways

✔ Use mipi_rgb
✔ Vendor init sequence is mandatory
✔ 16 MHz pixel clock = stability
✔ PCA9554 control is required
✔ LVGL in PSRAM for smooth UI

📚 Resources

ESPHome
https://esphome.io/

LVGL
https://lvgl.io/

Waveshare Wiki
https://www.waveshare.com/wiki/ESP32-S3-Touch-LCD-2.8B

🙏 Credits

Waveshare (C driver reference)

ESPHome display component developers

LVGL project

⭐ Contributing

PRs & improvements welcome!
