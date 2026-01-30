Web Client
Overview

The Web Client is the user-facing interface for the three-microcontroller system. It allows a user to view system status and send commands (such as text messages or LED control requests) using a standard web browser.

A key architectural feature of this project is that the web client never communicates directly with the ESP32. Instead, all web requests are routed through the Pico 2W, which acts as the central network gateway and traffic controller for the entire system.

This design ensures:

A single, well-defined network endpoint

Clear separation of responsibilities

Easier debugging and extensibility

Reduced attack surface for embedded devices

Role in the System

The Web Client:

Runs in a browser (desktop or mobile)

Sends HTTP requests to the Pico 2W

Displays system status returned by the Pico 2W

Allows the user to:

Send text messages to be displayed on the ESP32 OLED

Control LED states on the Pico slave

Query system health and status

The Web Client is stateless and does not maintain persistent connections to any microcontroller.

Communication Model

All communication follows this flow:

Web Browser
     ↓ HTTP (GET / POST)
Pico 2W  (central controller)
     ↓
 ├── SPI → Pico (slave)
 └── Wi-Fi → ESP32-S3

Important Design Rule

The Web Client never talks directly to the ESP32.

Even though the ESP32 exposes its own REST API (e.g. /api/status, /api/text), the web client always targets the Pico 2W. The Pico 2W then forwards or translates requests as needed.

Supported Endpoints (as seen by the Web Client)

All URLs below are served by the Pico 2W.

GET /api/status

Returns a JSON object describing the current system state, including:

Raw ADC value from the Pico slave

Temperature (derived from thermistor)

LED on/off states

Timestamp or uptime

Example response:

{
  "raw": 2381,
  "temperature": 61,
  "led": 0,
  "timestamp": 45058
}

POST /api/text

Sends a text message through the Pico 2W to the ESP32, which displays it on the OLED.

Example request body:

{
  "text": "Hello World"
}


Flow:

Browser → Pico 2W → ESP32 → OLED

POST /api/cmd (if enabled)

Used to send structured commands (e.g. LED control, temperature queries).

Example:

{
  "led": 1
}


The Pico 2W translates this into SPI commands for the Pico slave.

Technologies Used

HTML5

CSS

JavaScript (vanilla or minimal framework)

Fetch API / XMLHttpRequest

JSON over HTTP

No backend server is required beyond the Pico 2W itself.

Deployment

The web client can be:

Served directly by the Pico 2W

Hosted on an external web server on the same LAN

Opened locally in a browser (as long as CORS rules allow it)

Regardless of hosting location, the Pico 2W is always the HTTP target.

Why This Design Matters

This architecture demonstrates several important embedded-systems concepts:

Gateway pattern: One microcontroller manages all external communication

Protocol translation: HTTP ⇄ SPI ⇄ GPIO ⇄ Wi-Fi

Security and simplicity: Devices are not exposed unnecessarily

Scalability: Additional devices can be added without changing the web client

The Web Client remains simple, portable, and hardware-agnostic.

Summary

The Web Client is the user interface for the system

It communicates only with the Pico 2W

The Pico 2W routes messages to:

The Pico slave (SPI)

The ESP32 (Wi-Fi)

The client uses standard HTTP + JSON

No direct ESP32 access is required or expected

***
The Pico folder is a web folder. It contains PHP, HTML, CSS, and JS files. In the inc/setup.php file, you will have to specify a MySQL database named pico with a table named log. For simplicity's sake I named the username: pico, password: pico, localhost.
The log table: id = int 10 primary key auto-increment; raw = int(5); temperature = int(3); led = int(2); timestampt = timestamp.

Also in the setup.php file:
There are two IP addresses involved in this project: that of the Pico 2W (master), and that of the ESP32-S3-WROOM. The Pico 2W transacts both ways with the website, and the Pico 2W and ESP32-S3-WROOM transact with each other, both ways, all over Wi-Fi.
But, the website ONLY talks to the Pico 2W. The website could be coded, of course, to transact with the ESP32, but that's not part of the current project.
The website makes GET requests to the Pico 2W for raw ADC data and the LED state. These come from the Pico slave. These transactions are effected using short JSON strings that look like this:

{"raw":2381,"temperature":61,"led":0,"timestamp":45058}

This load balancing is key to a stable web platform on the Pico 2W. Let the Pico 2W do what it was made for, getting and sending data to peripheral electronics, and let a dedicated web server make sense of it all.

The Pico 2W communicates with the ESP32 over Wi-Fi, so the website doesn't have to. This makes the Pico 2W the middle man, or the station dispatcher. It receives requests and commands form the web server, then relays those through the appropriate channels to the Pico via SPI bus, or to the ESP32 via Wi-Fi.

Known to work using XAMPP - Apache, PHP, MySQL.

For the website to work requires the appropriate hardware steps be taken, including the setup of the Pico 2W, Pico, and ESP32-S3-WROOM. Equally important is the setup of an external thermistor on the Pico, as well as the LED output pins. These are all available in the C source code for each separate project, master, slave, and esp32.


