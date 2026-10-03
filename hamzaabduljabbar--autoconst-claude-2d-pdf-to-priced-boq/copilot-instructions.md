## autoconst-claude-2d-pdf-to-priced-boq

> > This file is read by Claude at the start of every session in this project.

# CLAUDE.md — 2D PDF Drawing → Priced BOQ
# AutoConst | Antigravity Project Brain

> This file is read by Claude at the start of every session in this project.
> It defines what the project does, the pipeline stages, and the output format.

---

## Quickstart (for a demo / first-time user)

You need two things in place:
1. A drawing PDF in `inputs/` (any name — Claude will pick it up)
2. A rate library at `rates/rate-library.csv` (copy `rate-library.template.csv` and fill in prices)

Then say to Claude:

> "Run the takeoff on my PDF and produce a priced BOQ. Save to `outputs/`."

Claude executes the pipeline and hands back a formatted Excel BOQ.

### For Claude — what to do when the user asks to run the pipeline

When the user asks something like *"run the takeoff"*, *"do the takeoff"*, *"price this drawing set"*, or *"generate the BOQ"* — this is a single request: run the WHOLE sequence below in order without stopping to ask between stages. The user should never have to name a subcommand; you orchestrate them.

**Step 0 — pick the input path.** Look in `inputs/` (or wherever the user pointed):
- **A `.dxf`** → run the **DXF path** (exact). Skip the PDF passes entirely.
- **A `.pdf`** → run the **PDF path**. If several files, ask which; if none, ask for one.
- Confirm the rate library exists at `rates/rate-library.csv` (else tell the user to copy the template and fill it — never invent rates).

### DXF path (exact — 4 commands)
```
py scripts/takeoff.py dxf "inputs/<name>.dxf" --db outputs/<name>.db
py scripts/to_boq.py    --db outputs/<name>.db --out outputs/<name>_q.csv
py scripts/classify.py  outputs/<name>_q.csv outputs/<name>_c.csv
py scripts/price_boq.py --quantities outputs/<name>_c.csv --rates rates/rate-library.csv --out outputs/<name>-priced-boq.xlsx
```
Then report. No schedules/reconcile/measure — the geometry is the truth.

