## adk-sql-reflection-pattern

> This project implements a robust SQL generation agent using the Google Gen AI Agent Development Kit (ADK) and MCP Toolbox. It features a reflection loop that validates generated SQL using BigQuery's `dry_run` capability before execution, ensuring safety and reliability.

# Project Context: ADK SQL Reflection Pattern

## Overview
This project implements a robust SQL generation agent using the Google Gen AI Agent Development Kit (ADK) and MCP Toolbox. It features a reflection loop that validates generated SQL using BigQuery's `dry_run` capability before execution, ensuring safety and reliability.

## Key Technologies
*   **Language:** Python 3.11+
*   **Framework:** Google ADK (`google-adk`)
*   **Tooling:** MCP Toolbox (`mcp-toolbox`)
*   **Database:** Google BigQuery (specifically `bigquery-public-data.google_trends`)
*   **Dependency Management:** `uv`
*   **Process Management:** `honcho`

## Architecture
The agent is a `SequentialAgent` composed of three main stages:
1.  **Schema Inspector:** Fetches database schema using MCP tools (`list_tables`, `get_table_info`).
2.  **SQL Generator Loop (`LoopAgent`):**
    *   *Generator:* Drafts SQL queries.
    *   *Validator:* Performs a `dry_run` via MCP `execute_sql` tool.
    *   *Reviewer:* Analyzes `dry_run` results and either escalates (success) or provides guidance for the next iteration.
3.  **Final Responder:** Executes the valid SQL and presents results to the user.

## Development & Running
*   **Install Dependencies:** `uv sync`
*   **Run Agent:** `uv run honcho start` (Starts both MCP Toolbox and ADK Web UI)
*   **Run Tests:** `PYTHONPATH=. uv run pytest`
*   **Environment:** Requires `.env` file with `GOOGLE_CLOUD_PROJECT`, `TOOLBOX_HOST`, and `TOOLBOX_PORT`.

## Key Files
*   `agents/sql_agent/agent.py`: Root agent definition.
*   `agents/sql_agent/sql_generator_loop.py`: Core reflection loop logic.
*   `tools.yaml`: MCP Toolbox configuration for BigQuery.
*   `Procfile`: Service definitions for `honcho`.
*   `pyproject.toml`: Python dependencies.

## Critical Implementation Details: Dataplex and BigQuery Location Compatibility

When integrating Google Cloud Dataplex with BigQuery, it is crucial to correctly configure resource locations to avoid deployment errors.

**Problem:**
A `google_dataplex_asset` configured to point to a BigQuery dataset will fail if there is a mismatch in their `location_type`. Specifically, if the Dataplex Lake and Zone are in a single region (e.g., `us-central1`), but the BigQuery dataset is implicitly or explicitly set to a multi-region (e.g., `US`), the asset creation will fail with an error like:
`Error creating Asset: googleapi: Error 400: BigQuery dataset location us-central1 is invalid, allowed regions are {US}.`

This error message can be misleading, as it suggests the BigQuery dataset's single-region location is invalid, when the actual conflict is with the `location_type` configured on the Dataplex Zone.

**Solution:**
Ensure that the `location_type` of the `google_dataplex_zone` resource matches the actual region type of the BigQuery datasets it will manage.

*   If your BigQuery datasets are in a **single region** (e.g., `us-central1`), the `google_dataplex_zone`'s `resource_spec` should have `location_type = "SINGLE_REGION"`.
*   If your BigQuery datasets are in a **multi-region** (e.g., `US`), the `google_dataplex_zone`'s `resource_spec` should have `location_type = "MULTI_REGION"`.

**Example of Correct Configuration (for single-region BigQuery datasets):**

```terraform
resource "google_dataplex_zone" "analytics_curated_zone" {
  # ... other configurations ...
  resource_spec {
    location_type = "SINGLE_REGION" # Must match BigQuery dataset location type
  }
}

resource "google_bigquery_dataset" "my_dataset" {
  # ... other configurations ...
  location = "us-central1" # Single region
}
```

