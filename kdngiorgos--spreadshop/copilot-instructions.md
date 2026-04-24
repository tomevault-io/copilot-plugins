## spreadshop

> Spreadshop is a **market analysis PoC for Greek resellers**. It ingests supplier product catalogs (XLSX/PDF), scrapes Skroutz.gr marketplace for current selling prices, and produces a profitability report identifying which products to stock.

# Spreadshop — Claude Code Project Context

## What This Is

Spreadshop is a **market analysis PoC for Greek resellers**. It ingests supplier product catalogs (XLSX/PDF), scrapes Skroutz.gr marketplace for current selling prices, and produces a profitability report identifying which products to stock.

Target audience: resellers operating in the Greek market (Skroutz.gr is the dominant Greek price-comparison marketplace).

---

## Running the App

```bash
# Install deps (first time only)
pip install -r requirements.txt

# Launch
streamlit run app.py
# → http://localhost:8501
```

**Or use the deploy command:** `/deploy`

### App workflow (5 tabs)

1. **Import** — upload `.xlsx` or `.pdf` supplier files → auto-parsed, summary shown
2. **Products** — browse the full parsed catalog with filters
3. **Scrape** — launch concurrent async tasks using httpx to search Skroutz JSON endpoints for each product; pause/resume/stop controls; results cached
4. **Dashboard** — overview of scraped results
5. **Analysis** — KPI cards + sortable table with margins, competition, opportunity scores; download XLSX report

---

## Proving It Works Without a Browser

```bash
python scripts/demo.py
```

Parses all 3 test files, validates price ratios, runs analysis with mock Skroutz data, exports a sample report to `reports/demo_report.xlsx`. No browser required. Expected output: 207 unique products, 2 parse errors.

---

## Architecture

```
app.py                      Streamlit UI (5 tabs, session state, threading)
parsers/
  base.py                   ProductRecord, SkroutzResult, ParseError dataclasses + parse_file() dispatcher
  xlsx_parser.py            Bio Tonics XLSX (openpyxl, handles category header rows)
  pdf_biotonics.py          Bio Tonics PDF (pdfplumber, comma-decimal prices)
  pdf_viogenesis.py         VioGenesis PDF (pdfplumber + 3-pattern garble decoder)
scraper/
  skroutz.py                httpx async JSON endpoint client, concurrent scraping
  cache.py                  JSON file cache (24h TTL, keyed by barcode)
analysis/
  compare.py                Margin %, opportunity score (0-100), recommendations
  export.py                 Multi-sheet XLSX report with openpyxl styling
scripts/
  demo.py                   Proof-of-concept run: all parsers + mock analysis + export
.streamlit/
  config.toml               Dark theme (#0F1117 bg, #6366F1 indigo accent, monospace font)
cache/                      Auto-created: skroutz_cache.json + upload temp files
reports/                    Auto-created: generated XLSX reports
```

---

## Supplier File Formats

### Bio Tonics — XLSX (`parsers/xlsx_parser.py`)

Sheet name: `Table 1`. Columns:

| Col | Header | Field |
|-----|--------|-------|
| A | ΚΩΔΙΚΟΣ | `code` |
| B | ΠΕΡΙΓΡΑΦΗ | `name` |
| C | ΧΤ | `wholesale_price` (numeric float) |
| D | ΠΛΤ | `retail_price` (numeric float) |
| E | BARCODE | `barcode` (int cast to str) |

Category header rows: text only in col A, C/D/E empty. These set `current_category` for subsequent rows and are not parsed as products.

### Bio Tonics — PDF (`parsers/pdf_biotonics.py`)

Same layout as XLSX but extracted via pdfplumber. Prices are comma-decimal strings (`"7,02"` → `7.02`). One known garbled row (price bleeds into name column) — handled by `_parse_price_str()` which regexes `(\d+)[,.](\d{1,2})` from the cell value.

### VioGenesis — PDF (`parsers/pdf_viogenesis.py`)

Complex garbled format. The PDF was exported with overlapping column layout causing text from adjacent cells to be interleaved. Column mapping (0-indexed):

| Index | Header | Field |
|-------|--------|-------|
| 0 | BARCODE | `barcode` |
| 1 | ΚΩΔ. | `code` |
| 2 | ΠΡΟΪΟΝ | `name` ← **clean, not garbled** |
| 6 | ΚΑΤΗΓΟΡΙΕΣ | `category` (pipe-separated, take first) |
| 9 | Χ.Τ. (24%) | `wholesale_price` ← **garbled** |
| 10 | Λ.Τ. | `retail_price` ← **garbled** |

---

## VioGenesis Price Garble — CRITICAL

The price columns are garbled by interleaved characters from adjacent PDF columns. Three regex patterns decode them. **Do not simplify these without re-testing against the PDF.**

### XT (wholesale) patterns

**Pattern: Main** — covers ~70 products
Raw: `'iasd.gsr2//22w,400p1 €'`  → `22.40`
Encoding: `gsr{d1}//{d2}digits_w,{cents}` → price = `d1 + first_digit(d2) + "." + cents`
Regex: `gsr(\d)//(\d)\d*\w,(\d{2})` → `f"{g1}{g2}.{g3}"`

**Pattern: Single-digit** — covers ~11 products (cheap supplements)
Raw: `'iasd.gsr/9/2,w608p2 €'` → `9.60`
Encoding: `gsr/{d1}/digits,w{cents}` → price = `d1 + "." + cents` (d1 is the full integer)
Regex: `gsr/(\d)/\d+,\w?(\d{2})` → `f"{g1}.{g2}"`

