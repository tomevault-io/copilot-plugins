## claude-code-r

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an R project repository currently in its initial setup phase.

## R Environment

This project uses R and follows standard R project conventions:
- R-specific files (.Rproj.user, .Rhistory, .RData, .Ruserdata) are gitignored
- Standard R project structure expected

## Development Workflow

When R code is added to this repository, typical commands will include:

**Running R scripts:**
```bash
Rscript path/to/script.R
```

**Installing dependencies (if DESCRIPTION file exists):**
```bash
R -e "devtools::install_deps()"
```

**Running tests (if using testthat):**
```bash
R -e "devtools::test()"
```

**Building package (if this becomes an R package):**
```bash
R CMD build .
R CMD check packagename_version.tar.gz
```

**Interactive R session:**
```bash
R
```

## Notes

- The repository is currently minimal and will be populated with R code, packages, or analysis scripts
- Development conventions should be established as the project grows

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/buddha2490) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
