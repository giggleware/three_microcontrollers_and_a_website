Pico Slave (SPI Peripheral)
Overview

The Pico Slave is a Raspberry Pi Pico microcontroller that functions as a hardware SPI slave within the three-microcontroller system. Its primary responsibilities are:

Reading temperature data from an external thermistor (ADC)

Managing the state of multiple GPIO-controlled LEDs

Reporting sensor data and GPIO states to the Pico 2W (SPI master)

Responding deterministically to commands issued by the Pico 2W

The Pico Slave has no direct network connectivity and does not communicate with the web server or ESP32 directly. All external communication flows through the Pico 2W.

Role in the System Architecture

The Pico Slave occupies a low-level, real-time hardware role:

Thermistor + LEDs
        ↓
     Pico Slave
        ↓  (SPI)
     Pico 2W (Master)
        ↓  (Wi-Fi)
 Web Client / ESP32


Key characteristics:

Acts only as an SPI slave

Provides authoritative sensor readings

Executes GPIO state changes requested by the Pico 2W

Designed for deterministic timing and low latency

Inter-Microcontroller Communication
Communication with Pico 2W (SPI)

Bus type: SPI (4-wire)

Role: SPI slave

Direction: Full-duplex

Data exchanged:

Raw ADC values

Converted temperature readings

LED on/off states

Control commands from the master

The Pico Slave never initiates communication — it only responds to SPI transactions started by the Pico 2W.

Communication with ESP32

Direct communication: ❌ None

Indirect communication: ✅ Via Pico 2W

The Pico Slave does not speak Wi-Fi, HTTP, or JSON. Any interaction with the ESP32 originates from the Pico 2W, based on data previously supplied by the Pico Slave.

Dual-Core Design

The Pico Slave uses both RP2040 cores to cleanly separate time-critical hardware work from communication logic.

Core 0 — SPI Communication Core

Responsibilities:

SPI slave protocol handling

Receiving commands from the Pico 2W

Sending structured response data

Synchronization with Core 1

Why Core 0?

Keeps SPI timing predictable

Avoids delays caused by ADC sampling or GPIO toggling

Ensures reliable communication under load

Core 1 — Hardware & Control Core

Responsibilities:

Reading thermistor values via ADC

Converting ADC readings to temperature

Managing LED GPIO states

Maintaining the current system state

Why Core 1?

Isolates hardware sampling from SPI timing

Allows smooth periodic ADC reads

Ensures GPIO changes do not block communication

Sensor & GPIO Functions
Temperature Measurement

External thermistor connected to an ADC pin

Periodic ADC sampling

Conversion to temperature units

Values stored in shared memory for SPI access

LED Control

Multiple GPIO pins configured as outputs

LED states updated based on SPI commands

Current states included in SPI responses

Wiring
SPI Connections (Pico Slave ↔ Pico 2W)
Signal	Pico Slave GPIO	Pico 2W GPIO
MISO	GPIO 16	GPIO 16
CS	GPIO 17	GPIO 17
SCK	GPIO 18	GPIO 18
MOSI	GPIO 19	GPIO 19

Note: UART pins may appear in the codebase but are not used in the current design.

Additional Connections

Thermistor: Connected to an ADC-capable GPIO

LEDs: Connected to dedicated GPIO output pins

Power & Ground: Shared ground between all devices

Programming Environment

Language: C

SDK: Raspberry Pi Pico SDK

IDE: VS Code with Pico extension

Build System: CMake

Concurrency: Multicore RP2040 APIs

Design Philosophy

The Pico Slave is intentionally:

Simple

Deterministic

Hardware-focused

By offloading networking, parsing, and application logic to the Pico 2W and ESP32, the Pico Slave remains highly reliable and easy to reason about — an ideal embedded peripheral processor.

Summary

The Pico Slave:

Acts as a real-time sensor and GPIO controller

Communicates only via SPI with the Pico 2W

Uses both RP2040 cores for clean separation of concerns

Forms the hardware foundation of the overall system
