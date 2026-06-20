## tariff-claude-skill

> Digitize tariff document PDFs into structured Excel spreadsheets and CSVs. Use this skill whenever the user uploads a tariff PDF, tariff schedule, customs duty document, harmonized tariff schedule (HTS/HTSUS), trade regulation PDF, Schedule A (Classification of Imports), or any PDF containing tariff rate tables, duty rates, or customs classification data. Also trigger when the user mentions 'tariff digitization', 'tariff table extraction', 'convert tariff PDF', 'tariff spreadsheet', 'Schedule A', 'Classification of Imports', '1950 tariff', '1939 tariff', 'commodity numbers', or asks to extract/digitize tariff table data from a PDF into Excel or CSV format. Even if the user just says 'process this tariff' or 'extract the rates from this PDF', use this skill. This skill handles scanned and text-based tariff PDFs alike, including historical U.S. tariff schedules from 1939 and 1950.


# Tariff Document Digitizer

Convert tariff document PDFs into clean, structured Excel (.xlsx) and CSV files by extracting and reconstructing the tables they contain. Designed for academic research on historical tariff schedules, particularly the U.S. Classification of Imports (Schedule A), but adaptable to any tariff document format.

## Core Principle: Complete Replica

**Extract ALL columns** from the source document, not just a subset. The final spreadsheet should be a complete replica of the input PDF's tabular data. Every column visible in the PDF table should appear in the output.

---

## Training Data

Before starting any extraction, check for bundled training data to understand the baseline output format:

- **`references/training_data.csv`** — A clean, pre-parsed CSV containing correctly extracted tariff data from research. This is your ground truth for description formatting and hierarchy conventions.
- **`references/training_data.pdf`** — The original source PDF that the CSV was extracted from. Use this to understand the kinds of layouts you'll encounter.

At the start of Phase 2, load the CSV training data if available:
```python
import pandas as pd
import os
skill_path = "<skill_path>"
training_csv = os.path.join(skill_path, "references", "training_data.csv")
if os.path.exists(training_csv):
    training_df = pd.read_csv(training_csv)
    print(training_df.head(10))
    print(training_df.columns.tolist())
```

Use the training data to:
1. **Learn the baseline column structure and description conventions.** The training CSV may contain a simplified subset of columns (e.g., `Schedule A Commodity Number`, `Commodity Description`, `Tariff Paragraph`). Source documents typically contain additional columns. **Always extract ALL visible columns from the source document** — the training data represents a baseline, not the complete target schema.
2. **Understand formatting conventions.** Commodity numbers use the format `NNNN NNN` (e.g., "0010 600"). Descriptions use colon-separated hierarchies. Tariff paragraphs can be simple numbers ("701"), compound references ("701, §2491 (c) I.R.C"), or subsection references ("708 (a)").
3. **Validate your extraction.** After extracting data, programmatically compare any overlapping commodity numbers between your output and the training data.

---

## Document Edition Detection

Historical U.S. tariff schedules come in different editions with distinct formats. **Identify the edition first** — it determines column structure, number format, and output schema.

### 1950 Edition
- Commodity numbers in `XXXX XXX` format with a space (e.g., `0010 600`)
- **7 columns** with two separate Rate of Duty sub-columns
- Date printed: "August 1, 1950"

### 1939 Edition
- Commodity numbers use period notation (e.g., `0010.6`, `*0046.45`)
- **6 columns** with a single Rate of Duty column
- Contains Revenue Act references

### Other / Unknown Editions
- Adapt the column schema to whatever the source document contains
- Follow the same hierarchy and extraction principles below

---

## Output Format by Edition

### 1950 Edition Columns (7 columns)

| Column | Width | Description |
|--------|-------|-------------|
| **Schedule A Commodity Number** | 25 | `XXXX XXX` format (e.g. `0010 600`) |
| **Commodity Description** | 120 | Fully-qualified hierarchical description with colon separators |
| **Economic Class** | 15 | The parenthetical class number, e.g. `(2)`, `(4)`, `(9)` |
| **Unit of Quantity** | 20 | e.g. `Lb`, `No`, `Gal`, `Piece; Lb`, `Doz`. Keep the cattle `v` superscript inline (see **Unit of Quantity** below) |
| **1930 Tariff Act (except as noted)** | 40 | Statutory rate, including `(Sec. 336)` notes |
| **Trade Agreement** | 70 | GATT and country-specific concession rates with abbreviations |
| **Tariff Paragraph** | 25 | Including compound references like `701, 2491 (c) I.R.C.` |

