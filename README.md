# smart_ro

**smart_ro** is an Arduino Nano-based reverse osmosis monitoring project for tracking drinking water flow, TDS, temperature, filter life, and Pure RO vs. remineralized water usage.

The project is designed for under-sink RO systems where water can be routed between plain RO water and remineralized water. It uses a small OLED dashboard, a real-time clock, EEPROM-backed usage storage, and sensor-driven diagnostics to give a practical view of system health and filter replacement timelines.

> Status: Work in progress / prototype
> Platform: Arduino Nano / ATmega328P

---

## Features

* Tracks product water flow using a Hall-effect flow sensor
* Estimates daily, weekly, and lifetime water usage
* Separates Pure RO and remineralized water usage
* Tracks filter life for:

  * Sediment / carbon pre-filters
  * RO membrane
  * Post-filter / remineralization stages
* Applies RO waste-water processing logic for pre-filter and membrane life estimates
* Reads TDS through an external analog TDS driver board
* Reads water temperature through the MH-01 temperature output
* Displays live system information on a 128x64 I2C OLED
* Uses a DS3231 real-time clock for date-aware usage tracking
* Stores usage and calibration data in EEPROM
* Includes button-based navigation and reset/calibration flows
* Supports OLED sleep/wake behavior to reduce display wear

---

## Hardware

Core components:

* Arduino Nano R3 / ATmega328P
* MH-01-G1-4 3-in-1 Flow + TDS + Temperature sensor
* 0.96" I2C OLED display, SSD1306, 128x64
* DS3231 RTC module
* External analog TDS driver board
* 10k potentiometer for manual valve-position sensing
* 3 momentary push buttons
* 10k resistor for temperature sensing, if required by the sensor wiring
* USB power supply
* RO tubing and fittings appropriate for the installation

Optional future hardware:

* Motorized 3-way valve
* Valve position feedback
* External 12V actuator power supply
* Motor driver / H-bridge / relay module, depending on valve type

---

## Current Pin Map

| Function                     | Arduino Nano Pin |
| ---------------------------- | ---------------: |
| Flow sensor pulse            |        D2 / INT0 |
| Left button                  |               D4 |
| Select button                |               D5 |
| Right button                 |               D6 |
| Valve position potentiometer |               A0 |
| TDS analog output            |               A1 |
| Temperature signal           |               A2 |
| OLED SDA + RTC SDA           |               A4 |
| OLED SCL + RTC SCL           |               A5 |
| 5V power                     |               5V |
| Ground                       |              GND |

---

## MH-01 Sensor Notes

The MH-01 sensor includes flow, TDS, and temperature functions, but the wiring should be verified against the specific seller harness before final assembly.

Known flow wiring from the seller diagram:

| Wire   | Function          |
| ------ | ----------------- |
| Red    | Flow sensor VCC   |
| Black  | Flow sensor GND   |
| Yellow | Flow pulse signal |

Flow formula:

```text
Hz = 36 × Q
Q = flow rate in L/min
2160 pulses = 1 liter
```

TDS wiring:

```text
MH-01 TDS red/blue electrode pair -> TDS driver board input
TDS driver board analog output -> Arduino A1
```

Do not connect the raw TDS electrode pair directly to an Arduino analog pin.

Temperature wiring is still installation-dependent. The green temperature wire may be a raw NTC thermistor signal or a conditioned analog output. Verify with a multimeter before finalizing the temperature circuit and firmware constants.

---

## Button Controls

The project uses three buttons:

| Button | Purpose                    |
| ------ | -------------------------- |
| Left   | Previous / decrease / back |
| Select | Select / advance / confirm |
| Right  | Next / increase            |

Typical behavior:

* Short press left/right to move between screens or values
* Short press select to choose or advance
* Long press select to confirm reset or save actions
* Long press left can be used as back/cancel behavior in settings flows

---

## Display Screens

The OLED UI is designed around quick glanceability.

Current screen areas include:

* Dashboard

  * Flow rate
  * Current mode: Pure RO or Remin
  * TDS
  * Water temperature
* Filter health

  * Remaining filter life
  * Estimated days remaining
* Diagnostics

  * Raw sensor readings
  * Valve position reading
  * Flow pulse state
* Settings

  * Reset filter stages
  * Calibrate remineralization valve closed position
  * Set date

---

## Filter-Life Logic

The flow sensor is installed after the RO storage tank, so it measures product water only.

For filter-life estimates:

```text
Pre-filters and RO membrane:
1 L product water = 4 L processed water estimate

Post-filter and remineralization stages:
1 L product water = 1 L filter usage
```

This means daily usage statistics remain based on actual drinking water dispensed, while pre-filter and membrane life calculations account for RO waste-water production.

---

## Firmware Setup

Recommended repo structure:

```text
smart_ro/
  firmware/
    smart_ro/
      smart_ro.ino
  docs/
    wiring.md
    images/
  hardware/
    enclosure/
    cad/
  README.md
  LICENSE
  .gitignore
```

The Arduino sketch should live in a folder with the same name as the `.ino` file:

```text
firmware/smart_ro/smart_ro.ino
```

---

## Required Arduino Libraries

Install these through the Arduino Library Manager:

* Adafruit GFX Library
* Adafruit SSD1306
* RTClib by Adafruit

The sketch also uses standard Arduino libraries:

* Wire
* EEPROM
* math

---

## Uploading to the Arduino Nano

In Arduino IDE:

1. Open `firmware/smart_ro/smart_ro.ino`
2. Select board: `Arduino Nano`
3. Select processor:

   * `ATmega328P`, or
   * `ATmega328P (Old Bootloader)` if upload fails
4. Select the correct serial port
5. Compile and upload

---

## Calibration

Before relying on readings, calibrate and verify:

1. Flow sensor

   * Confirm pulse output on D2
   * Validate measured volume against a known container

2. TDS

   * Confirm the MH-01 TDS electrode pair is connected to a proper TDS driver board
   * Calibrate against known TDS solution or a trusted TDS meter

3. Temperature

   * Test the green temperature wire with a multimeter
   * Confirm whether it behaves like a 10k NTC thermistor
   * Adjust firmware constants if needed

4. Valve position

   * Set the remineralization valve to fully closed
   * Run the valve calibration flow
   * Confirm that opening the valve changes the analog reading enough to detect remin mode

---

## Safety Notes

This is a hobbyist monitoring project, not a certified water-quality or plumbing safety device.

Before installing:

* Confirm all parts are compatible with potable water
* Use fittings rated for your RO system pressure
* Leak-test the system thoroughly
* Keep electronics protected from water
* Use proper strain relief on cables
* Do not power motors or valves directly from Arduino pins
* Use a separate power supply for any future motorized valve or actuator

---

## Roadmap

Potential future improvements:

* Motorized Pure RO / Remin valve switching
* Built-in valve position feedback
* Automatic flushing / maintenance reminders
* Better TDS calibration workflow
* Serial diagnostic mode
* Enclosure design
* PCB version
* Home Assistant or MQTT integration
* Data logging to SD card or cloud endpoint

---

## License

This project is intended to be open source under the MIT License.

See `LICENSE` for details.
