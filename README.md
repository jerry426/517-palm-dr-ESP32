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

## Advanced Troubleshooting with Oscilloscope

If the motor doesn't respond despite correct commands and proper wiring, use an oscilloscope to verify the physical layer signals:

### 1. Verify Baud Rate

**Connect oscilloscope probe to RS-485 A or B line.**

Measure the width of a single bit:
- **4800 baud** = 208μs per bit (this is what works)
- **9600 baud** = 104μs per bit (what technical support incorrectly states)

**Compare waveforms:**
- Capture signal from installer configuration tool (baseline)
- Capture signal from ESPHome device
- Verify bit timing matches exactly

### 2. Check Parity Settings

Count bits per frame:
- Should be **10 bits total**: 8 data + 1 parity + 1 stop
- Verify parity bit makes total number of 1's **odd** (ODD parity)

### 3. Verify Signal Integrity

Check for:
- Clean square waves (not rounded or distorted)
- Proper voltage levels (RS-485 differential: ±1.5V to ±6V)
- No excessive noise or ringing
- Consistent timing across entire frame

### 4. Physical Layer vs Data Layer Analysis

| Layer | What to Check | Common Issues |
|-------|---------------|---------------|
| **Application** | Command bytes match documentation | Usually correct |
| **Data** | Hex sequences are valid | Usually correct |
| **Physical** | Baud rate, parity, timing | **Most common failure point** |

**This was the key breakthrough** - the sniffer showed identical bytes between installer and ESPHome, but the oscilloscope revealed the baud rate mismatch (9600 vs 4800).

### 5. Common Oscilloscope Settings

- **Timebase:** 100μs/div to 500μs/div
- **Vertical:** 2V/div (differential probe) or 1V/div (single-ended)
- **Trigger:** Rising edge on start bit
- **Capture:** Single frame to analyze individual bits

**Why this matters:** Even with perfect command bytes, mismatched physical layer settings (baud rate, parity) will cause the motor controller to reject or ignore the transmission entirely.