### 1939 Edition Columns (6 columns)

| Column | Width | Description |
|--------|-------|-------------|
| **Economic Class** | 15 | Numeric class (e.g. `2`, `4`, `5`) |
| **Schedule A Commodity Number** | 25 | Converted from period to `XXXX XXX` format |
| **Commodity Description** | 120 | Fully-qualified hierarchical description |
| **Unit of Quantity** | 25 | e.g. `Lb.......1`, `No......20`, `Gal.......7`. Keep the cattle `v` superscript inline (see **Unit of Quantity** below) |
| **Rate of Duty** | 60 | Full duty text with country-specific rates separated by semicolons |
| **Tariff Paragraph** | 25 | Including Revenue Act references |

### Other Editions
Define columns dynamically based on the source document's table headers. At minimum extract: commodity number, description, unit of quantity, rate(s) of duty, and tariff paragraph.

---

## Commodity Number Formats

### 1950 Edition
- Already in `XXXX XXX` format with a space
- Strip trailing symbols: `*`, `°`, superscripts, footnote markers from the number itself
- **But preserve annotations in a separate representation** (see Commodity Number Annotations below)

### 1939 Edition
- Uses period notation (e.g., `0010.6`, `*0046.45`, `04.120`)
- **Convert** to `XXXX XXX` format: `0010.6` → `0010 600`, `0046.45` → `0046 450`, `04.120` → `0400 120`
- For glove numbers like `04.120`, `04.121`, `05.120`: pad to 7 digits → `0400 120`, `0400 121`, `0500 120`
- Strip asterisks `*` from numbers

### Commodity Number Annotations

Source documents print superior notations on commodity numbers (e.g., `0010 600 *`, `0023 800 d1`, `0030 500 L5`, `0046 990**ei1`). These annotations reference footnotes, contingent provisions, and legal modifiers. **Always preserve these annotations exactly as printed in the source document.** They carry legal and regulatory meaning that researchers need.

---

## Rate of Duty Columns

### 1950 Edition
The 1950 edition has **two** Rate of Duty sub-columns:
- **1930 Tariff Act (except as noted)** — the statutory rate. Include `(Sec. 336)` where the PDF shows it.
- **Trade Agreement** — GATT rates and country-specific concession rates.

Preserve country abbreviations as shown: Can., Arg., Urug., Para., Mex., U. K., Cuba, Ice., Fin., Switz., Neth., France, Iran, Peru, C. Rica, Belg., Turk.

Use semicolons to separate multiple rates within a cell. Examples:
- `1 1/2c lb. Can., Mex., bound GATT`
- `5c lb. Arg.; 25% min. Arg.`
- `Free, bound Can., GATT`

### 1939 Edition
The 1939 edition has a **single** Rate of Duty column. Separate multiple country rates with semicolons:
- `1c lb.; 3/4c lb. Canada; 0.4c lb. Cuba`
- `7c lb.; 35% min.; 5c lb. France; 25% min. France`

### General Rule
**Preserve duty rate formatting exactly as printed.** Keep fractions as Unicode (½, ¼, ⅛, ⅜, ⅝, ⅞, etc.), keep compound rates intact (e.g., "1¢ lb.+3¢ lb. I. R. C."), keep percentage rates as strings (e.g., "35%", "12½%"), keep "Free" as "Free". Never convert any rates to decimal numbers.

---

## Unit of Quantity

Record the unit-of-quantity code exactly as printed: `Lb.......1`, `No......20`, `Gal.......7` (1939 edition) or `Lb`, `No`, `Gal`, `Doz` (1950 edition). Keep the dot leaders that connect the unit to its reporting code in the 1939 edition.

### The cattle “v” superscript
On the live-cattle entries (commodity numbers `0010 600`–`0010 900`) the weight line carries a small superscript **v** on the `Lb.` symbol, and the row is reported in two units — a `No.` (count) line over a `Lb.ᵛ` (weight) line. **Keep the `v` inline, inside the Unit cell, in the normal flow of the unit text — do NOT split it into a separate column.** Write it with the Unicode superscript character `ᵛ` (U+1D5B), and join the two stacked unit lines in the one cell with `; ` (semicolon-space), in printed top-to-bottom order:

`No......20; Lb.ᵛ......1`

This rule is specific to the cattle `v` mark; it is not a general instruction for other superscript marks on units.

