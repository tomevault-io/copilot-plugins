## scanner-minute

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**scanner_minute** is a financial data scanning application that uses the [Polygon.io](https://polygon.io) API for market data. 

The project is built of several components.

### Component 1: Polygon.io or massive.com API
- We have `src/polygon_utils.py` that contains all the functions to download data from the internet. 
- The polygon_utils.py will have two main 

- It will have the following commands:
    1. Function that connected to the API and returns a client.
    2. A function to download several 


### Component 1: Database
- We use [Polygon.io](https://polygon.io) api to download stocks data. 
- We use the database DuckDB to keep all the data we already downloded from the internet. 
- When ever we need a data, if the data is not in the database, we download it from polygon.io and push it into the database. 
- The database should have the following fields:
```
CREATE TABLE ohlcv (
    ticker VARCHAR,
    ts TIMESTAMP,
    open DOUBLE,
    high DOUBLE,
    low DOUBLE,
    close DOUBLE,
    volume DOUBLE
);
```

## Repository Layout

- `src/` — application source code (to be built out)
- `api_keys/` — local API key storage (not tracked in git)
- `docs/` — personal development notes

## Stack

## Architecture

## Conventions

## API Keys

- Polygon API key is stored locally in `api_keys/polygon_api_key.txt`
- This directory is **not tracked by git** and must not be committed
- A `.gitignore` should be added to exclude `api_keys/` and other secrets

---
> Source: [dotd/scanner_minute](https://github.com/dotd/scanner_minute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
