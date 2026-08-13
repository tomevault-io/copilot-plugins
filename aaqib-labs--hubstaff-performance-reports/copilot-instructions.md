## hubstaff-performance-reports

> Automates bi-weekly workforce performance compliance reports for ~80–90 WebLife Ventures employees tracked via Hubstaff (plus Centrifuse Engineers via TMetric). Reports are published as static HTML via GitHub Pages for internal executive review.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Repo Does

Automates bi-weekly workforce performance compliance reports for ~80–90 WebLife Ventures employees tracked via Hubstaff (plus Centrifuse Engineers via TMetric). Reports are published as static HTML via GitHub Pages for internal executive review.

---

## Session Startup Checklist

At the start of every session, before doing anything else:

1. Read this file (`CLAUDE.md`)
2. Read `data/reference/sla_violation_legend.md` — authoritative threshold logic
3. Read `data/personnel/personnel_index.md` — authoritative team/role assignments
4. Check `data/input/` for any new master table CSV files
5. Execute whatever is asked

---

## Folder Structure

```
data/input/
  monthly/           Full calendar month CSVs      → HS-YYYY-MM-master.csv
  biweekly/          Partial/bi-weekly period CSVs → HS-YYYY-MM-DD_to_YYYY-MM-DD.csv
                                                     CE-YYYY-MM-DD_to_YYYY-MM-DD.csv
data/personnel/      Personnel Index — authoritative role/team source
data/reference/      SLA thresholds and violation legend
scripts/             Python report generation scripts
  utils.py           Shared helpers (working-days, proration) — import from here
templates/           Jinja2 HTML report templates
docs/                GitHub Pages source — index.html + all report HTML files
CLAUDE.md            This file
```

**File naming conventions (strictly enforced):**

*CSV inputs (`data/input/`):*
| Type | Format | Example |
|------|--------|---------|
| Hubstaff full month | `HS-YYYY-MM-master.csv` | `HS-2026-03-master.csv` |
| Hubstaff bi-weekly | `HS-YYYY-MM-DD_to_YYYY-MM-DD.csv` | `HS-2026-03-01_to_2026-03-24.csv` |
| Centrifuse Engineers (CE) | `CE-YYYY-MM-DD_to_YYYY-MM-DD.csv` | `CE-2026-05-01_to_2026-05-19.csv` |

*HTML outputs (`docs/`):*
| Type | Format | Example |
|------|--------|---------|
| Hubstaff bi-weekly | `YYYY-MM-DD_to_YYYY-MM-DD_biweekly_top_violators.html` | `2026-03-01_to_2026-03-24_biweekly_top_violators.html` |
| Centrifuse Engineers (CE) | `YYYY-MM-DD_to_YYYY-MM-DD_ce_report.html` | `2026-03-01_to_2026-03-24_ce_report.html` |
| Pattern Analysis (HS) | `YYYY-MM-DD_to_YYYY-MM-DD_pattern_analysis.html` | `2026-01-01_to_2026-03-31_pattern_analysis.html` |
| Pattern Analysis (CE) | `YYYY-MM-DD_to_YYYY-MM-DD_ce_pattern_analysis.html` | `2026-01-01_to_2026-03-31_ce_pattern_analysis.html` |
| Peer Comparison | `YYYY-MM-DD_to_YYYY-MM-DD_peer_comparison.html` | `2026-03-01_to_2026-03-31_peer_comparison.html` |

Note: HTML outputs use date-range prefix + report-type suffix. No `HS-`/`CE-` prefix on outputs — the suffix identifies the source/type. This is by design so `update_index.py` can parse them consistently.

**GitHub Pages serves from `/docs`.** All generated reports are written directly to `/docs/` — there is no separate `/reports/` archive folder. Never rename or remove the `/docs` folder.

---

## Recurring Bi-Weekly Workflow

When asked to "generate the bi-weekly report for [date range]":