---

## Commodity Description: Hierarchy Flattening

This is the most complex column. The PDF uses indentation to organize commodities hierarchically. **Flatten** the hierarchy into a single string using `: ` (colon-space) as separator.

### Critical Rule: Indentation Determines Hierarchy

**Always look at the actual horizontal indentation level** to determine parent-child relationships. Do NOT assume adjacent items share parents — check where each line starts horizontally in the PDF.

### Rules
- **ALL CAPS section headers** (e.g. "MEAT PRODUCTS", "DAIRY PRODUCTS") are EXCLUDED from descriptions. They reset context.
- **All other ancestor labels** from the indentation hierarchy ARE included, joined by `: `
- Every child row MUST carry its full ancestry chain — no dropping parents
- When a new ALL CAPS header appears, all accumulated parent context resets
- When a continuation header says "—Continued" (e.g., "DAIRY PRODUCTS—Continued"), treat it as the same section — do not include "—Continued" in the hierarchy prefix
- Strip dot leaders, trailing dashes, and line-fill characters
- Preserve abbreviations: `n.s.p.f.`, `n.e.s.`, `n.e.s`
- Preserve references: `(T. D. 46795)`, `(formerly part of 0046 020)`, `(C. D. 610--4/6/42)`
- Preserve the exact wording from the source document at each level. Do not paraphrase or abbreviate.

### Examples

**Two-level:**
`Poultry, live:` → `Turkeys` = `Poultry, live: Turkeys`

**Three-level (section header excluded):**
`MEAT PRODUCTS` (excluded) → `Fresh, chilled, or frozen:` → `Pork:` → `Fresh or chilled` = `Fresh, chilled, or frozen: Pork: Fresh or chilled`

**Deep hierarchy:**
`Fish:` → `Fresh or Frozen:` → `Whole, or beheaded...` → `Fresh-water fish, n.e.s:` → `Whitefish` = `Fish: Fresh or Frozen: Whole, or beheaded, or eviscerated, or both: Fresh-water fish, n.e.s: Whitefish`

**Siblings share parents:**
`Milk and cream:` → `Condensed or evaporated milk:` → `In airtight containers:` → `Unsweetened` AND `Sweetened` both get the full chain.

### Hierarchy Gotcha: Indentation Level Matters

If an item's indentation goes back to the far left (same level as top-level entries), it is standalone even if it appears near nested items. Example from glue section:
```
Glue, glue size, and fish glue:           ← parent
  Valued less than 40 cents per pound:     ← sub-parent
    Glue size and fish glue, n.s.p.f.     ← child: gets BOTH parents
    Glue, animal, n.s.p.f.               ← child: gets BOTH parents
  Valued 40 cents or more per pound        ← child: gets TOP parent only
Casein glue                                ← STANDALONE (back at far left)
```
Result: `Casein glue` — NOT `Glue, glue size, and fish glue: Casein glue`

### Standalone Items (no parent context)
Items at the top indentation level within their group: `Butter`, `Lard`, `Fish paste and fish sauce`, `Turtles`, `Casein glue`, `Ossein`, `Albumen, n.s.p.f.`, `Rennet`, etc.

---

## Section Headers to Exclude

These appear in ALL CAPS/bold and should NOT be part of the commodity description (non-exhaustive — exclude any ALL CAPS group header):
- `ANIMALS, EDIBLE (EXCEPT FOR BREEDING)`
- `MEAT PRODUCTS`
- `ANIMAL OILS AND FATS, EDIBLE`
- `DAIRY PRODUCTS`
- `FISH AND FISH PRODUCTS, EXCEPT SHELLFISH`
- `SHELLFISH AND PRODUCTS`
- `OTHER EDIBLE ANIMAL PRODUCTS`
- `HIDES AND SKINS, RAW (EXCEPT FURS)`
- `LEATHER`
- `LEATHER, RAWHIDE, AND PARCHMENT MANUFACTURES`
- `FURS AND MANUFACTURES`
- `ANIMAL AND FISH OILS, FATS, AND GREASES, INEDIBLE`
- `OTHER INEDIBLE ANIMALS, AND ANIMAL PRODUCTS`

Group-level headers (e.g., "Group 00—ANIMALS AND ANIMAL PRODUCTS, EDIBLE") are also excluded.

---

## Tariff Paragraph Rules

