## preceptorials-tracker-streamlit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Streamlit dashboard for tracking progress of learners in the MV preceptorial program.

## Development Setup
- Python Project using Pipenv
- Main Dependencies:
  - Streamlit
  - Pandas
  - NumPy
  - Requests
  - PyDeck

## Common Commands
- Start Development Server:
  ```
  pipenv shell
  streamlit run ./src/Preceptorials_Dashboard.py --server.runOnSave true
  ```

## Project Structure
- Main Dashboard: `src/Preceptorials_Dashboard.py`
- Pages: Located in `src/pages/`
- Utility Functions: Located in `src/util/`
  - `directus_connector.py`: Database connection
  - `queries.py`: Database query management
  - `backend.py`: Backend utility functions

## Important Notes
- Uses Streamlit for web interface
- Connects to Directus for data management
- Dashboard focuses on tracking learner progress in preceptorial program
- declare types of variables and functions,

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samisaf)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/samisaf)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