**Pattern: Ads variant** — covers ~2 products (drink powders, high-price items)
Raw: `'ads3/52,0071'` → `35.00`
Encoding: `ads{d1}/{d2}digits,{cents}` → price = `d1 + first_digit(d2) + "." + cents`
Regex: `ads(\d?)/(\d)\d*,(\d{2})` → `f"{g1}{g2}.{g3}"`

### RT (retail) pattern
Raw: `'/eVnito/3Gu7p,e5nl0o e€'` → `37.50`
Strategy: extract all digit characters from the string; the first 4 individual digits form the price as `d1d2.d3d4`.

### Validation
All extracted prices must satisfy: `0.48 ≤ wholesale/retail ≤ 0.78` (typical supplier ratio ~0.595). Rows outside this range are flagged as `ParseError` and skipped.

---

## Skroutz Scraping

### URL pattern
We now use the fast and much less restrictive JSON endpoints for Skroutz:
```text
https://www.skroutz.gr/search.json?keyphrase={product_name}
# Fallback: search by barcode EAN
https://www.skroutz.gr/search.json?keyphrase={barcode}
```

Since we use JSON endpoints, browser automation is no longer required. `httpx.AsyncClient` is used to scrape extremely fast concurrently via `asyncio`.

Example JSON endpoint usage:
When querying `https://www.skroutz.gr/search.json?keyphrase=xiaomi15`, the JSON might return:
```json
{"redirectUrl":"https://www.skroutz.gr/c/40/kinhta-thlefwna/.../Xiaomi-15.html?o=xiaomi15"}
```
If `redirectUrl` is present, the script automatically follows it by replacing `.html` with `.json` to get the list of SKUs.

Important JSON response fields used by the parser:
- `skus`: List of products.
- `name`: Product name.
- `sku_url`: Product url.
- `price`: Lowest price string (e.g. "1.347,78 €").
- `shop_count`: Number of shops selling the item.
- `review_score`: Rating string (e.g. "4,7").
- `reviews_count`: Number of reviews.
- `id`: SKU id — used for the second-step `filter_products.json` call.

#### Second-step: `filter_products.json`
Once a SKU id is found, the scraper fetches:
```
https://www.skroutz.gr/s/{skroutz_id}/filter_products.json
```
This returns `product_cards` with per-variant prices (`final_price`) and an accurate `shop_count`. The scraper uses this to set `lowest_price` and `highest_price` on the result.

### `SkroutzScraper` config

| Param | Default | Notes |
|-------|---------|-------|
| `delay` | `0.1s` | Base delay between requests |
| `delay_jitter` | `0.1s` | Random ±jitter added to delay |
| UA | Chrome 124 / Win10 | Set in `scraper/skroutz.py` `_UA` constant |

### Result types

The scraper handles JSON responses by fuzz-matching the results with the query and selecting the best match (`_parse_search_results_json`). Unrecognized formats will yield a `SkroutzResult(found=False)`.

### Cache

File: `cache/skroutz_cache.json`. Key: barcode (preferred) or product name (truncated). TTL: 24 hours. All 200 products take ~1-2 minutes to scrape from scratch (async concurrent, 5 workers, 0.1s delay); the cache makes subsequent runs instant.

---

## Analysis Logic (`analysis/compare.py`)

### Metrics per product

```python
margin_absolute  = skroutz_lowest - wholesale_price
margin_pct       = (margin_absolute / wholesale_price) * 100
competition_level = "Low"    # < 5 shops
                  = "Medium" # 5–15 shops
                  = "High"   # > 15 shops
```

### Opportunity score (0–100)

```
margin_score  = min(margin_pct / 100, 1.0) * 50   # 50% weight — higher margin = better
comp_score    = max(0, (30 - shop_count) / 30) * 30  # 30% weight — fewer shops = better
demand_score  = min(review_count / 100, 1.0) * 20  # 20% weight — more reviews = higher demand
score         = margin_score + comp_score + demand_score
```

### Recommendation thresholds

| Label | Condition |
|-------|-----------|
| `strong_buy` | margin ≥ 30% AND shops ≤ 10 |
| `consider` | margin ≥ 15% |
| `skip` | margin < 15% OR not profitable |
| `not_found` | product not on Skroutz |

---

## Adding a New Supplier

1. Create `parsers/pdf_newsupplier.py` modeled after `pdf_biotonics.py`
2. Use pdfplumber to extract tables; identify column indices for: barcode, code, name, wholesale, retail, category
3. Add detection in `parsers/base.py` `parse_file()`:
   ```python
   if "suppliername" in name_lower:
       from .pdf_newsupplier import parse_newsupplier_pdf
       return parse_newsupplier_pdf(path)
   ```
4. Verify: `python scripts/demo.py` (add the new file to `TEST_FILES`)
5. Check price ratios are in range and product count matches expectations

---

## Key Technical Decisions

| Decision | Why |
|----------|-----|
| httpx AsyncClient | Used to query JSON endpoints efficiently concurrently, bypassing the need for a browser completely. |
| JSON cache (not SQLite) | Zero dependencies, human-readable for debugging, sufficient for <1000 products |
| Monospace font theme | Matches the data-dense B2B SaaS aesthetic; makes price columns easier to scan |
| Thread for scraping | Streamlit reruns the entire script on every interaction; the scraper must run in a daemon thread with results written to `session_state` |
| Barcode as cache key | More stable than product name (names may vary slightly across files/searches) |
| VioGenesis PDF garble | The supplier's PDF was generated with a broken column layout — the 3 regex patterns are the only reliable extraction method; do not replace with a generic digit extractor |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kdngiorgos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
