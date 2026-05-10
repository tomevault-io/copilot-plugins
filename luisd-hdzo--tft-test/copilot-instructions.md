## tft-test

> Python web application.

# CLAUDE.md

## Project Overview

Python web application.

## Tech Stack

- Language: Python
- Framework: Flask
- Type: Full-stack web app

## Commands

- Install dependencies: `pip install -r requirements.txt`
- Run dev server: `python run.py`
- Run tests: `pytest`

## Project Structure

```
claude test/
├── app/
│   ├── __init__.py          # App factory (create_app)
│   ├── routes/              # Blueprint route handlers
│   │   └── main.py
│   ├── services/            # Business logic & Pyppeteer scraping
│   ├── models/              # Data models
│   ├── templates/           # Jinja2 HTML templates
│   └── static/
│       ├── css/
│       ├── js/
│       └── images/
├── config/
│   └── settings.py          # App configuration
├── tests/                   # Test files
├── run.py                   # Entry point
├── requirements.txt
├── .env                     # Environment variables (not committed)
└── .gitignore
```

## Code Style

- Follow PEP 8 conventions
- Use type hints for function signatures
- Keep functions focused and small

## Guidelines

- Write tests for new features
- Keep dependencies minimal
- Use environment variables for configuration (never commit secrets)

---
> Source: [Luisd-hdzo/tft_test](https://github.com/Luisd-hdzo/tft_test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
