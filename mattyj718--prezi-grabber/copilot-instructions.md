## prezi-grabber

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

prezi-grabber is a Python CLI tool and library that captures Prezi presentations as PDF files or individual PNG slides using a headless browser (Playwright). It's a single-script application.

## Commands

```bash
# Install dependencies
python3 -m pip install -r requirements.txt
playwright install chromium

# Capture a Prezi as PDF
python3 prezi_grabber.py -u "PREZI_URL"

# Capture as PNGs
python3 prezi_grabber.py -u "PREZI_URL" -f png -o ./output

# Run unit tests
pytest test_prezi_grabber.py -k "not integration"

# Run integration tests (requires network + Playwright)
pytest test_prezi_grabber.py -m integration
```

## Architecture

The application lives in `prezi_grabber.py`. Key functions:

- `extract_pres_id(url)` — parses Prezi URL to extract presentation ID
- `capture_slides(page, max_slides, wait_time)` — navigates slides in the browser, returns PNG byte buffers
- `download_presentation(url, wait_time, max_slides)` — launches Playwright, loads the Prezi, calls capture_slides
- `save_as_pdf(slides, path)` — converts PNG byte buffers to a single PDF via img2pdf
- `save_as_pngs(slides, directory)` — writes PNG byte buffers as numbered files
- `grab_prezi(url, output_format, output_dir, wait, max_slides)` — main public API, orchestrates the full pipeline
- `main()` — CLI entry point using argparse

Supported URL patterns: `prezi.com/view/<id>`, `prezi.com/p/<id>`, `prezi.com/<id>`

## Dependencies

- `playwright` — headless Chromium browser for rendering Prezi's WebGL viewer
- `img2pdf` — image-to-PDF conversion
- `pytest` — testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattyj718) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