- **Bracket rule:** A tariff paragraph beside a vertical bracket `}` applies to ALL rows the bracket spans
- Preserve suffixes: `(a)`, `(b)`, `(c)`, `(1)`, `(2)`, etc.
- Compound paragraphs: `701, 2491 (c) I.R.C.`, `52, 2491 (a) I.R.C.`
- 1939 Revenue Act references: `701, (Sec. 701) Rev. Act 1936`, `703, (Sec. 301)`

### OCR Misread Warning
Letters `c` and `e` are frequently confused in scans. Cross-reference visually. When a section consistently uses one paragraph (e.g. all "other than bovine" leather = `1530 (c)`), apply consistently.

### Tariff Paragraph Patterns by Section (1950)
- Animals/Cattle: 701, 702, 703, 711, 712
- Meat: 701-706, 1558
- Dairy: 707, 708 (a/b/c), 709, 710
- Fish: 717 (a/b/c), 718 (a/b), 719 (1-5), 720, 721
- Shellfish: 1756, 1761, 1790
- Eggs/edible: 705, 713, 41
- Hides: 1530 (a), 1678, 1681, 1691, 1765
- Leather bovine: 1530 (b)(1-7)
- Leather other: 1530 (c)
- Leather fancy: 1530 (d)
- Footwear: 1530 (e)
- Gloves: 1532 (a/b)
- Luggage: 1529 (a), 1531
- Furs: 1519 (a-e), 1520, 1526 (a), 1681
- Oils/fats: 52, 701, 1730 (b), 2491 (a) I.R.C.
- Other inedible: 714, 715, 1606 (a), 1607, 1682, 1695
- Misc: 19, 41, 69, 1507, 1518, 1533, 1536-1538, 1545, 1556, 1605, 1624, 1627, 1637, 1666, 1677, 1680, 1683, 1689, 1693-1695, 1701, 1709, 1715, 1738, 1741, 1751, 1755, 1780, 1784, 1796, 1799, 1813

---

## Processing Workflow

Execute these four phases in order — do not skip ahead.

### Phase 1: PDF Upload and Assessment

The user uploads a tariff document PDF in chat. Before proceeding:

1. Confirm the file is available at `/mnt/user-data/uploads/` by listing that directory. Also check `/mnt/project/` for any project-level files.
2. Verify it is a PDF (check the extension and run `pdfinfo` on it to get page count and metadata).
3. **Determine if the PDF is scanned or text-based:**
   ```python
   import pdfplumber
   with pdfplumber.open("<file>") as pdf:
       text = pdf.pages[0].extract_text()
       if text and len(text.strip()) > 50:
           print("TEXT-BASED PDF — use pdfplumber extraction in Phase 3")
       else:
           print("SCANNED PDF — use visual/context-window extraction in Phase 3")
   ```
4. **Identify the edition** (1950, 1939, or other) by examining the date, column headers, and number format.
5. Tell the user which file(s) you found, whether each is scanned or text-based, the edition, the page count, and confirm you're ready to begin processing.

If no PDF is present, ask the user to upload one.

### Phase 2: Read and Understand the Document

The goal is to understand the full structure and content of the tariff document before attempting any extraction.

1. **Load the training data** (see Training Data section above) to calibrate your understanding of expected output format.

2. **For text-based PDFs:**
   - Use `pdftotext -f 1 -l 1 <file> - | head -40` to preview the first page.
   - Extract text from all pages using pdfplumber to get a complete picture:
     ```python
     import pdfplumber
     with pdfplumber.open("<file>") as pdf:
         for i, page in enumerate(pdf.pages):
             text = page.extract_text()
             print(f"--- Page {i+1} ---")
             print(text[:500] if text else "[No text found]")
     ```
   - Rasterize a sample page to visually confirm the table layout:
     ```bash
     pdftoppm -jpeg -r 200 -f 1 -l 1 <file> /tmp/tariff-preview
     ```
     Then view the resulting image.

3. **For scanned PDFs (no embedded text):**
   - **Check if page images are already in the conversation context.** When users upload PDFs to Claude, the pages are often rendered as images visible in the context window. If you can see the page content, extract directly from what you see — this is the most efficient approach for scanned documents.
   - **If pages are NOT visible in context**, rasterize them for viewing:
     ```bash
     pdftoppm -jpeg -r 200 -f 1 -l 1 <file> /tmp/tariff-preview
     ```
     Then view the resulting image(s) to understand the layout.

