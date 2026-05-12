## claude-skill-aso-cosmicmeta-ss

> This skill generates high-converting App Store and Google Play screenshots with device frames, benefit-driven headlines, and multi-language support.

# ASO Screenshot Generator — CosmicMeta

This skill generates high-converting App Store and Google Play screenshots with device frames, benefit-driven headlines, and multi-language support.

## Architecture

- `SKILL.md` — Main workflow definition (6 phases)
- `compose.py` — Screenshot compositor (ios, android, ipad, android_tablet)
- `generate_frame.py` — Device frame generator (iPhone, Android, iPad Pro 13", Android tablet)
- `gemini_enhance.py` — AI enhancement + translation via Gemini (nano-banana-pro / nano-banana-2)
- `showcase.py` — Preview gallery generator (side-by-side comparison)
- `update.py` — Version check and auto-update script
- `VERSION` — Current version number
- `assets/` — Generated device frame PNGs

## Key Rules

- **Never include paywall or IAP screenshots** in the generated set
- Always generate frames before composing (`python generate_frame.py`)
- Maintain exact platform dimensions:
  - iOS phone: 1320×2868 | Android phone: 1080×1920
  - iPad Pro 13": 2064×2752 | Android tablet: 1600×2560
- Output filenames use `{lang_code}_{n:02d}.png` format (e.g., `en_01.png`, `tr_03.png`)
- Text adapts to background luminance (white on dark, dark on light)
- `--lang-code` enables both correct output naming and cultural visual touches in Gemini prompts
- English-first workflow: generate English, then translate per language with `--translate-to` + `--lang-code`

## Dependencies

- Python 3
- Pillow (`pip install Pillow`)
- Font: SF Pro Display (macOS) — falls back to system fonts on other platforms
- Gemini API key (required, for AI enhancement with nano-banana-pro or nano-banana-2)

---
> Source: [abutun/claude-skill-aso-cosmicmeta-ss](https://github.com/abutun/claude-skill-aso-cosmicmeta-ss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
