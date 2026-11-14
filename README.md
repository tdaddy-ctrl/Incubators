# DVC Pond — DIY Bacterial Incubator

A low-cost incubator project built around an Arduino Nano. The sketch reads a TMP36 temperature sensor, switches a heater through a relay, circulates air with a fan, and displays the current and target temperatures on an OLED. A rotary encoder adjusts the target temperature.

The design is intended to make it easier to run supervised experiments at different temperatures in parallel. It is a DIY project, not a certified laboratory incubator; validate its temperature performance for your use before relying on it.

## How It Works

1. The TMP36 measures the temperature inside the insulated chamber.
2. The Arduino Nano averages 20 sensor readings and converts the result to Celsius.
3. The Nano compares the measured temperature with the target and controls the heater relay.
4. A fan circulates air to help reduce temperature differences within the chamber.
5. The OLED displays the measured temperature, target temperature, heater state, and DVC Pond logo.
6. Turning the rotary encoder changes the target in 0.5°C increments.

## Components

| Component | Purpose and notes |
|---|---|
| Arduino Nano (ATmega328P-compatible) | Reads the sensor, controls the relay, and runs the display and encoder. |
| TMP36 temperature sensor | Analog temperature sensor; no separate library is required. |
| 5V, single-channel relay module | Switches the heater. The sketch assumes an active-high input: `HIGH` turns the heater on. |
| Two heating pads | Provide heat; the proposed build connects them in parallel. Confirm their voltage and current ratings before wiring. |
| Regulated 12V DC, 3A power supply | Proposed supply for the heating pads and fan. |
| Buck converter (12V to 5V) | Provides regulated 5V for the Nano and logic components. Set and verify its output before connecting the Nano. |
| 128×64 I2C OLED, SSD1306-compatible | Displays temperature and status; the sketch uses I2C address `0x3C` and the Adafruit SSD1306 library. |
| Rotary encoder | Adjusts the target temperature. CLK must connect to D3 for the sketch's interrupt. |
| Small 12V computer fan | Circulates air continuously. |
| Insulated box | Incubation chamber; the proposed internal dimensions are approximately 15 × 21 × 10 inches. |
| Aluminum foil reflector | Proposed between the heating pads and lid; keep it flat and clear of wiring. |

Component models, dimensions, and cost are build-specific. Check component datasheets and ratings before purchasing or wiring.

## Wiring Reference

| Arduino Nano pin | Connection |
|---|---|
| A0 | TMP36 analog output |
| D2 | Relay module input |
| D3 | Rotary encoder CLK (interrupt pin) |
| D4 | Rotary encoder DT |
| D5 | Rotary encoder SW; configured with a pull-up, but not used by the current sketch |
| A4 / A5 | I2C SDA / SCL for the OLED on a typical Nano |
| 5V / GND | Logic power and common ground, as required by the modules |

Follow the pin labels and datasheets for your exact modules. The sketch's relay logic expects an active-high relay input; verify the relay's behavior with the heater disconnected before use.

## Sketch Behavior

The Arduino sketch is in [`IncubatorWokeringCode#1`](./IncubatorWokeringCode%231).

- **Temperature:** Reads A0 twenty times per loop, averages the samples, converts ADC counts to voltage using a 5.0V reference, then applies the TMP36 formula `(voltage - 0.5) * 100`. `TEMP_OFFSET` is currently `0.0`; calibrate against a reliable thermometer if needed.
- **Target:** Starts at 37.0°C. Encoder movement changes it by 0.5°C per count, with limits of 20°C and 45°C.
- **Heater control:** Turns the relay on below `targetTemp - 1.0°C` and off at or above the target. A five-second minimum interval separates relay changes.
- **Display:** Refreshes every 100ms. The ripple animation updates faster while the heater is on.
- **Encoder:** Uses an interrupt on D3 and reads D4 to determine rotation direction. The push button on D5 is not implemented.

## Software

Install these libraries through the Arduino IDE Library Manager:

1. Adafruit GFX Library
2. Adafruit SSD1306

The sketch also uses the Arduino `Wire` library for I2C. Confirm your OLED is compatible with the SSD1306 library and configured for address `0x3C`.

## Safety and Validation

- Use a regulated, correctly rated low-voltage DC supply and verify the buck converter output before connecting electronics.
- Keep heater wiring within the ratings of the pads, relay, wiring, and power supply. Do not connect mains voltage to the Arduino or exposed project wiring.
- Keep reflective material, wiring, and other combustible materials away from hot surfaces; use the build under supervision.
- Verify that the relay switches in the expected direction with the heater disconnected, then test temperature readings and chamber uniformity with an independent thermometer.
- The sketch does not detect a disconnected or failed sensor, enforce an independent over-temperature cutoff, or certify temperature uniformity. Do not leave it unattended or use it for an application requiring certified laboratory equipment.
- Follow your institution's biosafety procedures for any cultures and dispose of materials appropriately.