4. **Identify the table structure.** Look for:
   - Column headers (e.g., Schedule A Commodity Number, Commodity Description and Economic Class, Unit of Quantity, Rate of Duty — 1930 Tariff Act, Trade Agreement, Tariff Paragraph)
   - Whether the document uses a single table layout or multiple formats across pages
   - Header rows that span multiple lines or use hierarchical indentation
   - Footnotes at the bottom of pages — note their numbering scheme and content
   - Superior notations on commodity numbers (*, d, e, i, j, k, L, o, b, etc.)
   - Whether the PDF covers a single group or spans multiple groups
   - Vertical brackets `}` that group rows under a single tariff paragraph

5. **Plan the column schema.** Based on the edition and what you see, select the appropriate column set from the Output Format by Edition section above. If the document doesn't match a known edition, define columns to match whatever the source document contains.

Document your findings before moving to Phase 3.

### Phase 3: Reconstruct the Tables

This is the core extraction phase. The goal is to produce clean, structured tabular data.

#### Path A: Text-based PDFs

1. **Use pdfplumber's table extraction first:**
   ```python
   import pdfplumber
   all_tables = []
   with pdfplumber.open("<file>") as pdf:
       for i, page in enumerate(pdf.pages):
           tables = page.extract_tables()
           for table in tables:
               all_tables.extend(table)
   ```

2. **Validate the extraction** by comparing a sample of extracted rows against the rasterized page image. Common issues to watch for and fix:
   - Merged header cells split incorrectly — reconstruct multi-level headers into a single header row
   - Indented description rows that indicate sub-categories — build the full hierarchy (see Commodity Description section)
   - Cells that span multiple rows (e.g., a single heading covering many sub-items) — fill down the parent value
   - Missing or misaligned columns — realign by checking column positions
   - Special characters or encoding issues — clean up Unicode artifacts

3. **If pdfplumber produces poor results**, fall back to Path B.

#### Path B: Scanned PDFs or failed text extraction

1. **If page images are visible in the conversation context**, read them directly and build the data row by row in Python. This is the primary method for scanned historical documents.

2. **If page images are NOT in context**, rasterize each page at 200 DPI, then view each image and extract the data:
   ```bash
   pdftoppm -jpeg -r 200 <file> /tmp/tariff-pages
   ```

3. **Build the extraction systematically** — process page by page, maintaining the current hierarchy context (which section headers are active) as you go.

#### For both paths — Data cleaning rules:

4. **Clean the data:**
   - Strip leading/trailing whitespace from all cells.
   - **Preserve duty rate formatting exactly as printed** (see Rate of Duty Columns section).
   - **Preserve commodity numbers with all annotations** (see Commodity Number Annotations section).
   - **Convert 1939-format numbers** to `XXXX XXX` format (see Commodity Number Formats section).
   - Remove page headers/footers that got mixed into table rows (e.g., "CLASSIFICATION OF IMPORTS", "August 1, 1950", page numbers).
   - Handle continuation rows (where a description spans multiple lines) by joining them into a single description.
   - Build description hierarchies per the rules in the Commodity Description section.

5. **Collect footnotes.** As you process each page, capture any footnotes printed at the bottom. Store them with their page number and footnote marker for the Footnotes sheet.

6. **Structure the output** as a pandas DataFrame with clear column headers derived from the original document.

#### Batching for large documents

For documents over approximately 30 pages, process in batches (e.g., 10 pages at a time), accumulating data into a single list, then build the DataFrame at the end. This prevents context overflow and tool timeouts.

### Phase 4: Output Files

Every extraction produces **four output files**. All four are required — no exceptions.

#### 1. Excel workbook (.xlsx)

Create using openpyxl with two or three sheets:

**"Tariff Data" sheet (or "Sheet2" if user requests legacy format):**

```python
from openpyxl import Workbook
from openpyxl.styles import Font, Alignment, PatternFill, Border, Side

wb = Workbook()
ws = wb.active
ws.title = "Tariff Data"
```

Apply professional formatting:
- Bold header row with dark blue fill (RGB: 2F5496) and white text
- Freeze the top row so headers stay visible when scrolling
- Use Arial 10pt throughout
- Apply thin borders to all data cells
- Left-align description and rate columns, center-align commodity numbers, units, and tariff paragraphs
- Set column widths appropriate to content and edition (see Output Format by Edition)

**"Metadata" sheet** (required):
Include these fields:

