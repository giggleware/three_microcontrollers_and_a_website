Pico 2W — SPI Master & Wi-Fi Gateway
Overview

The Pico 2W is the central coordinator of the system. It acts as:

SPI master to a Pico-based slave device

Wi-Fi gateway between the local web client and the ESP32

Protocol translator, bridging wired SPI traffic and wireless HTTP traffic

All external interactions (web requests, user commands, and system status queries) flow through the Pico 2W. Neither the web client nor the SPI slave communicates directly with the ESP32.

In short, the Pico 2W is the middle manager that knows how to talk to everyone.

Responsibilities
1. SPI Master (Pico 2W ↔ Pico Slave)

The Pico 2W is the SPI master on a four-wire SPI bus:

#define PIN_MISO 16
#define PIN_CS   17
#define PIN_SCK  18
#define PIN_MOSI 19


Over SPI, the Pico 2W:

Requests temperature data from the slave

Receives:

Raw ADC readings

Calculated temperature

LED on/off states

Sends commands to:

Toggle LEDs on the slave

Request updated sensor values

The SPI protocol is application-specific and optimized for short, deterministic transactions.

2. Wi-Fi + HTTP Server (Pico 2W ↔ Web Client)

Using its onboard Wi-Fi, the Pico 2W hosts a lightweight HTTP server that allows a browser or external web app to:

Query system status

Send LED control commands

Send text messages intended for display on the ESP32 OLED

Example endpoints:

GET  /api/status
POST /api/led
POST /api/text


The web client never communicates directly with the ESP32. All requests are terminated at the Pico 2W.

3. Wi-Fi HTTP Client (Pico 2W → ESP32)

When the Pico 2W receives a request that targets the ESP32 (such as /api/text), it:

Parses the incoming HTTP request

Extracts and validates the JSON payload

Builds a new HTTP POST request

Sends that request to the ESP32 over Wi-Fi

Example forwarded request:

POST /api/text HTTP/1.1
Host: 192.168.1.248
Content-Type: application/json
Content-Length: 14

{"text":"Hello"}


This HTTP-to-HTTP forwarding is a key design feature of the project.

Multicore Design

The Pico 2W uses both RP2040 cores to keep timing-critical and network tasks isolated.

Core 0 — Real-Time & SPI

Core 0 is responsible for:

SPI master transactions

GPIO control

Timing-sensitive logic

Maintaining coherent system state

This core guarantees deterministic communication with the Pico slave.

Core 1 — Wi-Fi & Networking

Core 1 handles:

Wi-Fi stack

HTTP server

HTTP client requests to the ESP32

JSON parsing and construction

Separating networking from SPI ensures that Wi-Fi latency never disrupts real-time SPI communication.

Communication Topology
Web Client
    │
    │  Wi-Fi (HTTP)
    ▼
Pico 2W
 ├── SPI (Master)
 │     ▼
 │   Pico Slave
 │
 └── Wi-Fi (HTTP Client)
       ▼
     ESP32-S3-WROOM


Additional wiring:

The Pico 2W controls one GPIO line to the ESP32 for simple signaling.

The ESP32 does not send data back to the Pico 2W in the current design.

Programming Environment

Language: C

SDK: Raspberry Pi Pico SDK

IDE: VS Code

Networking: lwIP

JSON: cJSON

Build System: CMake

Key Design Features

Clean separation between wired and wireless domains

Deterministic SPI timing unaffected by Wi-Fi

HTTP protocol used end-to-end (web → Pico → ESP32)

Pico 2W as a transparent message broker

Easily extensible API surface

Summary

The Pico 2W is the heart of the system. It:

Speaks SPI to hardware

Speaks HTTP to humans

Speaks Wi-Fi to displays

By centralizing all communication in one device, the system remains modular, debuggable, and easy to extend.
