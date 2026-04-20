## food-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Description
Food Tracker is a system that automatically analyzes food photos from Google Photos and provides nutritional summaries. It runs as a weekly cron job to track eating habits.

## Commands
- **Install dependencies**: `poetry install`
- **Run main script**: `poetry run python scripts/food_tracker_poc.py`
- **Download test images**: `poetry run python scripts/download-test-images.py`
- **Set up Google Photos**: `poetry run python scripts/google_photos_auth.py`
- **Format code**: `poetry run black .`
- **Run tests**: `poetry run pytest`
- **Run single test**: `poetry run pytest tests/test_file.py::test_function`

## Code Guidelines
- Use pathlib.Path for file operations
- Handle errors with appropriate try/except blocks
- Follow common Python naming conventions
- Log important events and errors

## Project Structure
- Environment variables in .env file
- Scripts in scripts/ directory
- Test images in test_images/ directory
- Results in test_results/ directory

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dipam7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
