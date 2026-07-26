## tiktokautouploader

> This file provides context for AI coding agents working on this repository.

# AGENT.md — tiktokautouploader

This file provides context for AI coding agents working on this repository.

---

## Project Overview

`tiktokautouploader` is a Python library that automates uploading and scheduling videos to TikTok via browser automation. It uses [Phantomwright](https://pypi.org/project/phantomwright/) (a patched Playwright fork) as its browser engine for bot-detection evasion, and calls into Node.js/JavaScript assets for the initial login flow.

**PyPI package:** `tiktokautouploader`  
**Primary entry point:** `tiktokautouploader/function.py` → `upload_tiktok()`  
**Public API surface:** `tiktokautouploader/__init__.py` (exports only `upload_tiktok`)

---

## Repository Structure

```
tiktokautouploader/
├── __init__.py               # Exports upload_tiktok
├── function.py               # Core upload logic — all Python automation lives here
└── Js_assets/
    ├── login.js              # Node.js script: opens browser for first-time login & saves cookies
    └── package.json          # JS deps: playwright, playwright-extra, puppeteer-extra-plugin-stealth

TelegramAutomation/
├── Fancy_Upload.py           # Standalone community script: folder-based uploads with Telegram bot control
└── README.md

README.md
DOCUMENTATION.md              # Full parameter reference for upload_tiktok()
```

---

## Setup & Installation

### Python dependencies
```bash
pip install tiktokautouploader
```

All Python deps (`phantomwright`, `requests`, `Pillow`, `inference`) install automatically.

### Browser binaries (required once after install)
```bash
phantomwright_driver install chromium
```

### Node.js (required)
Node.js must be installed and `npm` must be on PATH. The JS dependencies (`playwright`, `playwright-extra`, `puppeteer-extra-plugin-stealth`) self-install on first function call — no manual step needed.

---

## Key Behaviours Agents Must Know

### Cookie-based authentication
- On **first use per account**, the library spawns a visible browser via `login.js` and prompts the user to log in manually.
- Cookies are saved to `TK_cookies_<accountname>.json` in the **current working directory**.
- On subsequent runs, cookies are read from that file. Expired cookies trigger re-login automatically.
- Never delete or move cookie files during a session.

### JS asset auto-install
- `install_js_dependencies()` checks for `Js_assets/node_modules/` and runs `npm install` if absent.
- This adds ~20–30 seconds on the very first run only.

### Captcha solving
- Captchas are solved automatically via Roboflow inference (`inference_sdk`).
- Two captcha types are supported (see `DOCUMENTATION.md`).
- The solver downloads a temporary `captcha_image.jpg` to the working directory and removes it after solving.

### Error exits
- The codebase uses `sys.exit()` extensively for hard failures (bad proxy, unsolvable captcha, upload timeout, etc.).
- When adding new failure paths, prefer `sys.exit("DESCRIPTIVE MESSAGE")` over raising exceptions, to stay consistent with the existing pattern.

---

## Core Function Signature

```python
upload_tiktok(
    video: str,           # Path to video file
    description: str,     # Caption text (no hashtags here)
    accountname: str,     # Account name — determines which cookie file is used
    hashtags=None,        # List of hashtag strings e.g. ['#fun', '#viral']
    sound_name=None,      # TikTok sound name to search for or find in favorites
    sound_aud_vol='mix',  # 'mix' | 'main' | 'background'
    schedule=None,        # 'HH:MM' in local time, minute must be multiple of 5
    day=None,             # Day-of-month integer (requires schedule)
    copyrightcheck=False,
    suppressprint=False,
    headless=True,
    stealth=False,        # Extra delays on top of Phantomwright's always-on evasion
    proxy=None,           # {'server': '...', 'username': '...', 'password': '...'}
    search_mode='search', # 'search' | 'favorites'
) -> str                  # Returns 'Completed' or 'Error'
```

---

## Internal Architecture

`function.py` is decomposed into focused private helpers — do not inline them back into `upload_tiktok()`:

| Helper | Responsibility |
|---|---|
| `_load_or_create_cookies()` | Read cookies file or trigger login flow |
| `_make_stealth_context()` | Launch Phantomwright browser + apply stealth |
| `_goto_with_retry()` | Navigate with 2-attempt retry |
| `_wait_for_upload_or_captcha()` | Poll until upload UI or captcha appears |
| `_solve_captcha_if_needed()` | Roboflow-based captcha solver |
| `_set_video_input()` | File input injection |
| `_add_description_and_hashtags()` | Type description + click hashtag suggestions |
| `_wait_for_upload_ready()` | Wait for TikTok to finish processing the video |
| `_apply_schedule()` | Interact with date/time picker |
| `_add_sound_from_upload_page()` | Open Sounds panel, search/select, adjust volume |
| `_run_upload_copyright_check()` | Toggle copyright check and wait for result |
| `_submit_upload()` | Click Post/Schedule and confirm success |

Selector constants for all major UI elements are defined at the top of `function.py` as module-level strings. When TikTok's UI changes, update these constants first.

---

## CSS Selectors & Fragility

The automation relies on TikTok Studio's DOM structure. These are the most likely breakage points:

- `CAPTCHA_*` selectors — captcha UI changes frequently
- `SCHEDULE_DAY_ICON_SELECTOR` / `SCHEDULE_TIME_ICON_SELECTOR` — SVG path-based selectors, will break on icon updates
- `SOUND_VOLUME_ICON_*` selectors — base64 SVG `src` attributes, highly fragile
- `DRAFT_EDIT_ICON_SELECTOR` — SVG path-based

When fixing broken selectors, set `headless=False` to observe the live DOM and update the relevant constant at the top of `function.py`.

---

## Adding New Features

- New upload options should be added as optional keyword arguments to `upload_tiktok()` with safe defaults.
- New browser interaction steps belong in their own `_helper_function()` following the existing decomposition pattern.
- If a feature requires JS, add it to `Js_assets/` and invoke via `subprocess.run(["node", ...])`.
- Update `DOCUMENTATION.md` and the parameter table in `README.md` for any new parameters.

---

## Telegram Automation (`TelegramAutomation/`)

`Fancy_Upload.py` is a **standalone community script**, not part of the library. It imports `upload_tiktok` from the installed package. Do not modify `function.py` to accommodate Telegram-specific behaviour — keep concerns separated.

---

## Do Not

- Do not add `print()` statements outside the `suppressprint` guard (`if not suppressprint: print(...)`)
- Do not change `sys.exit()` error handling to exceptions without updating all callers
- Do not store state between calls — `upload_tiktok()` is stateless by design
- Do not hardcode account names, cookie paths, or video paths in library code
- Do not commit `TK_cookies_*.json` files — they contain session credentials

---
> Source: [haziq-exe/TikTokAutoUploader](https://github.com/haziq-exe/TikTokAutoUploader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
