## ir-project

> This project is a **Web Intelligence Data Pipeline** developed for the **CS313x – Information Retrieval & Data Analysis** course. It implements a data collection, processing, and analysis pipeline designed to collect, store, and understand web-based textual data.

# GEMINI.md

## Project Overview
This project is a **Web Intelligence Data Pipeline** developed for the **CS313x – Information Retrieval & Data Analysis** course. It implements a data collection, processing, and analysis pipeline designed to collect, store, and understand web-based textual data.

**Core Pipeline:**
1.  **Scraping:** Multi-page collection of 50–200 records (respecting `robots.txt`).
2.  **Storage:** Structured data storage in `data.json`.
3.  **Processing:** Tokenization, normalization, and stopword removal to produce `processed.json`.
4.  **Analysis:** Keyword frequency and statistical dataset exploration.

## Building and Running
The implementation follows a sequential execution flow as defined in `EXCUTION.md`:

1.  **Scrape Data:** `python scraper.py`
2.  **Process Data:** `python process.py`
3.  **Run Analytics:** `python analysis.py`

*Note: The project is focused on the data pipeline requirements (1-4) of the course.*

## Development Conventions
- **Minimalism:** Adhere to a "no over-engineering" philosophy; focus on satisfying course requirements 1-4 with clean, functional code.
- **Standardized Schema:** Ensure `data.json` and `processed.json` maintain consistent fields (e.g., `title`, `content`, `url`).
- **Data Integrity:** Implement robust handling for missing or noisy data during the processing layer.

---
> Source: [wildkemo/ir-project](https://github.com/wildkemo/ir-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
