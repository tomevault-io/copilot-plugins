## rob-court

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a badminton/tennis court booking automation tool. It uses Selenium to automate form submission on jinshuju.com to reserve sports court slots.

## Commands

### Single User Mode
```bash
python booking.py --usr "姓名" --phone "手机号" --num 人数 --type "羽毛球" --court 场地编号 --date 从今天开始第几天 --time 时间段序号
```

### Batch Mode (Recommended)
```bash
python booking.py --batch
```

Batch mode reads users from `config.json` and runs bookings in parallel using threading.

## Architecture

The project consists of the following files:

- **booking.py** - Unified entry point that supports both single-user and batch modes:
  - Uses Selenium WebDriver with Chrome
  - Single user mode: accepts CLI arguments for one booking
  - Batch mode: reads from config.json and runs multiple bookings in parallel using threading
  - Waits until a specified time before executing (configured in `pause()` function)

- **config file for batch mode.json** - Configuration:
  - Contains user list with name, phone, court, and time
  - Common settings: num, type, date

- **jinze_rob.py** - Core booking script (legacy, called by booking.py)

- **multi_jinze.py** - Legacy batch runner (deprecated, use booking.py --batch)

## Dependencies

- Selenium (`selenium` package)
- Pillow (`PIL` package)
- ChromeDriver (located at `C:/chromedriver-win64/chromedriver.exe`)

## Configuration

In `booking.py`:
- Line 24: Set the start time (`START_TIME = '09:00:00.175'`)
- Line 22: ChromeDriver path if needed

The script currently targets `https://jinshuju.com/f/vcswPJ` for form submission.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sharkjingyang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