1. Find the master table CSV in `data/input/biweekly/`
2. Run `python scripts/generate_biweekly_report.py --input data/input/biweekly/[FILE].csv --start YYYY-MM-DD --end YYYY-MM-DD`
3. Script writes HTML directly to `docs/[start]_to_[end]_biweekly_top_violators.html` and updates `docs/index.html`
4. Commit with message: `Report: Bi-Weekly [start] to [end]`
5. Push to GitHub

If any data anomaly is detected (unexpected columns, missing data, parse errors), flag it before finalising.

---

## Master Table CSV — Column Reference

The input CSV always has these columns (Hubstaff export format):

| Column | Notes |
|--------|-------|
| `Team(s)` | Hubstaff team label — do NOT use for role matching; use personnel_index.md |
| `Member` | Employee name |
| `Activity %` | Keyboard/mouse engagement percentage |
| `Total Worked Hours` | Total logged hours in the period |
| `Break Time` | Raw break hours |
| `Break % of Total` | Break as % of total worked hours |
| `Total Manual Hours` | Manually entered hours |
| `Manual % of Total` | Manual as % of total worked hours |
| `Low Activity Hours (≤20%)` | Hours where activity ≤ 20% |
| `Low Activity % (≤20%)` | Low activity ≤20% as % of total worked hours |
| `Low Activity Hours (≤30%)` | Hours where activity ≤ 30% |
| `Low Activity % (≤30%)` | Low activity ≤30% as % of total worked hours |
| `SLA Violation Legend` | Pre-populated flag string — use as reference only |
| `Red Flag Count` | Pre-populated count — re-evaluate from raw data, do not trust blindly |
| `Yellow Flag Count` | Pre-populated count — re-evaluate from raw data |
| `Total Flags` | Pre-populated count — re-evaluate from raw data |

**Always re-evaluate all flags from raw data.** Pre-populated flag columns are for reference; the scripts are the authoritative source of flag logic.

**Important — H⚠️ in the pre-populated SLA column:** The upstream CSV builder adds a yellow hours warning (H⚠️) for employees slightly below 160h. Our system has NO yellow band for hours — only 🔴 red (below prorated threshold) and 🟠 orange (overwork). The script correctly ignores H⚠️ and re-evaluates hours as red or orange only. This is expected and correct — do not treat H⚠️ entries as anomalies.

---

## Threshold Logic

See `data/reference/sla_violation_legend.md` — this is the ONLY source of truth.

**Key rules:**
- NEVER guess or infer thresholds
- NEVER prorate thresholds that are explicitly provided — only auto-prorate when computing from a date range
- Red flag overrides yellow for the same metric — never show both
- Break yellow (B ⚠️) excluded from displayed flag count but included in severity score
- Hours SLA: strict red/orange only, no yellow band

**Auto-prorate hours thresholds from date range:**
```
prorated_red    = (working_days_in_period / working_days_in_full_month) × 160
prorated_orange = (working_days_in_period / working_days_in_full_month) × 200
```
Print both calculated thresholds to console before processing. Count Mon–Fri only.

**Holiday handling:** Do NOT attempt to subtract US holidays from working-day counts. Hubstaff and TMetric already include approved time-off and holiday hours in the employee's `Total Worked Hours` figure. The raw CSV total is the authoritative hours count — no further adjustment is needed.

---

## Personnel Index Rule

The `Team` column displayed in reports comes directly from the CSV's `Team(s)` column — the script does not do a personnel index lookup for team names.

The personnel index (`data/personnel/personnel_index.md`) is used for:
- Context when interpreting anomalies (e.g. role-appropriate low activity for executives)
- Confirming whether a name in the CSV maps to a known employee
- Flagging name mismatches between the CSV and the index (e.g. name changes, new hires not yet added)

If a name in the CSV does not match anyone in the index, flag it to Aaqib before finalising — do not silently skip or guess.

**Centrifuse Engineers (CE):** Will have a separate dedicated report page. Do not include CE members in the standard bi-weekly Hubstaff report unless explicitly instructed.

