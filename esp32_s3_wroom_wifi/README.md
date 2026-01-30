ESP32-S3-WROOM – Wi-Fi Display & HTTP API Node
Overview

The ESP32-S3-WROOM acts as a Wi-Fi–enabled display and API endpoint in the three-microcontroller system. Its primary role is to receive HTTP requests (either directly from a web client or relayed by the Pico 2W), process JSON payloads, and present information on a 16×2 I²C OLED/LCD display.

In the current design, the ESP32 does not initiate communication with the other microcontrollers. Instead, it behaves as a passive network service that responds to incoming HTTP requests and updates its display accordingly.

Responsibilities

The ESP32 is responsible for:

Connecting to the local Wi-Fi network as a station

Hosting an embedded HTTP server

Exposing REST-style API endpoints

Parsing JSON payloads

Displaying text and status information on a 16×2 OLED

Reporting its own runtime status (IP address, uptime, free heap)

Communication Model

The ESP32 participates in the system using the following communication paths:

Web Client ──HTTP──▶ Pico 2W ──HTTP──▶ ESP32


Optionally (for testing or diagnostics), the ESP32 can be accessed directly by a browser or cURL client on the local network:

Web Client ──HTTP──▶ ESP32

Key Design Rule

The ESP32 never communicates directly with the SPI slave Pico.
All network-originated commands flow through the Pico 2W, which acts as the system coordinator.

API Endpoints
POST /api/text

Displays a text message on the 16×2 OLED using word wrapping.

Request body (JSON):

{
  "text": "Hello from the Pico 2W"
}


Behavior:

Parses the JSON payload

Extracts the text field

Wraps the message across two 16-character lines

Displays the text on the OLED

Response:

{ "ok": true }

POST /api/cmd

Displays temperature values (or other numeric command data) on the OLED.

Request body (JSON):

{
  "tf": 72,
  "tc": 22
}


Behavior:

Displays Fahrenheit on line 1

Displays Celsius on line 2

Response:

{ "ok": true }

GET /api/status

Returns the ESP32’s internal runtime status.

Response (JSON):

{
  "status": "ok",
  "uptime_ms": 133598538,
  "free_heap": 268680
}


This endpoint is primarily used for diagnostics and system health monitoring.

OLED Display Behavior

Display size: 16×2 characters

Interface: I²C

Text messages are word-wrapped where possible

Numeric values (temperature, status messages) are displayed in fixed positions

Wi-Fi connection status and IP address are briefly displayed after successful connection

Wi-Fi Behavior

The ESP32 connects to a predefined SSID in station mode

On boot:

Displays “ESP32 Ready / Waiting…”

Attempts Wi-Fi connection

On successful connection:

Displays assigned IP address

Starts the HTTP server

On disconnect:

Automatically retries connection

Updates display with retry status

Firmware Architecture
Key Components

FreeRTOS (default ESP-IDF task model)

esp_http_server for REST endpoints

cJSON for JSON parsing

i2c_lcd driver for OLED control

esp_event for Wi-Fi and IP events

Execution Model

No explicit multi-core partitioning is required

ESP-IDF schedules Wi-Fi, HTTP, and LCD updates cooperatively

HTTP handlers are lightweight and non-blocking

Wiring
I²C OLED Connections
OLED Pin	ESP32 Pin
SDA	Configured in i2c_lcd
SCL	Configured in i2c_lcd
VCC	3.3V or 5V (module dependent)
GND	GND

No SPI or UART connections are required for the ESP32 in the current design.

Development Notes

Written using ESP-IDF

Built and flashed via PlatformIO or idf.py

Serial logging via ESP_LOGI() is used extensively for diagnostics

Designed to be robust against malformed JSON and partial HTTP bodies

Role in the System (Summary)
Function	ESP32
Wi-Fi client	✅
HTTP server	✅
JSON parsing	✅
OLED display	✅
SPI communication	❌
System coordinator	❌
Direct Pico slave control	❌
The ESP32 acts as a network-visible display endpoint, while the Pico 2W remains the central orchestrator of the system.

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
