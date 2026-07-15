## pacioli

> **Pacioli** is a personal finance tracking application named after Luca Pacioli, the father of accounting. It leverages Open Banking APIs (via GoCardless) to fetch transaction data and uses a hybrid approach (Regex + Agentic AI) to categorize spending.

# Pacioli Project Context

## Overview
**Pacioli** is a personal finance tracking application named after Luca Pacioli, the father of accounting. It leverages Open Banking APIs (via GoCardless) to fetch transaction data and uses a hybrid approach (Regex + Agentic AI) to categorize spending.

The core philosophy is **immutable raw data** combined with **derived state**. The raw JSON responses from the bank are stored permanently, and all categorization and analysis are computed on top of this ground truth.

## Architecture

### Data Pipeline
1.  **Ingestion (`update_transactions.py`):**
    *   Fetches data from GoCardless API.
    *   Writes **immutable** JSON files to `raw/<account_id>/<date>.json`.
    *   Uses an **overlapping fetch window** to ensure late-settling transactions are captured.
    *   **Idempotent:** Uses exclusive file creation (`x` mode) to prevent overwriting existing data.

2.  **Loading (`transaction_loader.py`):**
    *   Reads all JSON files from `raw/`.
    *   **Deduplicates** transactions based on `internalTransactionId`.
    *   Returns a flat list of unique `Transaction` objects.

3.  **Enrichment (`transaction_manager.py`):**
    *   Resolves categories using a priority hierarchy (`SOURCE_PRIORITY`):
        1.  **Manual Overrides** (`data/manual_assignments.json`)
        2.  **Transfer Matching** (paired inter-account transactions)
        3.  **Zero Amount Checks** (Ignored/Excluded)
        4.  **Regex Patterns** (`data/patterns.json`)
        5.  **Agent/AI Classification** (`data/llm_cache.json`)
    *   Outputs a flat **Polars DataFrame**.

## Data Schema & Constraints

### `data/patterns.json`
- **Format**: A dictionary where keys are the **Master Category List** and values are lists of pattern objects. 
- **Integrity**: This is the single source of truth for categorization. Every category used in the system **must** exist as a key here.
- **Validation**: Entries are validated with pydantic at load time (`PatternRule` in `transaction_manager.py`). A missing/empty/uncompilable `pattern`, unknown `field`, non-numeric bound, or unparseable time fails loudly with the offending category named — fix the entry rather than working around the error.
- **Fields (per object)**:
    - `pattern`: A regex string (applied case-insensitively).
    - `field`: (Optional) The transaction field to search. Defaults to `counterparty`. Valid values: `counterparty`, `remittance`, or `any` (both).
    - `clean_name`: The "human-friendly" merchant or entity name.
        - **Entity-First Philosophy**: This field MUST represent a specific entity (e.g., "Waitrose", "Uber", "British Gas"). Do NOT use groupings or categories (e.g., "Groceries", "Taxis", "Utility") as the name.
    - `min_amount`: (Optional) Minimum absolute amount to match.
    - `max_amount`: (Optional) Maximum absolute amount to match.
    - `min_day`: (Optional) Minimum day of month (1-31) to match.
    - `max_day`: (Optional) Maximum day of month (1-31) to match.
    - `min_time`: (Optional) Minimum time of day (ISO format, e.g., "11:30") to match.
    - `max_time`: (Optional) Maximum time of day (ISO format, e.g., "15:00") to match.

### `data/manual_assignments.json`
- **Format**: A dictionary mapping `internalTransactionId` to an object with `clean_name` and `category`.
- **Purpose**: Use for "one-off" transactions or outliers that don't warrant a recurring regex pattern.
- **Priority**: This is the **highest priority** source. It overrides Patterns, Transfers, and AI Cache.
- **Schema**:
    ```json
    {
      "tx_id": {
        "clean_name": "Merchant Name",
        "category": "Category > Subcategory"
      }
    }
    ```

