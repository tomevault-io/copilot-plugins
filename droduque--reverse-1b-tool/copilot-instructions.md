## reverse-1b-tool

> SVN Rock Advisors is a Canadian real estate development consulting firm based in Burlington, Ontario. They produce **feasibility studies** (called "1A") for apartment developers. The 1A includes a one-page Excel proforma with unit mix, rents, operating expenses, NOI, and valuation at three cap rates.

# SVN Rock — Reverse 1B Automation

## Project Context

SVN Rock Advisors is a Canadian real estate development consulting firm based in Burlington, Ontario. They produce **feasibility studies** (called "1A") for apartment developers. The 1A includes a one-page Excel proforma with unit mix, rents, operating expenses, NOI, and valuation at three cap rates.

After delivering the 1A, they present a detailed **Reverse 1B**, a 15-sheet Excel financial model that reverse-engineers the full development cost structure from that single proforma page. This currently takes their fractional CFO (Noor) 2-3 hours per project. The goal is to automate it so junior staff can generate it in minutes, then Noor reviews.

### The Business Flow
1. Client pays $20-25K for a feasibility study
2. The feasibility study contains a **1A proforma** (one-page Excel with rents, expenses, NOI, valuation)
3. SVN takes that 1A and builds a **Reverse 1B**, a 15-sheet financial model working backwards from the building's value
4. The Reverse 1B is presented to the client for FREE as a "wow" moment to demonstrate expertise
5. This leads to paid services: full 1B financial modeling ($5K), mortgage brokerage, lease-up, etc.

---

## Project Tracking

At the END of every session:
- Update guides/PROJECT-SUMMARY.md with what was built
- Mark completed items as done
- Update "Last Updated" date

---

## Strategic Reference (read before any Derek interaction)

- [notes/derek-9-steps-playbook.md](notes/derek-9-steps-playbook.md) — Distilled from the 10 ADFSE Deep Dive videos. Per-step Derek vocabulary, dollar hooks, metaphors. Drop a Derek-phrase per conversation.
- [notes/svn-rock-consulting-launch.md](notes/svn-rock-consulting-launch.md) — Patron-platform engagement model, the "final hook." Phase 1 (Movie Mode, awaiting Derek) → Phase 2 (SVN OS modules) → Phase 3 (equity). **Don't act on Phase 2 until Derek surfaces it.**
- Raw transcripts: [context/reference/adfse-deep-dive/](context/reference/adfse-deep-dive/)

---

## Key Files

```
reference/
├── 1A_Birchmount_2240.xlsx           # Project A — 1A proforma (170 units, 3 unit types)
│                                      # This is the SOURCE data that was used to build the template
├── 1A_490_St_Clair.xls               # Project B — 1A proforma (372 units, 9 unit types incl. affordable)
│                                      # TEST CASE — .xls format, needs conversion
├── REVERSE_1B_Template.xlsx          # The finished Reverse 1B for Birchmount (15 sheets, full formulas)
│                                      # Sheet 1 contains the Birchmount 1A data
├── Reverse 1B - Example & Inputs.xlsx # Spec sheet listing all ~100 input parameters with defaults
└── SAMPLE - 1B Model.xlsx            # The FULL 1B model (8 sheets) — reference only

context/reference/
└── 1B_User_Manual.pdf                # Official user manual with color conventions and sheet descriptions

guides/
├── DATA_MAP.md                       # Cell-by-cell mapping: 1A proforma → Reverse 1B template
├── INVESTIGATION_REPORT.md           # Sheet-by-sheet analysis of the template
└── PROJECT-SUMMARY.md                # Build progress tracking
```

---

## Critical Rules — Read These Before Writing Any Code

### Rule 1: NEVER Overwrite Formula Cells
The Reverse 1B template uses a color convention:
- **BLUE cells** = user inputs/assumptions, SAFE to write to
- **BLACK cells** = formulas, **DO NOT TOUCH**
- **GREEN cells** = formula but overridable, write with caution

Before writing to ANY cell, check if it contains a formula. If it does, skip it unless you're absolutely certain it's meant to be overwritten.

### Rule 2: Sheet 1 Is The Entry Point
Sheet 1 ("1. 1A Proforma") is literally a copy of the standalone 1A proforma. The other 14 sheets pull from Sheet 1 via cell references. If you correctly replace Sheet 1's values, the model cascades automatically.

### Rule 3: Preserve Everything When Copying The Template
Load the template with `data_only=False` to preserve all formulas, formatting, charts, named ranges, data validations, and conditional formatting.

### Rule 4: The 1A Layout Has A Row Offset
The standalone 1A file and the 1A tab inside the Reverse 1B have the SAME layout but the Reverse 1B version is shifted down by 1 row. Always verify actual cell positions by reading both files.

### Rule 5: Unit Mix Is The Hard Part
The Birchmount template has 3 unit type rows. Other 1As may have 9+ unit types. Consolidate into the template's structure using weighted averages for SF and rents.

---

## 1A Proforma Layout (Dynamic Parsing)

The proforma has consistent COLUMNS but the ROW positions shift depending on unit count. Parser must find sections dynamically, DO NOT hardcode row numbers.

**Columns are fixed:**
```
D col:  Labels (unit type names, expense names)
E col:  Unit size (SF) / label text
F col:  Unit count / parking spaces / rates
G col:  Unit mix % / monthly fees / expense rates
H col:  Total SF / annual per unit
I col:  Monthly rent per unit / monthly per unit
J col:  $/SF
K col:  Monthly total / Annual total
L col:  Annual total / % of Revenue
```

