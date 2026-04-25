## hushbell-poc

> Dog-friendly smart doorbell -- laptop-first proof of concept.

# HushBell POC

Dog-friendly smart doorbell -- laptop-first proof of concept.
Thomas Frumkin's design from the AI Craftspeople Guild.

## Architecture

- `src/hushbell/controller.py` -- Main orchestrator (ring lifecycle)
- `src/hushbell/audio_engine.py` -- 40Hz + variable-frequency tone generation with envelopes
- `src/hushbell/pleasant_tones.py` -- Harmonic layering + vibrato for warm tones
- `src/hushbell/visual_engine.py` -- WS2812B LED strip simulation (pygame)
- `src/hushbell/notification.py` -- Platform-specific notification facade
- `src/hushbell/mqtt_bridge.py` -- MQTT pub/sub (paho-mqtt)
- `src/hushbell/battery_sim.py` -- 1200 rings/charge state machine
- `src/hushbell/config.py` -- Pydantic settings with frequency strategy config
- `src/hushbell/spectrum.py` -- Real-time FFT visualiser with dynamic markers
- `src/hushbell/triggers/` -- Input sources (keyboard, MQTT, HTTP)
- `tests/` -- Pytest suite with FFT verification (87 tests)
- `web/` -- Browser demo (Web Audio API, frequency controls)

## Key Parameters

- Primary frequency: 40Hz (below 67Hz dog hearing floor)
- Secondary frequency: configurable (defeats classical conditioning)
- Frequency modes: fixed, random, preset, vagal
- Fade-in: ALWAYS applied (anti-startle safety guarantee)
- Envelope types: linear, sine, exponential
- Battery: 1200 rings/charge (simulated)
- MQTT topics: hushbell/ring, hushbell/status, hushbell/battery, hushbell/config, hushbell/config/state
- Visual: Warm amber (#FFBF00), WS2812B simulation

## Frequency Modes (Anti-Conditioning)

| Mode | Behaviour | Use Case |
|------|-----------|----------|
| `fixed` | Original 2000Hz (default) | Backwards compatible |
| `random` | Uniform random 800-3500Hz | Maximum anti-conditioning |
| `preset` | Rotate through user list | Curated pleasant tones |
| `vagal` | Gaussian ~900Hz +/- 200Hz | Vagal nerve resonance (pleasant) |

## Commands

```bash
python -m hushbell                          # Interactive (fixed mode)
python -m hushbell --freq-mode random       # Random frequency per ring
python -m hushbell --freq-mode vagal        # Pleasant vagal range
python -m hushbell --freq-mode preset --freq-presets 800,1000,1500
python -m hushbell --envelope sine          # Smoother fade-in curve
python -m hushbell --pleasant               # Harmonic layering + vibrato
python -m hushbell --web                    # Start HTTP trigger server
python -m hushbell --test --spectrum        # Single ring with FFT display
PYTHONPATH=src pytest tests/                # Run tests (87 tests)

# MQTT runtime config (publish to broker):
mosquitto_pub -t hushbell/config -m '{"frequency_mode": "vagal", "pleasant": true}'
mosquitto_pub -t hushbell/config -m '{"envelope_type": "sine"}'
```

## Principles

KISS, YAGNI, DRY. Laptop-first, cross-platform later.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CodeTonight-SA) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