### `data/llm_cache.json`
- **Format**: A dictionary caching classification results to avoid redundant research.
- **Integrity**: Entries can be updated by the agent using the `ops` skill. Mark decisions with `source: "AI_AGENT"`.

## Privacy & Security Guardrails

### 1. Data Classification
*   **Sensitive (Level 1)**: API Keys, Secret IDs, Tokens, and Personal Identifiers (`.env`, `token.json`, `TRANSFER_NAME`). **NEVER READ ALOUD OR COMMIT.**
*   **Personal (Level 2)**: Transaction history, raw JSON, CSV exports, account IDs, IBANs, real names. These live in `raw/`, `data/`, and `.csv` files.
*   **Configuration (Level 3)**: Regex patterns, category lists, logic. These are safe to share/commit *if* they don't contain hardcoded Level 2 data.

### 2. Pattern Design Mandates
*   **Disjoint Patterns**: All regex patterns in `data/patterns.json` MUST be disjoint (mutually exclusive) for any given transaction. 
  - The system picks the *first* matching pattern, which creates ambiguity if they overlap.
  - Use `min_amount`, `max_amount`, `min_time`, or `max_time` constraints to ensure that broad "catch-all" patterns do not overlap with specific "constrained" patterns.
  - Never rely on "fallback" behavior where a less-specific pattern is intended to catch misses from a more-specific one without explicit exclusion criteria in the broad pattern.

### 3. Standing Orders for AI Agents
*   **Custom Heuristics**: Always consult @data/ai_instructions.md for project-specific naming philosophies, meal timing, and personal schedule context before performing enrichment or labeling.
*   **Grounded Categorization**: Before labeling a transaction, directly read `data/patterns.json` and `data/manual_assignments.json` to ensure consistency with existing rules.
*   **Grep, Don't Read**: When inspecting large files in Level 2 directories, always use grep-style search with specific patterns rather than reading whole files, to minimize exposure of irrelevant PII.
*   **Scrub Before Commit**: If you are asked to create a new test or documentation example, **generate fake data**. Never copy-paste a real transaction ID or counterparty string into a tracked file.
*   **Anonymization**: If you see a real name (e.g., "SMITH") or an account number in a string you are processing, replace it with a placeholder like `[USER]` or `[ACCOUNT_ID]` if that string is intended for a non-ignored file.
*   **Pre-Commit Check**: Before performing a `git add`, scan the content for things that look like Level 1 or Level 2 data. If found, warn the user and stop.

### 4. File System Protection
*   The `.gitignore` is the primary line of defense. Ensure it always covers `raw/`, `data/`, and `*.csv`.
*   If you create a new data-storing file, immediately verify if it falls under an existing ignore rule or needs a new one.

## Large File Handling
Some files in this project (e.g., `enriched_transactions.csv`, `llm_cache.json`) can grow very large. **Do not attempt to read these files entirely.**

Instead, use targeted shell commands or paginated tool calls to inspect subsets:
- **Search CSV/JSON**: `grep "pattern" enriched_transactions.csv | head -20` (e.g., a transaction ID or merchant name).
- **Inspect CSV structure**: `head -10 enriched_transactions.csv`.
- **Paginated reading**: use your file-reading tool's offset/limit parameters rather than reading the whole file.

### Configuration
*   **Environment:** managed via `.env`.
*   **Data Storage:**
    *   `raw/`: Raw bank API dumps. **Do not modify manually.**
    *   `data/`: Configuration and cache files (patterns, categories, LLM cache).

## Key Files
*   **`update_transactions.py`**: The primary script to sync with the bank. Safe to run repeatedly.
*   **`transaction_manager.py`**: Contains the `TransactionManager` class which handles the business logic for categorization and enrichment.
*   **`go_cardless_client.py`**: A custom wrapper around the GoCardless Bank Account Data API. Handles token management (`token.json`).
*   **`transaction_loader.py`**: Helper module to load and deduplicate raw data.
*   **`find_uncategorized.py`**: Identifies gaps in categorization for the agent to resolve.
*   **`update_llm_cache.py`**: Records agent-led decisions into the cache.