---

## Formatting Standards

| Field | Format |
|-------|--------|
| Activity % | Whole number, e.g. `34%` |
| All other numeric values | 1 decimal place |
| All percentage fields | Always include `%` symbol |
| Break / Manual / Low Activity cells | `Xh (X.X%)` format |
| Flags badge | `🔴 2 ⚠️ 1` (B ⚠️ excluded from count) |
| Member name cells | No status emoji prefix |
| Empty / compliant cells | Leave blank — no "CLEAR" text |
| Violation indicators | Embedded in metric cells: `🔴 34%`, `⚠️ 8.3h (11.4%)` |

---

## Exclusions

**Permanent exclusions (always applied automatically by the script):**
- **Contractors** — all personnel listed under the Contractors section of `data/personnel/personnel_index.md`
- **Resigned personnel** — add to the `PERMANENT_EXCLUSIONS` list in `generate_biweekly_report.py` as they are confirmed offboarded

**Cycle-specific exclusions** (new hires in grace period, etc.) are passed via `--exclude "Name 1,Name 2"` after the initial report is generated.

---

## Severity Scoring (for Top 15 ranking)

```
Base: Red = 10 pts | Yellow = 3 pts | Orange = 7 pts

Multipliers:
  A  Activity        → 5×
  M  Manual          → 4×
  H  Low Hours 🔴    → 3×
  H  Overwork 🟠     → 2×
  20 Low Act ≤20%    → 2×
  30 Low Act ≤30%    → 1.5×
  B  Break           → 1×

Score = sum of (base × multiplier) across all flags
```

---

## Git Conventions

- Always commit and push after generating reports
- Commit message formats:
  - Bi-weekly: `Report: Bi-Weekly YYYY-MM-DD to YYYY-MM-DD`
  - Pattern Analysis: `Report: Q1 Pattern Analysis YYYY-MM-DD to YYYY-MM-DD`
  - CE Pattern Analysis: `Report: CE Q1 Pattern Analysis YYYY-MM-DD to YYYY-MM-DD`
  - Centrifuse Engineers: `Report: Centrifuse Engineers YYYY-MM-DD to YYYY-MM-DD`
  - Peer Comparison: `Report: Peer Comparison YYYY-MM`
- Never leave uncommitted changes after a session
- Never force-push

---

## Scripts Reference

| Script | Purpose |
|--------|---------|
| `scripts/utils.py` | Shared helpers — working-days, proration, SLA thresholds, flag evaluation, exclusion lists. Import from here, never duplicate. |
| `scripts/generate_biweekly_report.py` | Hubstaff bi-weekly report generation |
| `scripts/generate_ce_report.py` | Centrifuse Engineers (TMetric) report generation |
| `scripts/generate_pattern_analysis.py` | Quarterly repeated pattern analysis — fully implemented |
| `scripts/update_index.py` | Regenerate `docs/index.html` from all reports in `/docs/` |
| `scripts/generate_peer_comparison.py` | Role-based peer comparison — fully implemented |
| `scripts/generate_ce_pattern_analysis.py` | Centrifuse Engineers Q1 pattern analysis (TMetric data) |

---

## Report Structure

Each bi-weekly HTML report contains:

**Section 1 — Top 15 Violators**
Ranked by severity score. Columns: Rank, Member, Team, Activity %, Hours, Break %, Manual Hours, Low Act ≤20%, Low Act ≤30%, Flags, Score.

**Section 2 — Hours Violators**
All employees below the prorated hours threshold. Sorted ascending (worst first). Columns: Member, Team, Hours Worked, Expected Hours, Shortfall, Other Flags.

---

## Peer Comparison Workflow

When asked to "generate the peer comparison report for [month]":

