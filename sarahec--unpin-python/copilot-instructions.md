## unpin-python

> This tool correlates Nixpkgs package expressions with GitHub code search results to identify packages that are pinned to specific dependency versions (e.g., `hatchling==1.27.0`) and have corresponding source repositories on GitHub.

# Functional Specification: Nixpkgs Version Correlation Tool

This tool correlates Nixpkgs package expressions with GitHub code search results to identify packages that are pinned to specific dependency versions (e.g., `hatchling==1.27.0`) and have corresponding source repositories on GitHub.

## Overview

The tool operates in three primary phases:
1.  **Scan**: Analyzes a local Nixpkgs repository to find package definitions using `fetchFromGitHub` and referencing a target package name.
2.  **Search**: Queries the GitHub Code Search API for specific version strings in `pyproject.toml` files across the found repositories.
3.  **Report**: Correlates the scan data with search results to produce a report of Nixpkgs files that likely need updates.

## Architecture & Data Model

The tool is written in Python and uses an SQLite database (`db.sqlite`) for state management and data persistence.

### Database Schema

The database uses a normalized schema with the following tables:

*   **`scans`**: Tracks which packages have been scanned.
    *   `package_name`: Primary key (e.g., `hatchling`).
    *   `last_scan`: Timestamp of the last scan.
*   **`repositories`**: Stores metadata for GitHub repositories found during the Nixpkgs scan.
    *   `package_name`: Foreign key to `scans`.
    *   `nix_path`: Relative path to the Nix expression in Nixpkgs.
    *   `owner`, `repo`: GitHub repository identification.
    *   Primary key: `(package_name, owner, repo)`.
*   **`search_runs`**: Records individual GitHub API search executions.
    *   `id`: Primary key.
    *   `package_name`: Foreign key to `scans`.
    *   `search_string`: The exact string searched on GitHub.
    *   `last_update`: Timestamp of the search.
*   **`search_matches`**: Links search runs to found GitHub repositories.
    *   `search_id`: Foreign key to `search_runs`.
    *   `repo_full_name`: The `owner/repo` string of a match.
    *   Primary key: `(search_id, repo_full_name)`.

## Functional Components

### 1. Specifier Parsing
The tool accepts package specifiers in the format `package[operator][version]` (e.g., `hatchling==1.27.0`).
- If an operator is provided, it generates two search variants: `package==version` and `package == version`.
- If no operator is provided, it defaults to `==` and an empty version for scanning.

### 2. Nixpkgs Scanning (`scan`)
- Uses `ripgrep` (`rg`) to efficiently find files containing the package name.
- Parses selected Nix files for `fetchFromGitHub` blocks.
- Extracts `owner` and `repo` attributes from these blocks.
- Stores results in the `repositories` table, clearing old results for the same package.

### 3. GitHub Code Search (`search`)
- Requires a `GITHUB_SEARCH_TOKEN` in a `.env` file.
- Performs exact-match searches for the version string within `pyproject.toml` files globaly.
- Implements rate-limiting (10s delay between pages, 60s sleep on 403 errors).
- Stores all matching repository names (`owner/repo`) in the `search_matches` table.

### 4. Reporting (`report`)
- Joins the latest search results with the repository data collected during the scan.
- Groups results by the Nixpkgs file path.
- Outputs a tab-separated list: `nix_path \t package1, package2, ...`.

### 5. Management (`reset`)
- Allows clearing data for specific packages or the entire database (`*`).

## Command Line Interface

```bash
python main.py [-n NIXPKGS_PATH] <command> <arguments>
```

### Options:
- `-n, --nixpkgs-path`: Path to your local nixpkgs clone. If not provided, the tool will check the `NIXPKGS` environment variable, falling back to `../nixpkgs` if that is also unset.

### Commands:
- `scan <specifiers>...`: Scans Nixpkgs for the given packages.
- `search <specifiers>...`: Searches GitHub for the given version specifiers.
- `report <specifiers>...`: Generates a report for the specifiers.
- `all <specifiers>...`: Runs scan, search, and report in sequence.
- `reset <packages>...`: Resets data for the named packages (or `*`).

## Dependencies
- `sqlite3`: Data persistence.
- `requests`: GitHub API interaction.
- `python-dotenv`: Environment variable management.
- `tqdm`: Progress visualization.
- `ripgrep` (`rg`): High-performance file searching (system dependency).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sarahec)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sarahec)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
