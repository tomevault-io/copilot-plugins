## arcade

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Raspberry Pi 5 that drives a 128×128 RGB LED matrix (two 64×64 HUB75 panels on an Adafruit Matrix Bonnet). A `controller` daemon manages which mode is active and watches physical GPIO buttons for input.

**The code cannot run on macOS.** `adafruit_blinka_raspberry_pi5_piomatter` is Pi 5-specific hardware. All testing and execution happens on the Pi itself.

## Running on the Pi

```bash
# Activate the venv first (always required)
source ~/.venvs/blinka_venv/bin/activate

# Run the controller directly (foreground, Ctrl-C to stop)
python controller.py

# Run a single mode directly (bypasses controller)
python modes/art.py
python modes/art.py --dir ~/wallart/xmas-gifs
python modes/art.py --file ~/wallart/butterfly-gifs/butterfly.gif   # single-GIF loop

# Manage via systemd (after install)
sudo systemctl status arcade
sudo systemctl restart arcade
journalctl -u arcade -f       # live logs

# Force a single mode via systemd (stops the button controller)
sudo systemctl start arcade@game     # or wallart / christmas / butterflies / butterfly
sudo systemctl start arcade          # hand control back to the buttons
sudo scripts/arcade-mode.sh wallart  # switch cleanly between two forced modes
```

## Deploy / install

```bash
cp .env.example .env          # then edit .env for your paths
bash scripts/install.sh       # renders service template, enables + starts it
```

`scripts/install.sh` uses `sed` to substitute `__ARCADE_DIR__` and `__ARCADE_USER__` in **both** `systemd/arcade.service` and `systemd/arcade@.service`, so **do not hardcode paths in either template**.

## Architecture

The system has three layers:

**1. `controller.py` — the daemon**
Reads all config from environment variables at startup (see `.env.example`). Holds a `ModeController` instance that keeps one subprocess alive at a time. `gpiozero` button callbacks call `ModeController.switch_to()` from a background thread — the method is protected by a `threading.Lock`. If the active mode process exits unexpectedly, the main loop restarts it.

**2. Mode scripts — each runs as a subprocess until killed**
- `modes/art.py` — GIF player. `--dir` plays every GIF in a directory in random order (`wallart`, `christmas`, `butterflies` modes each pass a different `--dir`). `--file` loops a single GIF forever (`butterfly` mode). The value is baked into the `MODES` dict in `controller.py`.
- `virtualdisplay.py` — Starts an Xvfb virtual display, launches PICO-8 inside it, and mirrors each grabbed frame to the matrix. Receives its full config as CLI args from `controller.py`.

**3. Config (`MODES` dict in `controller.py`)**
Each named mode is just a `list[str]` command. Adding a new mode is: add an entry to `GIF_DIRS` + `MODES` + `BUTTON_MAP` + a button pin env var. No other files need to change — the `arcade@<mode>` systemd unit works automatically for any `MODES` key.

**Two ways to select a mode.** Normally the button controller (`arcade.service`, running `controller.py`) owns the matrix and switches modes on GPIO presses. `controller.py --run-mode <name>` instead `exec`s a single mode directly (no GPIO, no restart loop); the `arcade@.service` template uses this so `systemctl start arcade@<mode>` can force a mode. Only one process may drive the matrix at once — `arcade@.service` has `Conflicts=arcade.service` (symmetric), so starting either side stops the other. `scripts/run.sh` dispatches: no arg → controller, one arg → that mode.

## Adding a new GIF category mode

1. Add to `GIF_DIRS` in `controller.py`:
   ```python
   "space": os.environ.get("ARCADE_GIF_DIR_SPACE", f"{_WALLART_BASE}/space-gifs"),
   ```
2. Add to `MODES`:
   ```python
   "space": [PYTHON, _ART, "--dir", GIF_DIRS["space"]],
   ```
3. Add a pin variable and entry in `BUTTON_MAP`:
   ```python
   BTN_SPACE_PIN = int(os.environ.get("ARCADE_BTN_SPACE_PIN", "0"))
   # ... in BUTTON_MAP list:
   (BTN_SPACE_PIN, "space"),
   ```
4. Add `ARCADE_GIF_DIR_SPACE` and `ARCADE_BTN_SPACE_PIN` to `.env.example`.

## Key hardware details

- **Matrix**: 128×128px, `n_addr_lines=5`, `piomatter.Pinout.AdafruitMatrixBonnet`
- **Serpentine layout**: panels are connected in serpentine order (set in virtualdisplay.py args)
- **Pi 5 GPIO**: uses `lgpio` as the gpiozero pin factory (`GPIOZERO_PIN_FACTORY=lgpio`). Direct GPIO calls will fail without this.
- **Pi 5 power button**: the J2 header on the board handles graceful shutdown natively — no software needed for that specific button.

## Logs

| File | Written by |
|---|---|
| `$ARCADE_LOG_DIR/controller.log` | `controller.py` |
| `$ARCADE_LOG_DIR/art.log` | `modes/art.py` |

---
> Source: [jlhester/arcade](https://github.com/jlhester/arcade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
