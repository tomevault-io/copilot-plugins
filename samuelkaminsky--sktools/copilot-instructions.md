## sktools

> **SKTools** is an R package developed by Samuel Kaminsky designed to facilitate data import, cleaning, and analysis. It provides a collection of utility functions that streamline common tasks when working with data frames in R.

# Project Overview

**SKTools** is an R package developed by Samuel Kaminsky designed to facilitate data import, cleaning, and analysis. It provides a collection of utility functions that streamline common tasks when working with data frames in R.

## Key Features
-   **Data Import/Export:** Functions like `read_excel_all` and `read.xlsx.all` for handling multiple sheets/files.
-   **Descriptive Statistics:** Tools like `descriptives` and `frequencies` for quick data summarization.
-   **Data Manipulation:** Helpers like `append_2_prepend` and `distinct_2col`.
-   **Integration:** Designed to work well with the `tidyverse` ecosystem.

## Technical Architecture
-   **Language:** R
-   **Package Structure:** Standard R package structure (`R/` for source, `man/` for docs, `tests/` for unit tests).
-   **Dependencies:** Heavy reliance on the tidyverse (`dplyr`, `purrr`, `tidyr`, `readr`), plus `openxlsx`, `psych`, and others listed in `DESCRIPTION`.

# Building and Running

## Prerequisites
-   **R:** Version >= 4.1.0 (Required for native pipe `|>`)
-   **devtools/remotes:** For package development and installation.

## Installation
To install the package from the source or GitHub:

```r
# From GitHub
remotes::install_github("samuelkaminsky/SKTools")

# Locally (from project root)
devtools::install()
```

## Testing
The project uses the `testthat` framework for unit testing.

To run tests:
```r
devtools::test()
```

## Documentation
Documentation is generated using `roxygen2`. To update documentation (man pages):
```r
devtools::document()
```

# Development Conventions

## Coding Style & Modernization
-   **Native Pipe:** Use the native R pipe `|>` instead of `%>%` wherever possible.
-   **Modern purrr:** Favor `purrr::map() |> purrr::list_rbind()` over deprecated `map_df()` or `map_dfr()`.
-   **Tidy Evaluation:** Use `rlang::new_formula()` or `rlang::inject()` for safe dynamic model construction instead of `paste0()` and `as.formula()`.
-   **Input Validation:** Use `rlang::abort()` for consistent and informative error signaling.
-   **Internal Helpers:** Redundant logic (like percentile-based grouping) should be abstracted into internal helpers (e.g., `R/utils_cutpoints.R`).
-   **Explicit Namespacing:** Functions from other packages must be called with their namespace (e.g., `dplyr::mutate`).
-   **Pronouns:** Always use the `.data` pronoun inside `dplyr` verbs.

## Linting and Formatting
-   **Strict Compliance:** All code MUST pass `styler::style_pkg()` and `lintr::lint_package()` before being considered for a merge or push.
-   **Line Length:** Adhere to a strict 80-character line limit.

## Testing Standards
-   **Robustness:** Avoid brittle tests that rely on exact row indices (e.g., `slice(37)`). Instead, use `dplyr::filter()` to target specific IV/DV/Cutoff combinations to ensure tests are resilient to row-ordering changes.

## Version Control Workflow
-   **Jujutsu (jj):** The project uses Jujutsu for local version control. Ensure `.jj` is always in `.Rbuildignore`.
-   **Push Restriction:** **CRITICAL:** Never push changes to GitHub without explicit user confirmation.
-   **Pre-push Verification:** Documentation must be updated (`devtools::document()`), and a full `devtools::check()` must pass with 0 errors and 0 warnings before pushing.
-   **Branching:** Merge features into `main` only after verification.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samuelkaminsky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/samuelkaminsky)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