Always verify the explicit location of your BigQuery datasets and ensure the Dataplex Zone's `location_type` is set accordingly.

## Critical Implementation Details: Dataplex Aspect Model & API Usage

Several key learnings about the Dataplex data model and API behavior were discovered during this project. Understanding these is critical to avoiding significant errors.

### 1. The "One Aspect per Type per Entry" Rule

**Core Concept:** A single Dataplex Entry (e.g., a BigQuery table) can only have **one** Aspect of a given Aspect Type attached to it at any time.

**Problem Encountered:** The initial approach attempted to attach metadata for multiple table columns by creating multiple Aspects of the *same* Aspect Type (e.g., `business-rules-aspect-type`) and attaching them all to the same table Entry. This is not supported by the Dataplex data model and was the root cause of many subsequent errors.

**Solution:** To attach metadata to multiple columns within a single table, you must create a **separate and unique `google_dataplex_aspect_type` for each column**.

**Example:** For a `transactions` table, instead of one generic aspect type, create specific ones:
- `google_dataplex_aspect_type "refund-amount-aspect-type"`
- `google_dataplex_aspect_type "txn-ts-aspect-type"`
- `google_dataplex_aspect_type "sale-amount-aspect-type"`

This allows you to attach one of each of these unique aspect types to the `transactions` table Entry, effectively associating metadata with each column.

### 2. Misleading `PermissionDenied` Errors

**Problem Encountered:** During development, persistent `PermissionDenied` errors occurred when trying to update an Entry with new aspects, even when the service account had `owner` permissions.

**Root Cause:** The error was not due to user permissions. When an `UpdateEntry` API call is made, Dataplex re-validates *all* aspects on the entry, including system-managed ones like `schema` and `storage`. An implicit Organization Policy was in place that denied permission to validate these system aspects, causing the entire update operation to fail with a misleading permission error. The actual issue was the API's internal validation process, not the user's action.

**Solution:** This error was a symptom of the incorrect "one aspect per type" approach. By fixing the root cause and creating unique aspect types per column, the `UpdateEntry` call became simpler and no longer triggered the re-validation of system aspects in a way that caused the permission error.

### 3. Naming Conventions for `aspect_type_id`

**Problem Encountered:** Terraform `apply` commands failed with obscure errors when creating `google_dataplex_aspect_type` resources.

**Root Cause:** The `aspect_type_id` field has a strict naming convention. It **only allows lowercase letters, numbers, and hyphens (`-`)**. The initial implementation used underscores (`_`), which are invalid.

**Solution:** Always use kebab-case for `aspect_type_id` values in Terraform.
- **Incorrect:** `aspect_type_id = "business_rules_aspect_type"`
- **Correct:** `aspect_type_id = "business-rules-aspect-type"`

### 4. Correct `UpdateEntryRequest` API Call Structure

**Problem Encountered:** API calls to attach aspects were failing with "Duplicate aspect" errors even after creating unique aspect types.

**Root Cause:** The `UpdateEntryRequest` requires specific parameters to function correctly. The keys in the `aspects` map of the Entry object must be unique and correctly formatted.

**Solution:**
1.  **Use `aspect_keys`:** The `UpdateEntryRequest` must include the `aspect_keys` parameter. This parameter is a list of the unique keys for the aspects you intend to update or attach.
2.  **Unique Aspect Key Format:** The key for each aspect in the `aspects` map must be the full, unique path to the aspect type. The correct format is: `{project_id}.{region}.{aspect_type_id}`.

**Example (Python):**
```python
# This is the key for the aspect in the 'aspects' dictionary
aspect_key = f"{project_id}.{region}.{aspect_type_id}"

# The request must specify which aspect keys are being updated
update_request = {
    "aspects": {
        aspect_key: {"data": {"rule_description": "..."}}
    },
    "aspect_keys": [aspect_key]
}
```
By adhering to these four principles, you can avoid the most common and misleading pitfalls when working with the Dataplex aspect model.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cwest)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cwest)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
