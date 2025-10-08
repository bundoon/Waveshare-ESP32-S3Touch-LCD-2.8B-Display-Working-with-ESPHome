# Waveshare-ESP32-S3Touch-LCD-2.8B-Display-Working-with-ESPHome
This post details the successful process of configuring Waveshare ESP32-S3Touch LCD 2.8B display, which uses the ST7701S display controller. The core issue lies in finding the exact, vendor-specific initialization sequence required by the display panel. 
The solution came from translating a working C driver implementation (specifically, the 2.8-inch routine from a Waveshare demo) into the correct ESPHome init_sequence format.

The Problem: The Incompatible init_sequence
The ST7701S is a complex controller that requires a multi-page, precise initialization sequence involving commands for power control, gamma correction, and clock settings. Using generic or incomplete sequences results in the display powering on but remaining black or white (i.e., initialized but not displaying pixels).

The Solution: The C Driver Translation
The key to success was extracting the initialization commands from a working C driver for the 2.8-inch display (e.g., the ST7701S.c file often provided in vendor demos).

This routine was translated into the ESPHome YAML format: [Command, Data1, Data2, ...] or delay Xms.
