esp32_s3_wroom_wifi will turn your esp32-s3-wroom into a wi-fi station that does several operations:
- transacts with the pico 2w over wi-fi, receiving text strings that originate from the external website.
- the received text strings are outputted on a 16X2 OLED. That means the maximum string size should be 32 characters, otherwise clipping will occur.
- outputs to various LEDs, etc.

Look to the GPIO settings in the main.c program to know or change input/output pins. The only meaningful ones for this project are the pins used for the 16X2 OLED.

In this configuration, both esp32 cores are used: Core0 does things like reading from GPIO pins or toggling them. Core1 is delegated all Wi-Fi functionality. I like to think of it as the ESP32 being able to walk and chew gum at the same time.

The program was written and compiled using Visual Studio Code with the PlatformIO extension.

My platformio.ini file:
; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter
;   Upload options: custom upload port, speed and extra flags
;   Library options: dependencies, extra library storages
;   Advanced options: extra scripting
;
; Please visit documentation for the other options and examples
; https://docs.platformio.org/page/projectconf.html

[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = espidf