1. Confirm the monthly CSV is in `data/input/monthly/`
2. Run:
```
python scripts/generate_peer_comparison.py \
  --input data/input/monthly/HS-YYYY-MM-master.csv \
  --start YYYY-MM-01 --end YYYY-MM-DD
```
3. Script writes HTML to `docs/[start]_to_[end]_peer_comparison.html` and updates `docs/index.html`
4. Commit with message: `Report: Peer Comparison YYYY-MM`
5. Push to GitHub

**How to prompt for this report:**
> "Generate the peer comparison report for March 2026"

**Peer Comparison — Key Rules:**
- 17 hardcoded peer groups defined in `scripts/generate_peer_comparison.py` → `PEER_GROUPS`
- Groups are defined by role/function, NOT Hubstaff team labels
- Manager listed first in each group table with blue `Manager` badge
- Variance column = employee Activity % minus team average Activity %
- Peer Outlier = employee is >10pp below team average BUT has no SLA flag (amber row highlight)
- New hires flagged with green `New Hire` badge (list in `NEW_HIRES` set in script)
- Team Average row at bottom of each table — dark charcoal background for visibility
- Pill-styled metric cells: red/orange/yellow filled for violations, gray for clean
- CE members excluded (they have separate CE reports)
- Contractors excluded (PERMANENT_EXCLUSIONS in utils.py)
- **To update peer groups** (new hires, team changes, resignations): edit `PEER_GROUPS` list in `generate_peer_comparison.py`

**HTML output naming:** `YYYY-MM-DD_to_YYYY-MM-DD_peer_comparison.html`

---

## Repeated Pattern Analysis Workflow

When asked to "generate the Q1 / quarterly pattern analysis report":

1. Confirm 3 full monthly CSVs are in `data/input/monthly/`
2. Run:
```
python scripts/generate_pattern_analysis.py \
  --months data/input/monthly/HS-YYYY-MM-master.csv \
           data/input/monthly/HS-YYYY-MM-master.csv \
           data/input/monthly/HS-YYYY-MM-master.csv \
  --labels "Month1 YYYY" "Month2 YYYY" "Month3 YYYY" \
  --start YYYY-MM-DD --end YYYY-MM-DD
```
3. Script writes HTML to `docs/[start]_to_[end]_pattern_analysis.html` and updates `docs/index.html`
4. Commit with message: `Report: Q1 Pattern Analysis YYYY-MM-DD to YYYY-MM-DD`
5. Push to GitHub

**How to prompt for this report:**
> "Generate the Q1 pattern analysis report for January, February and March 2026"
> "Create the repeated pattern analysis for [Month1], [Month2], [Month3] [Year]"

**Pattern Analysis — Key Rules:**
- Only includes employees present in **all 3 months** — partial quarter employees excluded
- 10 sections: Activity Red, Activity Yellow, Overwork, Low Hours, Manual Red, Manual Yellow, Low Act ≤20% Red, Low Act ≤20% Yellow, Low Act ≤30% Red, Low Act ≤30% Yellow
- An employee appears in a section only if they have **2+ months** of violations of that specific severity for that metric
- Centrifuse Engineers members excluded (FS_EXCLUSIONS list in utils.py)
- Contractors excluded (PERMANENT_EXCLUSIONS list in utils.py)
- Hours thresholds for full months: red < 160h, orange ≥ 200h (no proration needed)
- Manual, Low Act ≤20%, Low Act ≤30% cells display as `XX.X% (Xh)` format
- Use `--sample` flag first to preview structure with 10 employees before running full report

---

## About This Project

**Owner:** Aaqib Hafeel, Process Optimization Lead, WebLife Stores LLC (TA-PO)
**Cycle:** Bi-weekly
**Coverage:** ~80–90 employees across 5 ventures
**Data sources:** Hubstaff (all teams) + TMetric (Centrifuse Engineers)
**Audience:** Executive review (Lucas Robinson, Jorn Wossner, department directors)

---
> Source: [aaqib-labs/hubstaff-performance-reports](https://github.com/aaqib-labs/hubstaff-performance-reports) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
