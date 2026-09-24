## astb-pbm

> Local trainer for the ASTB-E Performance Based Measures battery (stick, throttle, dichotic listening) on this MacBook Air with a Saitek/Logitech X-52 HOTAS and split headphones.

# ASTB-PBM practice console

Local trainer for the ASTB-E Performance Based Measures battery (stick, throttle, dichotic listening) on this MacBook Air with a Saitek/Logitech X-52 HOTAS and split headphones.

Not affiliated with NAMI / Navy Medicine. Practice scores are time-on-target and hit/false-alarm rates, not official PFAR/FOFAR.

## What this is

Published PBM sequence (Draheim et al. 2025, Phillips 2011, Walker 2007):

1. Dichotic listening (attend L, then R) — even number in attended ear → stick trigger; odd → throttle thumb; ignore letters and the other ear
2. VTT — throttle tracks a yellow aircraft on the left vertical lane
3. ATT — stick tracks a yellow aircraft in 2D; Y inverted (forward = cursor down)
4. AVTT — both at once
5. Multitrack — AVTT + DLT
6. Emergency stack — AVTT + DLT + T1–T6 spoken emergencies

Speed steps and “redirect if you are tracking well” are implemented. Official scoring weights are proprietary and are **not** reproduced.

## Run

```bash
cd ~/astb-pbm
./serve.sh          # http://127.0.0.1:8765
npm test            # logic + asset checks
python3 scripts/generate-audio.py   # regenerate spoken tokens
```

Open in **Chrome** (best Gamepad + `setSinkId` support). First screen after load: Hardware bench.

## Hardware

- X-52 via USB-A→C adapter. Allow the accessory if macOS asks. Click the page so the Gamepad API attaches.
- Split headset as the Mac’s output (bench “Choose headphones”).
- Keyboard fallback: WASD/arrows stick, Q/E or R/F throttle, Space = even, Left Shift = odd, 1–6 = T1–T6.

Default axis guess (Chrome / Mac X-52, HID usage index): AX0 roll, AX1 pitch, **AX2 throttle lever (inverted — push forward raises the VTT pip)**, AX3 Rx thumb rotary (never used for VTT), B0 trigger, B6 throttle D. Load + bench mapping pin VTT to AX2 with invert; they cannot rebind it to the thumb wheel.

## Layout

- `src/pbm-core.mjs` — scoring, motion, DLT rules (Node-tested)
- `src/controls.mjs` — X-52 mapping + keyboard
- `src/audio.js` — Web Audio, hard L/R pans, simultaneous pairs
- `src/render.js` — canvas (yellow aircraft, red reticle)
- `src/app.js` — views, battery, hardware wizard
- `assets/audio/` — `say`-generated tokens (Samantha, 22.05 kHz mono WAV)

## Decisions

- Built native instead of buying ASTB Prep / TBAS Study Pro so the hardware test path is inspectable and the X-52/headset can be verified without a paywall.
- Position control with ~50 ms lag (over-correction oscillates — matches the published coaching point).
- True stereo isolation via ChannelMerger (not speechSynthesis, which cannot lock a token to one cup while another token plays in the other).

---
> Source: [stefanopineda/astb-pbm](https://github.com/stefanopineda/astb-pbm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
