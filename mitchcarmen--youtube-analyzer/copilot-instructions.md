## youtube-analyzer

> Python project for YouTube analytics and data collection/analysis.

# YouTube Setup Project

## Overview
Python project for YouTube analytics and data collection/analysis.

## Tech Stack
- **Language**: Python 3.x
- **Package Manager**: pip with `requirements.txt`

## Project Structure
```
youtube_setup/
├── src/              # Main source code
├── data/             # Data files (gitignored)
├── tests/            # Test files
├── requirements.txt  # Dependencies
└── README.md
```

## Key Libraries (likely)
- `google-api-python-client` - YouTube Data API
- `yt-dlp` - Video/metadata downloading
- `pandas` - Data analysis
- `requests` - HTTP requests

## Commands
- Install deps: `pip install -r requirements.txt`
- Run tests: `pytest`

## Conventions
- Use type hints
- Follow PEP 8 style
- Store API keys in `.env` (never commit)

## Notes
- YouTube API quota: 10,000 units/day (free tier)
- Add any project-specific notes here as we build

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mitchCarmen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mitchCarmen)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
