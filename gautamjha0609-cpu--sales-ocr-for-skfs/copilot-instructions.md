## sales-ocr-for-skfs

> When asked to "read the new ledger photos" (or similar):

# Reading new ledger photos (no API key)

When asked to "read the new ledger photos" (or similar):

1. `python -m skfs_ocr pending` — numbers the unread photos in `input/` as #1, #2 …
   (any file name works, e.g. WhatsApp names; the date comes from the page).
2. `python -m skfs_ocr tiles --pending` — 4 zoomed tiles per photo in `data/tiles/pNN_*.png`.
3. For each photo: Read its 4 tiles, transcribe exactly what is written (rules: `PROMPT`
   in `skfs_ocr/schema.py`), save with `python -m skfs_ocr fill '#n' <<'EOF' {...} EOF`
   (compact format: `skfs_ocr/fill.py`). The command prints the checks.
   - Only on an ERROR, look again — and only where the message points:
     `[one-digit fix would be: X -> Y]` names the exact number to zoom into
     (`tiles '#n' --box l,t,r,b`); `LIKELY MISREAD` = two checks agree, apply it;
     "total sale written … is wrong" = the rest agrees, only the total is off.
   - If the page itself is wrong, keep what is written and explain in `"u"`. Never change
     a number to pass a check unless the zoomed photo supports it (say so in `"u"`).
   - Unreadable date? Leave `"d"` as written (e.g. "?/4/25"): the run dates the page from
     the meter readings (each day opens where the day before closed).
4. `python -m skfs_ocr run --no-api` and tell the user which days are OK / CHECK / ERROR.
   Used photos move to `raw_data/FY..../MM Mon-YYYY/<yyyy-mm-dd>.jpg` (FY = April-March);
   workbooks go to `output/FY..../MM Mon-YYYY/`. Add new regular udhar parties to
   `parties.yaml`. To correct a filed photo: `fill 2025-04-07.jpeg` and run again.

Do not edit `template/template.xlsx` or files in `output/` by hand.
Run `python -m pytest -q` after any code change.

---
> Source: [gautamjha0609-cpu/sales-ocr-for-skfs](https://github.com/gautamjha0609-cpu/sales-ocr-for-skfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
