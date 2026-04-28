## dcs-website-faculty-list-over-time

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goal

Analyze and visualize how the University of Toronto Department of Computer Science faculty has changed over time (2007–present). Three-stage pipeline: scrape Wayback Machine → parse HTML → fuzzy-match individuals → visualize.

## Setup

```bash
pip install -r requirements.txt
```

## Running

```bash
# Full pipeline
python3 pipeline.py

# Individual stages
python3 scrape.py        # download HTML snapshots → data/raw/
python3 parse.py         # extract faculty from HTML → data/parsed/
python3 match.py         # fuzzy-match across time → data/faculty_timeline.csv
python3 visualize.py     # generate plots → plots/

# Re-run from a specific stage
python3 pipeline.py --from parse
python3 pipeline.py --only visualize
```

### Useful flags

```bash
python3 scrape.py --dry-run          # see what would be fetched without downloading
python3 scrape.py --delay 3          # slower rate limit (be polite to Wayback Machine)
python3 parse.py --inspect data/raw/SOMEFILE.html   # dump raw text for debugging parsers
python3 parse.py --reparse           # overwrite existing parsed JSON
python3 match.py --threshold 90      # stricter fuzzy matching (default 85)
python3 match.py --verbose           # print every merge/new decision
```

## Architecture

### Data flow

```
Wayback Machine CDX API
        ↓ scrape.py
data/raw/*.html          (one file per monthly snapshot)
        ↓ parse.py
data/parsed/*.json       (structured: timestamp, faculty[{name, stream, title}])
        ↓ match.py
data/faculty_timeline.csv  (one row per canonical person: first_seen, last_seen, stream)
        ↓ visualize.py
plots/*.png
```

### URL patterns

The faculty listing URL changed several times. `scrape.py` queries the CDX API separately for each era:

| Era | URL slug | Date range |
|-----|----------|------------|
| 2007–2008 | `index.php?section=95` | `index_section95` |
| 2009–2020 | `/people/faculty.htm` | `faculty_htm` |
| 2020–2021 | `/contact-us/faculty-directory` | `contact_faculty_dir` |
| 2021+ | `/people/faculty-directory` | `people_faculty_dir` |

The output filename encodes both: `{YYYYMM}_{slug}.html`.

### HTML parsing (`parse.py`)

Three format detectors, dispatched by filename slug:

- **`old_php`** (2007–2008): table-based; falls back to `faculty_htm` parser if no table rows found.
- **`faculty_htm`** (2009–2020): scans for `<a>` tags with a `Professor|Lecturer` regex match in surrounding context.
- **`modern`** (2021+): confirmed structure — `<h2>` section headers ("Research Stream Faculty", "Teaching Stream Faculty", "Professors Emeriti", etc.) followed by faculty name links. Teaching stream uses a table; research stream uses a card/list layout.

The `classify_section()` function maps section header text to `research | teaching | skip`. Skip sections: Emeriti, Cross-Appointed, Status-Only, Adjunct, CLTA.

### Fuzzy matching (`match.py`)

Names are normalised (lower-case, strip punctuation and honorifics) then compared with `rapidfuzz.fuzz.token_sort_ratio` so word-order differences ("Smith John" vs "John Smith") don't penalise the score. Default threshold is 85. If the merge rate looks wrong, inspect with `--verbose` and adjust `--threshold`.

### Classification rules

- **Research stream**: "Professor", "Associate Professor", "Assistant Professor" (without "Teaching Stream" suffix)
- **Teaching stream**: "Teaching Stream" suffix, "Lecturer", "Teaching Professor"
- **Skip entirely**: Emeriti, Adjunct, CLTA, Cross-Appointed, Status-Only, Visiting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alecjacobson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
