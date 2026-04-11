## gemini-cli-bq-poc

> This directory is a playground for using the `gemini-cli` tool, with a specific focus on interacting with Google BigQuery. It contains the necessary setup instructions and context for using `gemini-cli` to perform data analytics tasks on BigQuery.

## Directory Overview

This directory is a playground for using the `gemini-cli` tool, with a specific focus on interacting with Google BigQuery. It contains the necessary setup instructions and context for using `gemini-cli` to perform data analytics tasks on BigQuery.

## Usage

The primary purpose of this directory is to provide a configured environment for running the `gemini` command-line tool.

To find information about Mozilla data infrastructure and data in BigQuery use [docs.telemetry.mozilla.org](https://docs.telemetry.mozilla.org/).

Use Glean Dictionary at https://dictionary.telemetry.mozilla.org/ to find information about existing telemetry probes, metrics, events, pings, and their definitions.

Use bigquery-data-analytics extension to search and analyze data in BigQuery. When searching, prefer `mozdata` project which contains user-facing views over `moz-fx-data-shared-prod` which contains raw tables.
Most tables are partitioned by day and require a date filter in the query. Also use `sample_id` filter to limit the number of rows processed.

## BigQuery Table Recommendations

When analyzing Firefox on iOS data, prefer using the following derived tables for better efficiency and simplicity:

*   **New iOS Profiles:** Use `mozdata.firefox_ios.new_profile_clients` to identify new profiles.
*   **Default Browser Status (iOS):** Use `mozdata.firefox_ios.baseline_clients_daily` to get the default browser status.
*   **Mobile Search Metrics:** Use `mozdata.search.mobile_search_clients_daily` for mobile search data.

In general, it's a good practice to look for derived, aggregated tables (often with `_daily` or `_clients` suffixes) before resorting to raw event or metric tables.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akkomar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akkomar)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
