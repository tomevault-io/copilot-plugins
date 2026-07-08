## arduinoclock

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Arduino Mega 2560 digital clock with alarm. Single sketch: `ArduinoClock.ino`.

**Hardware:**
- RTC: DS3231 (I²C on pins 20/21, battery-backed, ±2 ppm TCXO)
- Display: MAX7219 8-digit 7-segment module (SPI-like on pins 51/52/53)
- Buttons: 4× momentary, active LOW with internal pull-up (pins 2–5)
- Buzzer: active buzzer on pin 6

**Required libraries** (Arduino Library Manager):
- `LedControl` by Eberhard Fahle
- `RTClib` by Adafruit

## Building and uploading

There is no automated build. Compile and upload via the Arduino IDE or `arduino-cli`:

```bash
# Compile
arduino-cli compile --fqbn arduino:avr:mega ArduinoClock.ino

# Upload (adjust port as needed — usually COM3/COM4 on Windows)
arduino-cli upload  --fqbn arduino:avr:mega --port COM3 ArduinoClock.ino

# Compile + upload in one step
arduino-cli compile --fqbn arduino:avr:mega --upload --port COM3 ArduinoClock.ino
```

## Architecture

The sketch is a single-file cooperative state machine — no RTOS, no blocking `delay()` in the main loop. Everything runs off `millis()`.

### Main loop call order (every iteration)
```
loop() → pollButtons() → handleInput() → updateBuzzer() → refreshDisplay()
                                              ↑
                                        checkAlarm() is called here
```

### State machine (`ClockMode` enum)
Six modes advance linearly via `BTN_MODE`. Each mode determines which display face is shown and which field `adjustField()` modifies.

```
MODE_NORMAL → MODE_SET_HH → MODE_SET_MM → MODE_SET_SS
           → MODE_SET_ALHH → MODE_SET_ALMM → (saveAndExit → MODE_NORMAL)
```

`saveAndExit()` is the only place that writes to the DS3231. It preserves the current calendar date and only updates H/M/S from the edit buffer (`gEditH`, `gEditM`, `gEditS`).

### Display layer
All display writes go through four logical-position helpers (`writeDigit`, `writeDash`, `writeChar`, `writeBlank`). Logical position 0 is always the left-most digit. `physPos()` translates to the LedControl index; flip `DIGIT_REVERSED` if a module has inverted digit order.

Two display "faces" assemble a full 8-digit frame each loop:
- `showTimeFace()` — normal and time-edit modes (`HH-MM-SS`)
- `showAlarmFace()` — alarm-edit modes (`A  HH-MM `)

Blinking is driven by `gBlinkOn` (toggled by `refreshDisplay` every `BLINK_MS`). A face passes `gBlinkOn` as the `show` flag for the field being edited; when `false` the helper writes blanks instead of digits.

### Button debounce
`pollButtons()` uses a per-button edge-restart timer. A `fired` flag is set on the first stable LOW transition and consumed exactly once by `handleInput()`.

### Alarm trigger
`checkAlarm()` fires when `hour == gAlmH && minute == gAlmM && second == 0`. The buzzer beep pattern (300 ms ON / 200 ms OFF) is computed inside `updateBuzzer()` using `(elapsed % 500)`, so no state or timer is needed beyond `gRingStart`.

## Git workflow

After every meaningful change: commit locally with a conventional commit message (`feat:`, `fix:`, `refactor:`) and push to `origin/master`.

```bash
git add ArduinoClock.ino   # (or other changed files)
git commit -m "fix: ..."
git push
```

Remote: `https://github.com/Anqv/ArduinoClock`  
`gh` CLI is installed at `C:\Program Files\GitHub CLI\gh.exe` and authenticated.

## Key tuneable constants

| Constant | Default | Purpose |
|---|---|---|
| `BRIGHTNESS` | 10 | MAX7219 display brightness (0–15) |
| `DIGIT_REVERSED` | false | Flip if digits appear left↔right reversed |
| `DEBOUNCE_MS` | 50 | Button debounce window |
| `BLINK_MS` | 500 | Edit-mode blink half-period |
| `INACTIVITY_MS` | 10 000 | Auto-save timeout in settings |
| `ALARM_MAX_MS` | 60 000 | Alarm auto-stop duration |

---
> Source: [Anqv/ArduinoClock](https://github.com/Anqv/ArduinoClock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
