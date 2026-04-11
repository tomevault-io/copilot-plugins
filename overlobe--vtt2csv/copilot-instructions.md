## vtt2csv

> **vtt2csv** is a Python script that converts WebVTT (.vtt) subtitle/caption files to CSV format. Extracts timestamps, speaker names, and text into structured columns for easier searching and analysis.

# AGENTS.md

## Project Overview

**vtt2csv** is a Python script that converts WebVTT (.vtt) subtitle/caption files to CSV format. Extracts timestamps, speaker names, and text into structured columns for easier searching and analysis.

Single-file utility — no dependencies beyond Python 3 standard library.

## Architecture & Structure

```
├── vtt2csv.py    # The entire tool — single Python script
└── README.md     # Usage documentation
```

## Setup & Commands

```bash
# No installation needed — just Python 3
python3 vtt2csv.py              # Run with prompts
python3 vtt2csv.py /path/to/vtt # Process a folder of .vtt files
```

**Output:** Creates a `.csv` file alongside each `.vtt` file with columns: Timestamp, Speaker, Text.

## Code Style & Conventions

- Python 3, standard library only (no pip dependencies)
- Single-file script — keep it self-contained
- Regex-based VTT parsing
- Handles various VTT formats (with/without speaker names, different timestamp formats)

## What NOT to Do

- Don't add external dependencies — the point is zero-install simplicity
- Don't break single-file portability

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/overlobe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/overlobe)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
