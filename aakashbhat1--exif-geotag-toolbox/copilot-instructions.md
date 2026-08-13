## exif-geotag-toolbox

> Legend: 🎯session 🔴bugfix 🟣feature 🔄refactor ✅change 🔵discovery ⚖️decision 🚨security_alert 🔐security_note

<claude-mem-context>
# Memory Context

# [office_scripts] recent context, 2026-05-14 6:29pm GMT+5:30

Legend: 🎯session 🔴bugfix 🟣feature 🔄refactor ✅change 🔵discovery ⚖️decision 🚨security_alert 🔐security_note
Format: ID TIME TYPE TITLE
Fetch details: get_observations([IDs]) | Search: mem-search skill

Stats: 8 obs (3,618t read) | 104,934t work | 97% savings

### May 14, 2026
287 5:48p 🔵 Unified Image Processing Pipeline — run.py in office_scripts
S77 Refactor c:\dev\office_scripts\run.py — split 562-line monolith into focused modules, improve error handling, add typing, and optimize geocoding performance (May 14, 5:48 PM)
S73 Code improvement recommendations for office_scripts/run.py — a unified image geotagging pipeline (May 14, 5:48 PM)
S74 Code improvement recommendations for office_scripts/run.py — 20 issues identified and prioritized across impact tiers (May 14, 5:48 PM)
S75 Code improvement recommendations for office_scripts/run.py — analysis complete, 20 issues identified, no changes applied yet (May 14, 5:48 PM)
288 5:51p 🟣 Created models.py with GeoEntry and GeoInfo frozen dataclasses
289 5:52p 🟣 Extracted cli.py and csv_io.py as first two modules in run.py split
290 " 🟣 Created geo.py — geocoding and elevation module with deduplication and typed exception handling
291 " 🟣 Created exif.py — EXIF GPS injection module with split load/write error handling
292 5:53p 🔄 Created overlay.py — decomposed 180-line process_overlay_image into 7 focused functions
293 " 🔄 run.py rewritten as 141-line thin orchestrator — module split complete
294 5:54p 🟣 All 7 refactored modules compile cleanly — refactor complete and verified
S76 Refactor c:\dev\office_scripts\run.py — split 562-line monolith into focused modules, improve error handling, add typing, and optimize geocoding performance (May 14, 5:54 PM)
**Investigated**: - The original `run.py` (562 lines): contained CLI path collection, CSV parsing, geocoding/elevation calls, EXIF injection, and the entire PIL overlay rendering pipeline all in one file
    - Key pain points identified: monolithic 180-line `process_overlay_image`, silent broad except clauses, duplicate Nominatim calls per image for shared coordinates, magic numbers throughout, untyped dict-based row data, hardcoded GPU flag

**Learned**: - Nominatim enforces a 1.2s rate-limit delay; deduplicating unique (lat, lon) pairs before the image loop is the correct optimization pattern
    - `csv.Sniffer().has_header(sample)` can raise `csv.Error` on ambiguous input — always provide a string-match fallback
    - `piexif.load()` raises `InvalidImageDataError` (not generic `Exception`) for corrupt EXIF blocks
    - `torch.cuda.is_available()` should be wrapped in `try/except ImportError` so the script works without PyTorch installed
    - `response.raise_for_status()` must be called before `.json()` to surface HTTP-level failures from open-elevation.com
    - Frozen dataclasses (`@dataclass(frozen=True)`) enforce immutability on `GeoEntry`, `GeoInfo`, and the internal `_Rect` layout helper in overlay rendering

**Completed**: - Created 6 new modules: `models.py`, `cli.py`, `csv_io.py`, `geo.py`, `exif.py`, `overlay.py`
    - Rewrote `run.py` as a thin 125-line orchestrator
    - Geocoding deduplication: `resolve_unique_coords()` resolves unique coord set once; per-image loop calls `lookup()` from cache
    - All exception handling made specific: `GeocoderTimedOut/ServiceError/Unavailable`, `requests.RequestException`, `piexif.InvalidImageDataError`, etc.
    - `process_overlay_image` decomposed into 8 private helpers with single responsibilities
    - All magic numbers replaced with named module-level constants (`_BLUR_RADIUS`, `_CARD_CORNER_RADIUS`, `_JPEG_QUALITY`, etc.)
    - `GeoEntry` typed dataclass: lat/lon parsed to float at CSV read time; eliminates downstream `float()` re-parsing
    - All 7 modules verified: `python -m py_compile` clean; smoke test confirmed `decimal_to_dms(28.7041)` returns correct DMS tuple

**Next Steps**: - Invoking `/repo-doc-pass` to write REPO_CONTEXT.md, CURRENT_STEP.md, and changes.md to the vault at `C:\dev\second_brain\office_scripts\`
    - Storing learned knowledge back to the vault via `/brain` (Phase 5)
    - Remaining unimplemented items from the original 20-point list: `decimal_to_dms` seconds rounding precision fix, `_strip_non_latin` accented-Latin bug, `argparse` CLI fallback, pytest unit tests for pure functions


Access 105k tokens of past work via get_observations([IDs]) or mem-search skill.
</claude-mem-context>

---
> Source: [AakashBhat1/exif-geotag-toolbox](https://github.com/AakashBhat1/exif-geotag-toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
