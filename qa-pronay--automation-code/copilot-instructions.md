## automation-code

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Selenium-based web automation suite for testing the **Tradeify trading platform** (https://staging.d3nmvgw29kqyqo.amplifyapp.com). It automates end-to-end checkout flows, user registration, and payment processing.

## Project Structure

```
automation/
├── CLAUDE.md
├── .gitignore
├── requirements.txt
├── config/                          # Runtime configuration (edit before running)
│   ├── tradeify_config.py           # Checkout flow settings (credentials, plan, payment)
│   └── staging_config.py            # Staging registration settings
├── utils/                           # Shared helper modules
│   ├── browser.py                   # Chrome driver factory (standard + undetected)
│   ├── selectors.py                 # UI interaction helpers (click, fill, find)
│   ├── screenshot.py                # Screenshot helper → output/screenshots/
│   ├── captcha.py                   # reCAPTCHA v2/v3 solving via 2captcha.com
│   ├── oauth.py                     # Google OAuth popup handler
│   └── webdriver_paths.py           # Browser driver path resolver for Robot Framework
├── tests/
│   ├── ui/
│   │   ├── tradeify_checkout.py     # Full checkout flow automation
│   │   └── staging_registration.py  # Staging registration automation
│   ├── robot/
│   │   └── test_setup.robot         # Robot Framework test suite
│   ├── api/
│   │   ├── package.json             # Node.js dependencies
│   │   ├── checkout_api.js          # API checkout endpoint test
│   │   ├── checkout_poll.js         # Async API test
│   │   └── checkout_stress.js       # Stress test (1000 concurrent requests)
│   └── loadtest/
│       └── locustfile.py            # Locust load test for checkout API
└── output/                          # .gitignored test artifacts
    ├── screenshots/                 # Debug screenshots from test runs
    └── reports/                     # Robot Framework reports & load test CSVs
```

## Running Scripts

```bash
# Install Python dependencies
pip install -r requirements.txt

# Run the main checkout automation
python tests/ui/tradeify_checkout.py

# Run the staging registration test
python tests/ui/staging_registration.py

# Run Robot Framework tests
robot --outputdir output/reports tests/robot/test_setup.robot

# Run a specific Robot Framework test by name
robot --test "Test Case Name" --outputdir output/reports tests/robot/test_setup.robot

# Load testing with Locust (web UI at http://localhost:8089)
locust -f tests/loadtest/locustfile.py

# Load testing headless (CLI only, outputs CSV reports)
locust -f tests/loadtest/locustfile.py --headless -u 10 -r 2 -t 1m --csv output/reports/loadtest

# Load test only the direct API (skip app proxy)
locust -f tests/loadtest/locustfile.py --headless -u 10 -r 2 -t 1m DirectAPICheckout

# Node.js API tests
cd tests/api && npm install && node checkout_api.js
```

## Architecture

**Configuration-first pattern**: Each test has a dedicated config file in `config/` where all runtime parameters are set as constants. Change these values — not command-line args — to adjust behavior.

- [config/tradeify_config.py](config/tradeify_config.py) — Credentials, plan selection, payment method, CAPTCHA settings
- [config/staging_config.py](config/staging_config.py) — Staging HTTP auth, login credentials, new user details

**Shared utilities** in `utils/`:
- [utils/browser.py](utils/browser.py) — `create_driver(use_undetected, headless)` factory function
- [utils/selectors.py](utils/selectors.py) — `js_click()`, `find_btn()`, `click_btn()`, `click_by_text()`, `fill()`, `fill_variants()`, `fill_if_empty()`, `fill_input_in_frame()`
- [utils/screenshot.py](utils/screenshot.py) — `ss(driver, name)` saves to `output/screenshots/`
- [utils/captcha.py](utils/captcha.py) — reCAPTCHA v2/v3 solving and auto-detection
- [utils/oauth.py](utils/oauth.py) — Google OAuth popup credential filling
- [utils/webdriver_paths.py](utils/webdriver_paths.py) — Browser driver path resolution for Robot Framework

**CAPTCHA handling:**
- `USE_UNDETECTED_CHROME = True` uses `undetected_chromedriver` to avoid bot detection
- `TWOCAPTCHA_API_KEY` enables automated solving via the 2captcha.com API
- `MANUAL_CAPTCHA_PAUSE = True` pauses execution before payment so you can solve CAPTCHAs manually

**Screenshots**: All scripts use `ss(driver, name)` which writes PNGs to `output/screenshots/` for debugging.

**Browser targets**: Chrome is primary (via `undetected_chromedriver`). `utils/webdriver_paths.py` adds Firefox and Edge paths for Robot Framework use.

## Key Selectors Strategy

Scripts use multiple fallback selector strategies (ID → name → CSS → XPath → JS click) because the Tradeify UI uses dynamic class names. The shared helper functions in [utils/selectors.py](utils/selectors.py) implement this pattern.

---
> Source: [qa-pronay/automation_code](https://github.com/qa-pronay/automation_code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
