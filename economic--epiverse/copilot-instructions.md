## epiverse

> An R metapackage maintained by the Economic Policy Institute (EPI). Similar to `tidyverse` or `fastverse`, it provides a single entry point to install and load a collection of EPI R packages.

# epiverse

An R metapackage maintained by the Economic Policy Institute (EPI). Similar to `tidyverse` or `fastverse`, it provides a single entry point to install and load a collection of EPI R packages.

## Instructions

- Always keep this CLAUDE.md file up to date as the project evolves. When making changes that affect the project description, design decisions, package list, or distribution approach, update this file accordingly.

## Included packages

- [epidatatools](https://economic.github.io/epidatatools/)
- [epiextractr](https://economic.github.io/epiextractr/)
- [epitargets](https://economic.github.io/epitargets/)
- [prefab](https://economic.github.io/prefab/)
- [realtalk](https://economic.github.io/realtalk/)
- [swadlr](https://economic.github.io/swadlr/)

## Distribution

- Distributed via r-universe and GitHub (not CRAN)
- All sub-packages are also on r-universe and GitHub
- Package sources hosted under https://github.com/economic/ (the EPI GitHub organization)

## Usage

```r
# From r-universe
install.packages("epiverse", repos = "https://economic.r-universe.dev")

# Or from GitHub
remotes::install_github("economic/epiverse")

library(epiverse)
```

## Design

Follows the tidyverse metapackage pattern:
- All sub-packages listed in `Imports` (not `Depends`) so the metapackage controls attachment
- All 6 packages are core — all attached on `library(epiverse)`
- `.onAttach` hook attaches all packages and displays a startup message with versions
- Conflict detection and reporting
- Utility functions: `epiverse_packages()`, `epiverse_update()`, `epiverse_sitrep()`
- `options(epiverse.quiet = TRUE)` to suppress startup messages
- Dummy function referencing one symbol per import to avoid R CMD check notes
- MIT license
- No known conflicts between included packages
- Design document: `plans/2026-03-14-epiverse-metapackage-design.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Economic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