| Field | Description |
|-------|-------------|
| Source Document | Original filename |
| Document Title | From the document header if identifiable |
| Edition | 1950, 1939, or other |
| Groups Covered | Which tariff groups are in the document |
| Effective Date | From the document if printed (e.g., "August 1, 1950") |
| Pages Processed | Total page count |
| Total Rows Extracted | Row count of tariff data |
| Date of Extraction | Current timestamp |
| Extraction Method | "Visual inspection of scanned PDF" or "pdfplumber text extraction" |
| Column Definitions | One row per column explaining what it contains |
| Notes | Any caveats about data quality or difficult pages |

**"Footnotes" sheet** (include if any footnotes were found):
Columns: `Page Number`, `Footnote Marker`, `Footnote Text`. Capture all footnotes from the bottom of each page. These contain legally significant information about contingent provisions, trade agreement references, and definitional notes.

#### 2. Tariff data CSV (.csv)

Export the tariff data (same content as the "Tariff Data" sheet) as a UTF-8-with-BOM CSV:
```python
df.to_csv(csv_path, index=False, encoding='utf-8-sig')
```

#### 3. Metadata CSV (.csv)

Export the metadata as a separate CSV file with two columns (`Field`, `Value`):
```python
meta_rows = [
    ["Source Document", filename],
    ["Edition", edition],
    ["Pages Processed", str(page_count)],
    ["Total Rows Extracted", str(len(rows))],
    ["Date of Extraction", timestamp],
    # ... all other metadata fields
]
meta_df = pd.DataFrame(meta_rows, columns=["Field", "Value"])
meta_df.to_csv(meta_csv_path, index=False, encoding='utf-8-sig')
```

#### 4. Source document copy (.pdf)

Copy the original source PDF to the outputs directory so the user has the source document alongside the extraction:
```python
import shutil
shutil.copy(source_pdf_path, "/mnt/user-data/outputs/<source-filename>.pdf")
```

#### Naming convention and delivery

Save all files to `/mnt/user-data/outputs/` using a consistent naming convention:
- `<document-name>_Tariff_Data.xlsx`
- `<document-name>_Tariff_Data.csv`
- `<document-name>_Metadata.csv`
- `<document-name>_Source.pdf`

Use `present_files` to share all four files with the user. Provide a brief summary: how many rows were extracted, how many pages were processed, which groups/sections were covered, and any caveats about data quality.

---

## Edition Differences Quick Reference

| Feature | 1939 Edition | 1950 Edition |
|---------|-------------|-------------|
| Number format | Periods (e.g., `0010.6`) | Spaces (e.g., `0010 600`) |
| Columns | 6 (single duty column) | 7 (split duty columns) |
| Combined entries | Combines what 1950 splits | More granular |
| Footwear | Separates Children's/Infants' | Combines |
| Revenue Act refs | Yes | No |
| Glove numbering | `04.1201` | `0401 120` |

---

## Important Guidelines

- **Accuracy over speed.** Tariff data is used for academic research and legal analysis. Double-check a sample of extracted values against the source PDF before delivering. If you're uncertain about any values, flag them in a "Notes" column or in the metadata.
- **Preserve original formatting of codes and rates exactly as printed.** Duty rates, commodity codes, and unit descriptions should appear in the spreadsheet character-for-character as they do in the source document. Never convert percentage strings to decimal numbers, never strip fraction characters, never reformat tariff codes.
- **Preserve all superior notations on commodity numbers.** Annotations like *, d1, e11, L5, b, o, i, j, k carry legal meaning.
- **Preserve the cattle unit `v` superscript inline.** On the live-cattle rows the `Lb.` line carries a superscript `v`; keep it inside the Unit cell as a Unicode superscript and combined with the count line (`No......20; Lb.ᵛ......1`), never as a separate column.
- **Build complete description hierarchies.** Every commodity description should be prefixed with its full hierarchy of parent section headers, separated by colons. See the Commodity Description section for rules.
- **Handle multi-page tables gracefully.** Many tariff tables span dozens or hundreds of pages. Make sure continuation pages are appended correctly without duplicating headers. Maintain hierarchy context across page breaks.
- **Always produce all four output files.** XLSX, Tariff Data CSV, Metadata CSV, and Source PDF copy. No exceptions.
- **When in doubt, rasterize and look.** If text extraction is giving ambiguous results, viewing the actual page image will clarify the correct interpretation.

---
> Source: [SU-OSPO/tariff-claude-skill](https://github.com/SU-OSPO/tariff-claude-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