## Transaction Manager Actions
The `TransactionManager` class in `transaction_manager.py` provides the following core actions:

*   **`enrich_transactions(transactions)`**: The primary pipeline. Takes a list of raw transactions and returns a categorized Polars DataFrame, applying the hierarchy (Manual > Transfer > Zero > Pattern > AI/Agent).
*   **`detect_transfers(transactions)`**: Scans for matching transaction pairs (opposite amounts, nearby dates, user name in description) and marks them as "Internal Transfers".
*   **`test_pattern(transactions, pattern, field)`**: Dry-run a regex pattern against a set of transactions to see what it would match before saving it.
*   **`purge_override_cache(transactions)`**: Optimizes storage by removing cache entries for transactions that are now covered by more deterministic rules (Manual, Pattern, etc.).
*   **`explain_transaction(tx)`**: Provides a detailed diagnostic trace of how a specific transaction would be resolved, showing all matching rules and the final selection.

## Setup & Usage

### Prerequisites
*   Python 3.14+
*   `uv` (Universal Python Package Installer)
*   GoCardless Account (Bank Account Data API)

### Environment Variables (`.env`)
```toml
GOCARDLESS_SECRET_ID = "..."
GOCARDLESS_SECRET_KEY = "..."
TRANSFER_NAME = "..." # Your name as it appears in bank transfers (e.g. "SMITH")
```

### Common Commands
All commands should be run using `uv` to ensure the correct environment and dependencies are used.

*   **Sync Data:**
    ```bash
    uv run update_transactions.py
    ```
*   **Enrich Transactions:**
    ```bash
    uv run enrich_transactions.py
    ```
*   **Visualize Spending:**
    ```bash
    uv run generate_spending_viz.py
    ```
*   **Run Tests:**
    ```bash
    uv run pytest
    ```
*   **Linting & Formatting:**
    ```bash
    uv run ruff check .
    ```
*   **Type Checking:**
    ```bash
    uv run mypy .
    ```

## Development Conventions

*   **Security First:** NEVER commit personal data, transaction history, or sensitive configuration files. Files in `raw/`, `data/`, and all `.csv` files are explicitly ignored in `.gitignore`. If you modify the structure of these files, ensure you are testing with mock data or non-sensitive samples.

*   **Data Integrity:** Never modify files in `raw/` manually.

 If data needs to be fixed, use the `TransactionManager` to create a manual assignment or pattern override.
*   **Type Safety:** Uses `pydantic` to validate `data/patterns.json` at load and `mypy` for static analysis.
*   **Data Frames (Polars):**
    *   Import convention: `from polars import col as C`
    *   Column selection: Use property access `C.column_name` instead of function call `C("column_name")` whenever possible.
*   **Testing:** `pytest` is used for unit tests. Tests are located in `tests/`; `uv run pytest` works directly (`pythonpath` is configured in `pyproject.toml`).
*   **Comments:** do not add comments which are just restating what the code is doing. Only add comments that explain _why_, and document assumptions and why the assumptions are justified.

## Ops Skill
The project includes a specialized `ops` skill for managing the transaction pipeline.

**Location:** `skills/ops/SKILL.md` (symlinked into `.gemini/skills`, `.claude/skills`, and `.agents/skills` so every harness sees the same source of truth)

**Core Workflows:**
1.  **Sync Transactions:** `uv run update_transactions.py` - Fetches new data from GoCardless.
2.  **Enrich Transactions:** `uv run enrich_transactions.py` - Loads, deduplicates, and categorizes transactions.
3.  **Agent-Led Categorization:** The agent identifies gaps using `find_uncategorized.py` and resolves them using `update_llm_cache.py` after grounding research.
4.  **Prune Cache:** `uv run prune_cache.py <tx_id> ...` - Removes specific entries to force re-evaluation.

---
> Source: [genneth/pacioli](https://github.com/genneth/pacioli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