### PDF path (run every stage — don't skip the structural ones)
```
py scripts/takeoff.py build       "inputs/<name>.pdf" --db outputs/<name>.db   # + flags raster pages
py scripts/takeoff.py schedules   "inputs/<name>.pdf" --db outputs/<name>.db --auto
py scripts/takeoff.py dimensions  --db outputs/<name>.db                       # concrete m3 / steel from schedule dims
py scripts/takeoff.py annotations "inputs/<name>.pdf" --db outputs/<name>.db   # inline plan labels (columns/piles/French beams) — the ONLY thing that finds elements on schedule-less structural sets
py scripts/takeoff.py locate      --db outputs/<name>.db --include-numeric     # find scheduled marks on plans
py scripts/takeoff.py reconcile   --db outputs/<name>.db                       # promote MED->HIGH
py scripts/takeoff.py layers      "inputs/<name>.pdf" --db outputs/<name>.db   # OCG type-witness (no-op if flattened)
py scripts/takeoff.py measure     --db outputs/<name>.db                       # gross floor/slab area from printed dims
py scripts/takeoff.py quantities  --db outputs/<name>.db                       # headline qty + confidence per type
py scripts/takeoff.py rebar       "inputs/<name>.pdf" --db outputs/<name>.db   # reinforcement register + tonnage estimate
py scripts/to_boq.py    --db outputs/<name>.db --out outputs/<name>_q.csv
py scripts/classify.py  outputs/<name>_q.csv outputs/<name>_c.csv
py scripts/price_boq.py --quantities outputs/<name>_c.csv --rates rates/rate-library.csv --out outputs/<name>-priced-boq.xlsx
```
(Drop `--include-numeric` only if door marks are clearly not room-numbered. `schedules` is slow ~30s/page — that's expected, not a hang.)

**Report** the grand total, HIGH/MEDIUM/LOW/RATE_NOT_FOUND counts, per-trade quantities, any raster pages flagged, and the output path.

Do not invent quantities. Do not invent rates. Do not skip stages. **Do not eyeball the drawing for a count** — every number must be a query result from the DB. If the user asks how you know a number, back it with `cite`:

```
py scripts/takeoff.py cite --db outputs/<name>.db --type door
py scripts/takeoff.py cite --db outputs/<name>.db --mark 270A --crop
```

---

## Project overview

This project turns a **flat construction PDF** into a **priced Bill of Quantities**
in Excel. It reads the PDF's hidden vector text layer, builds a queryable
SQLite database once, cross-checks schedule counts against plan callouts, and
prices the reconciled quantities against a user-supplied rate library.

The output is a file, not a chat response.

Input is a flat PDF export — the same file a contractor would receive. No BIM
model needed. Works whether the drawings use `D-101`, `210.1`, `270A`, or any
other convention for door marks — see the `locate` step.

---

## The pipeline

```
  drawing.pdf                                                        rate library
      │                                                                    │
      ▼                                                                    ▼
   build ─► schedules ─► locate ─► reconcile ─► layers ─► quantities ─► to_boq ─► classify ─► price
      │        │           │          │           │           │           │           │          │
      ▼        ▼           ▼          ▼           ▼           ▼           ▼           ▼          ▼
   SQLite    schedule   plan       promote     type      recommended    BOQ         CSI codes   priced
   + words   rows       marks      MED->HIGH   witness   qty per type   quantities  + units    .xlsx
             (HIGH)     found                            + confidence   .csv
                        on plans
```

**`build`** — reads vector text, sheets, printed dimensions, tag callouts. Fast (~seconds per 100 pages).

**`schedules`** — table detection on schedule sheets. Each row = one HIGH-confidence element with full attributes captured (room, size, material, fire rating, hardware…). Slow (~30s/page) — point at just the schedule sheets, or use `--auto`. Skips **materials/finish legends** automatically (a table of `PT-1 / ACT-1 / VCT-1` is a code dictionary, not a count of elements).

**`dimensions`** — reads the dimension COLUMNS a schedule already carries and computes real per-element geometry: footing `L×W×D` → concrete m³, beam/column `Section + Span` → steel length + tonnage (a `W18×35` name literally encodes 35 lb/ft), concrete member `W×D×L` → m³. This is what turns structural elements from LOW ("no geometric quantity, verify") into HIGH — billed on m³/m/kg. Pure text, no vision. Doors/fixtures (billed each) are skipped — they don't need dimensions. Tags each element `concrete`/`steel` so the classifier picks the right unit. Re-run whenever `schedules` re-runs.

**`locate`** — the convention-agnostic plan matcher. Default regex `TAG_PATTERNS` only catch `D-101`-shaped marks; real sets label doors any number of ways. `locate` searches the plans for **the exact marks the schedule contains**, so it adapts to any convention. Presence-only — used to *confirm*, never to set a quantity (one door's mark appears on plan + RCP + finish = many hits per door).

**`reconcile`** — two-witness rule. Mark in schedule AND on plan → promote MEDIUM → HIGH. Also flags `schedule_only` (in schedule, no plan callout — verify) and `plan_only` (on plan, no schedule row — verify). Layer VETOes are preserved so the two passes are order-independent.

**`layers`** — opt-in. Uses PDF Optional Content Group (OCG / CAD layer) names as an independent type witness. A `D101` on layer `A-DOOR` confirms the type; on `A-GLAZ` it's a regex false-positive and gets vetoed to LOW. Assigns each word its layer by the *disappearance method* (hide one layer, see which words vanish). Degrades honestly when the PDF is flattened (no OCGs) — most exported print PDFs are.

**`quantities`** — turns raw witness data into a per-type recommended count with a confidence and a reason. Detects per-instance schedules (147 rows = 147 doors, HIGH) vs per-type schedules (2 rows × N callouts = per-type catalogue, MEDIUM). Uses only genuine regex-plan-symbols for the ratio; `locate` presence never inflates the count.

**`cite`** — the grounding/audit tool. `cite --type door` shows how the count was derived (schedule sheet + corroboration rate). `cite --mark 270A --crop` renders a PNG of every place `270A` appears on the PDF, so a human eyeballs one 2-second image instead of scanning an E-size sheet. Turns "trust the AI" into "look at the crop."

**`to_boq.py`** — the adapter. Converts drawing DB → the same `_quantities.csv` format the classifier consumes. Maps each element type to a synthetic IFC class (`door → IfcDoor`, `fixture → IfcSanitaryTerminal`…), one row per mark, tagged `qty_source=count_only`. Skips reference-only marks (grids, rooms). Any unmapped type is reported as `UNMAPPED` so you can add it to `TYPE_TO_IFC`.

**`classify.py`** — assigns CSI MasterFormat codes and billing units (doors → `08 11 00 ea`, fixtures → `22 40 00 ea`, windows → `08 50 00 m2`). Rule-based baseline; handles proxies, plates, coverings, MEP.

**`price_boq.py`** — joins classified quantities to the rate library and writes the formatted Excel BOQ. **Unit-aware**: enter rates in `sf`, `cy`, `lf`, `m2`, `m3`, `ea`, `ton`, `lb`, etc. and it converts the metric takeoff quantity into your rate's unit — no manual conversion needed. Cross-dimension mismatches (area rate on a length item) are flagged in the notes, never silently mis-multiplied.

---

## The rate library (REQUIRED — user brings own)

Path: `rates/rate-library.csv`. Copy from `rate-library.template.csv` and fill in real prices.

- Sources: RSMeans, 4BT OpenCOST, state DOT bid tabs, your own price book — see `docs/building-your-rate-library.md`
- Enter rates in **whatever unit your source uses** (`sf`, `cy`, `lf`, `m2`, `m3`, `ea`, `ton`, `lb`, …) — the pricer converts automatically
- `rate-library.csv` is git-ignored; licensed/proprietary rates stay private

---

## Output format — the priced BOQ

Columns:

| Col | Header             | Filled by | Contents                                   |
|-----|--------------------|-----------|--------------------------------------------|
| A   | Item No            | Yes       | Sequential                                 |
| B   | Element            | Yes       | Element type (IfcDoor, IfcSanitaryTerminal…) |
| C   | Description        | Yes       | Mark + status (e.g. "270A" or "P1 [UNDOCUMENTED]") |
| D   | Ref                | Yes       | `DWG-<type>-<mark>` (traces to the DB row) |
| E   | Unit               | Yes       | Rate's unit (EA / m2 / cy / sf …)          |
| F   | Quantity           | Yes       | Number, in the rate's unit                 |
| G   | Rate               | Yes*      | From your library                          |
| H   | Total              | Yes*      | `=F*G` live formula                        |
| I   | Notes              | Yes       | HIGH/MEDIUM/LOW confidence, flags, RATE_NOT_FOUND |

\* Rate/Total are blank and flagged `RATE_NOT_FOUND` (red row) when no matching rate exists — the estimator sees exactly what to add.

---

## Confidence rules

- **HIGH** — schedule roster row, and (if reconciled) the mark was also found on a plan.
- **MEDIUM** — one witness only (schedule with no plan corroboration, or plan callout with no schedule row). Verify.
- **LOW** — layer-vetoed (regex/layer type clash), or a `count_only` element billed on a geometric unit (m²/m³/m) — a 2D count can't give you an area, so windows/walls come back LOW by design.
- **RATE_NOT_FOUND** — element classified but no rate in the library. Row shaded red; Rate/Total blank.

---

## Ordering dependencies (important — don't skip)

The passes have a strict order because later passes read what earlier passes wrote:

1. `build` must precede everything.
2. `schedules` must precede `dimensions` and `locate` (both read schedule rows).
3. `locate` must precede `reconcile` (locate produces plan_locate witnesses).
4. `reconcile` must precede `quantities` (reconcile promotes MED → HIGH).
5. `dimensions` reads schedule attributes and writes geometry columns; run it any time after `schedules` and before `to_boq`.
6. **If you change or re-run `schedules`, re-run `dimensions → locate → reconcile → quantities → to_boq → classify → price`.** Stale rows will produce wrong quantities in the BOQ.

---

## Operating rules

1. Always read this file before starting a takeoff.
2. Never invent quantities. Every number is a query result.
3. Never invent rates. Missing rate → `RATE_NOT_FOUND`, leave blank.
4. Never commit `rates/rate-library.csv` (git-ignored).
5. Never commit client PDFs from `inputs/` or generated `.db` / `outputs/` files (git-ignored).
6. Cite on request. If the user asks how you know a number, back it with `cite` (and `--crop` for a PNG they can eyeball).
7. Confirm which drawing set, rate-library version, and any tag-pattern tuning were used at the end of each run.

---

## Tuning per client (patterns.py)

Every drawing set has its own conventions. When counts look off, look here:

- **`TAG_PATTERNS`** — regex-per-type LIST for plan callouts. Each type can carry multiple regexes so one set can mix conventions (`D-101` and `210.1` at once). For marks that don't fit any regex, `locate` still finds them via the schedule.
- **`MATERIAL_CODE_PREFIXES`** — two-letter finish/paint/glazing codes (`PT`, `GL`, `RS`, `ACT`, `VCT`…) that look like grid refs but aren't. The single biggest defence against false "grid" counts. Extend it per set.
- **`SCHEDULE_KEYWORDS` / `SCHEDULE_STOPWORDS`** — table title words that classify a schedule, and first-cell labels that mark a header row.
- **`LAYER_PATTERNS`** — CAD/OCG layer names → element type. Only matters for PDFs that kept their layers.

Rebuild the DB after tuning: `py scripts/takeoff.py build ...`

---

## Built and maintained by

Hamza Jabbar — hamzajabbar.online

---
> Source: [hamzaabduljabbar/autoconst-claude-2d-pdf-to-priced-boq](https://github.com/hamzaabduljabbar/autoconst-claude-2d-pdf-to-priced-boq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
