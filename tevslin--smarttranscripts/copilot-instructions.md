## smarttranscripts

> This document outlines key development rules to ensure consistency and maintainability within this repository.

# Development Rules for smarttranscripts repository

This document outlines key development rules to ensure consistency and maintainability within this repository.

## Dependency Management
- **Whenever code is added or modified that introduces new external Python libraries, ensure that `requirements.txt` is updated accordingly.** This keeps our dependency list accurate and reproducible.

## Environment Variables
- **Whenever an entry is added to `.env`, a corresponding placeholder entry must be added to `sample.env`.** This ensures that new environment variables are documented and easily discoverable for other developers. For example, if you add `API_KEY="your_key"` to `.env`, you should add `API_KEY=<your_api_key>` to `sample.env`.


## Shell Commands
- Commands for local execution should use PowerShell (PS) syntax, not Bash.

---
> Source: [tevslin/smarttranscripts](https://github.com/tevslin/smarttranscripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
