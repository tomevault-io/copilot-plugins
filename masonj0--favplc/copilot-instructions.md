## favplc

> The absolute highest priority for the Fortuna engine is **steady, reliable fetching**.

# Jules Agent Instructions

## Mission-Critical Priority: Continuous Fetching

The absolute highest priority for the Fortuna engine is **steady, reliable fetching**.
The engine must maintain a continuous stream of race data spanning **3 or more continents** (typically North America, Europe, Australia/Asia, and South Africa) to ensure that predictions never end.

## Fetching Categories

1.  **Entries (Mission-Critical)**: The discovery of upcoming races is the primary goal. It creates the predictions that power the app.
2.  **Results (Validation)**: The harvesting of results validates our strategies and provides the "Proof" needed for analysis.

## Guiding Principles for Agents

1.  **Fetcher Integrity**: Never allow fetchers to remain broken. If a site changes its structure, fixing the corresponding adapter is the top priority.
2.  **Global Coverage**: Prioritize adapters that contribute to 24/7 global coverage, including critical contributions from **South Africa**.
3.  **Ultimate Targets**: The extreme ultimate targets for integration and data quality are **TwinSpires**, **TVG**, and **FanDuel Racing**, as these are the primary platforms where user betting occurs using their specific odds.
4.  **Odds Hygiene**: Placeholder odds (like the default 7/4 or 2.75) must be detected and filtered out during discovery to maintain high-signal predictions.
5.  **Database Cleanliness**: Garbage records (generic runner names, placeholder odds) should be purged from the database regularly.
6.  **Monolithic Architecture**: Keep `fortuna.py` as a monolithic engine for high reliability and ease of deployment.
7.  **EXTRA CREDIT / impossible challenge**: Fetch data from `https://www.racingandsports.com.au/form-guide` to expand coverage for the Australian region.

---
> Source: [masonj0/favplc](https://github.com/masonj0/favplc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
