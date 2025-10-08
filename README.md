# Waveshare-ESP32-S3Touch-LCD-2.8B-Display-Working-with-ESPHome
This post details the successful process of configuring Waveshare ESP32-S3Touch LCD 2.8B display, which uses the ST7701S display controller. The core issue lies in finding the exact, vendor-specific initialization sequence required by the display panel. 
The solution came from translating a working C driver implementation (specifically, the 2.8-inch routine from a Waveshare demo) into the correct ESPHome init_sequence format.

The Problem: The Incompatible init_sequence
The ST7701S is a complex controller that requires a multi-page, precise initialization sequence involving commands for power control, gamma correction, and clock settings. Using generic or incomplete sequences results in the display powering on but remaining black or white (i.e., initialized but not displaying pixels).

The Solution: The C Driver Translation
The key to success was extracting the initialization commands from a working C driver for the 2.8-inch display (e.g., the ST7701S.c file often provided in vendor demos).

This routine was translated into the ESPHome YAML format: [Command, Data1, Data2, ...] or delay Xms.

 Key Takeaways and Troubleshooting Notes
A. The VCOM/Power Delay is Critical
The most common failure point, even with the correct sequence, is insufficient power stability. The display needs time to charge internal voltage pumps before initialization begins.

Solution: The on_boot delay is essential. We used 10 seconds of initial delay before enabling Wi-Fi and starting the main program loop.

YAML

on_boot:
  - then: 
      - delay: 10s 
      - wifi.enable: # Wait until VCOM is stable before drawing power for WiFi
      - delay: 1s
B. The Final Commands are Non-Negotiable
The following commands, placed at the end of the init_sequence (after all the page-switching/gamma settings), are what turn the screen on. They must be included with the correct timing.

Command	Hex	Function	Required Delay
Sleep Out	0x11	Exits low-power mode.	delay 200ms
Display ON	0x29	Activates the display.	delay 100ms
Pixel Format	0x3A, 0x55	Sets color depth (16-bit) for LVGL.	None
MADCTL	0x36, 0xA0	Sets rotation/color order.	None