**Sections found by scanning column D/E for landmark strings:**
1. **Title:** E2 (standalone) or F2 (template), contains "Estimated Stabilized Value"
2. **Unit Mix:** Starts at first row where D col has a unit type label. Ends at "TOTAL/AVG" row.
3. **Summary:** Find "Total Residential Units:" in E col
4. **Operating Revenues:** Find "ESTIMATED OPERATING REVENUES" in E col. Then scan for: "Rent", "Underground Parking"/"Parking Underground", "Storage Lockers", "Submetering", "Vacancies (Rent", "Commercial"/"Net Commercial", "Vacancies (Commercial", "TOTAL:"
5. **Operating Expenses:** Find "ESTIMATED OPERATING EXPENSES" in E col. Then scan for: "Management Fee", "Property Taxes", "Reserve"
6. **NOI:** Find "ESTIMATED NET OPERATING INCOME" in E col
7. **Valuation:** Find "ESTIMATED VALUATION" in G col. Three "Market Value @" rows follow.

---

## Write Targets — What We Actually Modify

Investigation (2026-03-04) revealed that Sheet 5 is **mostly formulas**. Our real write targets are three sheets:

### Sheet 1 ("1. 1A Proforma") — Primary data from the 1A
| Cell | What It Is | Source |
|------|-----------|--------|
| F2 | Title | 1A title |
| F3 | Address | Extracted from 1A title |
| D7:D9 | Unit type labels | 1A (consolidated if >3 types) |
| E7:E9 | Unit sizes (SF) | 1A (weighted avg if consolidated) |
| F7:F9 | Unit counts | 1A (summed if consolidated) |
| I7:I9 | Monthly rents | 1A (weighted avg if consolidated) |
| D18 | Parking label ("Underground Parking" or "Surface Parking") | Derived from 1A parking type |
| F18, G18 | Parking spaces and monthly fee | 1A (underground or surface) |
| F19, G19 | Visitor parking | 1A |
| F20, G20 | Retail parking | 1A |
| F21, G21 | Storage lockers (count, fee) | 1A |
| F24 | Vacancy rate | 1A |
| F26, G26 | Commercial retail (SF, $/SF) | 1A (or 0 if none) |
| F27 | Commercial vacancy | 1A (or 0) |
| G37 | Management fee % | 1A |
| F38 | Property tax rate | 1A |
| G38 | Assessed value per unit | 1A |
| H46:H48 | Cap rates (best/base/worst) | 1A |

### Sheet 1 — Internal Section (Operating Assumptions, rows 58+)
| Cell | What It Is | Default |
|------|-----------|---------|
| F62 | Building GFA | From 1A or estimated |
| F64 | Amenity space SF | ~2.5% of GFA |
| F80 | Utilities $/PSF common area | 11 |
| I93 | R&M per unit (rounded) | 1,050 |
| I109 | Staffing per unit | 1,200 |
| F117 | Insurance per unit | 450 |
| F122 | Marketing per unit | 300 |
| F127 | G&A per unit | 250 |
| I137 | Reserve for replacement % | 0.02 |

### Sheet 4 ("4. Area Schedule") — Building area breakdown
Needs values derived from 1A + estimation rules. See guides/INVESTIGATION_REPORT.md for full cell list.

### Sheet 5 ("5. Key Assumptions") — Only these TRUE input cells
Most of Sheet 5 is formulas cascading from Sheets 1, 4, 7, 13. Only write to:

| Cell | What It Is | Default |
|------|-----------|---------|
| E12 | Land purchase duration (months) | 0 |
| E13 | Pre-development duration | 12 |
| E14 | Construction duration | 18 |
| E16 | Stabilized duration | 0 |
| F15 | Lease-up offset | -3 |
| E37 | Profit percentage | 0.08 (8%) |
| R57 | Dev charge: 1-Bed (<700 SF) | $34,849 (Toronto) |
| R58 | Dev charge: 2-Bed (>=700 SF) | $50,248 (Toronto) |
| R59 | Dev charge: 3-Bed | $47,107 (Toronto) |

**DO NOT write to:** B4, G10, F20-F28, D36, these are all formulas.

**Formula-to-formula rewrite is OK for Altus row routing.** F29 + O48/P48/O49/P49 are formulas in the template, but the populator legitimately rewrites them via `is_formula=True` to point at the user-selected Altus row/column (storey_tier, construction_type, region, parking surface override). Never overwrite with a static value, only swap the formula target.

---

## Development Charges — Toronto Defaults

For Toronto projects, current per-unit charges:
- 1 Bed (< 700 SF): ~$34,000
- 2 Bed (>= 700 SF): ~$50,000
- 3 Bed: ~$47,000

For Phase 1, default to Toronto rates. Other municipalities will require manual lookup in Phase 2.

---

## Success Criteria

Phase 1 is done when:
1. Output .xlsx file opens in Excel without errors or warnings
2. All formula cells are preserved (not overwritten with static values)
3. 1A data correctly appears on Sheet 1 with accurate values
4. Key Assumptions (Sheet 5) updated with correct project-specific values
5. Downstream sheets (2-15) recalculate correctly when opened in Excel
6. Works for both Project A (Birchmount, 3 unit types) and Project B (490 St Clair, 9 unit types consolidated into 3)

---
> Source: [droduque/reverse-1b-tool](https://github.com/droduque/reverse-1b-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
