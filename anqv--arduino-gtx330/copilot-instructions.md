## arduino-gtx330

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Firmware and Air Manager instrument for a hardware replica of the **Garmin GTX330 Mode-C transponder**. An Arduino Mega 2560 drives a 256×64 SSD1322 OLED display and physical buttons/encoder. Sim Innovations **Air Manager 4.x** acts as the middleware between X-Plane and the Arduino over USB serial.

## Repository structure

```
Arduino_GTX330_test.ino          Arduino firmware (single sketch file)
GTX330_AirManager/
  instrument.lua                 Air Manager 4.x instrument script (Lua)
  manifest.lua                   Instrument metadata for Air Manager
```

## Building and flashing the Arduino sketch

There is no build script — use the Arduino IDE or Arduino CLI directly.

**Arduino IDE:**
1. Open `Arduino_GTX330_test.ino`
2. Board: **Arduino Mega 2560**
3. Install library: **U8g2** by olikraus (Library Manager)
4. Select the correct COM port → Upload

**Arduino CLI:**
```bash
arduino-cli compile --fqbn arduino:avr:mega Arduino_GTX330_test.ino
arduino-cli upload  --fqbn arduino:avr:mega --port COM<N> Arduino_GTX330_test.ino
```

There are no automated tests for the Arduino sketch.

## Air Manager instrument

The Lua files in `GTX330_AirManager/` are loaded by Air Manager 4.x at runtime — there is no separate compile or lint step. To reload after editing, press **Reload** in the Air Manager panel designer.

## Serial protocol (9600 baud)

All messages are newline-terminated (`\n`). This protocol is the contract between the two halves of the system; changes must be kept in sync across both files.

| Direction | Message | Meaning |
|---|---|---|
| AM → Arduino | `MODE:<OFF\|STBY\|ON\|ALT>` | Set transponder mode |
| AM → Arduino | `SQWK:<XXXX>` | Set squawk display |
| AM → Arduino | `UTC:<HH>:<MM>:<SS>` | Set UTC clock |
| AM → Arduino | `FLTM:<HH>:<MM>:<SS>` | Set flight time counter |
| AM → Arduino | `DISP:<FT\|UTC>` | Select time display |
| AM → Arduino | `IDENT:<0\|1>` | IDENT active flag |
| Arduino → AM | `READY` | Boot complete; triggers full state sync |
| Arduino → AM | `BTN:<name>` | Button press (OFF/STBY/ON/ALT/IDENT/VFR/FLTUTC/CLRFT) |
| Arduino → AM | `SQWK:<XXXX>` | Squawk changed by encoder |

## Key hardware pin assignments

| Pin(s) | Function |
|---|---|
| 47–51 | SSD1322 OLED (CS/DC/RES/SDA/SCK) — software SPI |
| 2, 3 | Rotary encoder A/B (hardware interrupts INT4/INT5 on Mega) |
| 4 | Encoder push button |
| 22–29 | Mode and function buttons (INPUT_PULLUP, active LOW) |

## Architecture notes

**Arduino firmware** maintains a local copy of all display state (mode, squawk, UTC, flight time). The local 1-second tick (`tickSecond()`) advances time smoothly between X-Plane updates. Air Manager is authoritative — when it sends a value, the Arduino overwrites its local state. When the Arduino boots it sends `READY` and Air Manager responds with a full push of all current X-Plane values.

**Squawk edit mode** is entirely local to the Arduino. Turning the encoder enters edit mode and highlights the active digit (inverted). The new squawk is committed to Air Manager (and therefore X-Plane) only when the user pushes the encoder button past digit 4, or after a 5-second inactivity timeout (`EDIT_TIMEOUT_MS`).

**Air Manager Lua** subscribes to four X-Plane datarefs and translates hardware button events (`BTN:*`) into dataref writes or commands. It uses a `cache` table to suppress re-sending unchanged values. The IDENT 18-second window is tracked by recording the target UTC second (`ident_end_utc`) and checking it on every UTC dataref callback — no timer API required.

**Display layout** (256×64):
- Left (x 1–57): mode box with REPLY/IDENT indicator
- Centre (x 62–185): large squawk digits (`u8g2_font_logisoso32_tn`), VFR/EMRG badge
- Right (x 189–255): FL TIME or UTC, GTX 330 label

## X-Plane datarefs

| Dataref | Type | Use |
|---|---|---|
| `sim/cockpit/radios/transponder_mode` | INT | 0=OFF 1=STBY 2=ON 3=ALT |
| `sim/cockpit/radios/transponder_code` | INT | Squawk as plain integer (e.g. 1200) |
| `sim/time/zulu_time_sec` | FLOAT | UTC seconds since midnight |
| `sim/time/total_flight_time_sec` | FLOAT | Flight time seconds |
| Command: `sim/transponder/transponder_ident` | — | Trigger IDENT |

---
> Source: [Anqv/Arduino-GTX330](https://github.com/Anqv/Arduino-GTX330) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
