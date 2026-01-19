# ESPHome Somfy 535A2 Roll-Up Door Controller

## Overview

This repository contains a working ESPHome configuration for controlling a Somfy 535A2 RS-485 motor using an ESP32 and XY-485 converter.

## Hardware Requirements

- **ESP32** (esp32dev board)
- **XY-485 RS-485 Converter**
- **Somfy 535A2 RS-485 Motor**

## Wiring Diagram

```
ESP32          XY-485          Motor
----------------------------------------
GPIO16 (TX) →  TX
GPIO17 (RX) →  RX
GND         →  GND           GND
               A+   →   A+
               B-   →   B-
```

**Note:** The XY-485 converter automatically handles the RS-485 DE/RE (Driver Enable/Receive Enable) signals, so no manual control pin is needed.

## ESPHome Configuration Location

The ESPHome dashboard is accessible at:

**http://10.20.1.1:6052**

### ESPHome Dashboard Credentials

- **OTA Password:** `517palmdr`
- **Device Static IP:** `10.20.1.240`

## UART Configuration

The motor uses the following serial communication settings:

- **Baud Rate:** 4800
- **Data Bits:** 8
- **Parity:** ODD
- **Stop Bits:** 1

**Important:** Technical support may state 9600 baud with no parity, but the working configuration is **4800 baud with ODD parity**.

## Working Commands

### UP Command (14 bytes)
```
FC 70 08 00 02 00 F5 1F F3 FE FF FF 00 06 79
```

### DOWN Command (14 bytes)
```
FC 70 08 00 02 00 F5 1F F3 FF FF FF 00 06 7A
```

### STOP Command (12 bytes)
```
FD 73 08 00 02 00 F5 1F F3 00 03 81
```

## Home Assistant Integration

The device appears as a "Roll-Up Door" cover entity with UP/DOWN/STOP controls. The configuration uses optimistic mode (no position feedback from the motor).

## Installation

1. Install ESPHome:
   ```bash
   pip install esphome
   ```

2. Compile the configuration:
   ```bash
   esphome compile ESPHome_Rollup_Door/esphome_rollup_door_config.yaml
   ```

3. Upload to ESP32 via OTA:
   ```bash
   esphome upload ESPHome_Rollup_Door/esphome_rollup_door_config.yaml
   ```

## Troubleshooting

If the motor doesn't respond:
1. Verify UART settings (4800, ODD, 8N1)
2. Check wiring (A+/B-/GND connections)
3. Ensure XY-485 converter is powered
4. Verify motor is in RS-485 mode

## Project Structure

```
517-palm-dr-ESP32/
├── ESPHome_Rollup_Door/
│   └── esphome_rollup_door_config.yaml
└── README.md
```

## Notes

- Motor NodeID (from label): `0CDD0E` (not needed in commands)
- No RS-485 termination resistor required for short cable runs (< 10ft)
- Commands work as broadcast (no specific addressing needed)
