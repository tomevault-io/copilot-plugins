## serp

> We are building a **Market Intelligence Tool** for a non-profit counselling agency based in Vancouver, BC.

# Project Context: SERP Intelligence & Content Strategy Map

## 1. The Mission

We are building a **Market Intelligence Tool** for a non-profit counselling agency based in Vancouver, BC.
**Our Methodology:** Bowen Family Systems Theory.
**Our Goal:** To map "Problem-Aware" search queries (e.g., "why is my son angry") to "Solution-Aware" content (e.g., "Differentiation of Self").

## 2. The Tech Stack

- **Language:** Python 3.9+
- **Core Library:** `pandas` (Data manipulation)
- **API Provider:** `serpapi` (Library: `google-search-results`)
- **Output:** Excel (`.xlsx`) via `openpyxl`
- **Environment:** Local Virtual Environment (`venv`)

## 3. The "Bridge" Strategy (Critical Context)

All data analysis must serve the **Bridge Strategy**:

1.  **The Symptom (User Search):** What the client feels (e.g., "Walking on eggshells", "Estrangement").
2.  **The Bad Advice (Competitor/PAA):** What Google currently tells them (e.g., "Cut them off", "They are a narcissist").
3.  **The Systemic Solution (Our Content):** The Bowen Theory reframe (e.g., "Emotional Reactivity", "Triangles", "Overfunctioning").

## 4. The Codebase: `serp_audit.py`

The main script performs a Deep SERP Audit.

- **Input:** `keywords.csv` (No headers, one keyword per line) or `SERP_SINGLE_KEYWORD` env override for one-off runs.
- **Output:** `market_analysis_v2.xlsx` (Multi-tab Excel file).

### Key Features & Constraints

- **Force Local Intent:** The script contains a `FORCE_LOCAL_INTENT` boolean. If `True`, it appends the `LOCATION` string to generic keywords to trigger commercial data (Ads/Maps).
- **Rate Limiting:** Must always include `time.sleep()` between API calls to prevent rate-limiting or file-lock errors.
- **Error Handling:** Never crash on a single failed keyword. Log the error and `continue`.
- **Multi-Engine Capture:** Automatically fetches Google Organic, AI Overviews (via secondary call if needed), and Google Maps (for local intent).
- **Auditability:** Every extracted row includes `Run_ID`, `Created_At`, `Google_URL`, and a `Params_Hash` to verify exactly what generated the data.

## 5. Data Dictionary (Output Tabs)

- **`Overview`:** Top 3 Organic Results + Key Metrics.
- **`Organic_Results`:** Full list of organic rankings.
- **`PAA_Questions`:** "People Also Ask" questions. _Use these to find the client's specific pain points._
- **`Related_Searches`:** Standard related searches from the bottom of the SERP.
- **`Derived_Expansions`:** "People Also Search For" (PASF) terms extracted from organic results or knowledge boxes.
- **`Competitors_Ads`:** Paid advertisements and their positioning.
- **`SERP_Language_Patterns`:** N-Gram analysis of snippets, ads, and PASF terms to identify dominant themes and "anxiety loops".
- **`Strategic_Recommendations`:** Automated mapping of SERP patterns to Bowen Theory "Bridge" strategies (Medical Model, Fusion, Resources).
- **`Local_Pack_and_Maps`:** Local business listings from the SERP and Google Maps.
- **`AI_Overview_Citations`:** Citations and sources from AI Overviews.
- **`SERP_Modules`:** Ordered list of all modules detected on the page.
- **`Rich_Features`:** Details on Knowledge Panels, Video Carousels, etc.
- **`Parsing_Warnings`:** Log of any fields that could not be extracted.
- **`AIO_Logs`:** Audit log of AI Overview retrieval attempts.

## 6. Interaction Guidelines for Gemini

When I ask for code updates or data analysis:

1.  **Prioritize "The Bridge":** Always look for opportunities to connect the search term to Bowen Theory concepts.
2.  **Respect the API:** Do not suggest web scraping (BeautifulSoup/Selenium). Stick to `google-search-results`.
3.  **Data Integrity:** Ensure all Pandas DataFrames handle missing values (`NaN`) gracefully before exporting to Excel.
4.  **CSV Formatting:** Remember `keywords.csv` has NO header. Read it with `header=None` unless `SERP_SINGLE_KEYWORD` is set.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dbgnvan2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dbgnvan2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
