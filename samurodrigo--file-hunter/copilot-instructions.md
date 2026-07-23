## file-hunter

> This project is a CLI tool specialized in locating and downloading files from **Archive.org**. The system automatically manages authentication and bypasses download restrictions (403 Forbidden) using persistent session cookies.

# Project: File Hunter Downloader (Archive.org Specialist)

## Overview

This project is a CLI tool specialized in locating and downloading files from **Archive.org**. The system automatically manages authentication and bypasses download restrictions (403 Forbidden) using persistent session cookies.

The user will:

1. Provide an Archive.org URL or a local HTML file (exported from the site).
2. Authenticate via terminal or browser (once).
3. Filter by extensions and filenames.
4. Select files via an interactive menu.
5. Choose the destination folder.
6. Monitor simultaneous downloads with progress bars.

---

# Project Goals

- Rich and responsive terminal interface (Rich/Questionary).
- Asynchronous and concurrent downloads (httpx + asyncio).
- Ease of adding new extensions in the future.
- Robust error handling and clear logs.

---

# Suggested Stack

## Language
- Python 3.10+

## Core Libraries
- **Typer**: Command-line interface.
- **httpx**: Asynchronous HTTP requests.
- **BeautifulSoup4**: HTML parsing.
- **Questionary**: Interactive selection (checkbox style in the terminal).
- **Rich**: Tables, colors, and progress bars.
- **Aiofiles**: Asynchronous file writing.
- **Tomli**: TOML configuration parsing.

---

# Main Features

## 1. Link Search
The system shall:

- Make an asynchronous request for the page.
- Extract all links (`href`).
- Normalize relative URLs.
- Filter by the provided extensions and search terms.

## 2. Interactive Selection
The CLI shall allow:

- Checking/Unchecking files individually (Space).
- Selecting all (A).
- Inverting selection (I).
- Navigation with a "Back to search" option.

## 3. Managed Download
The system shall:

- Support simultaneous downloads (configurable).
- Display individual progress bars and a general summary.
- Sanitize filenames to avoid OS errors.
- Implement a retry system with exponential backoff.

---

# Technical Requirements

## Performance
- Use of `asyncio` to avoid blocking during network or disk I/O.

## Security
- Validate URLs (Archive.org only).
- Avoid path traversal in filename sanitization.

## Logs
Register connection errors and failed downloads in a log file (`logs/downloader.log`).

---

# Recommended Structure

```txt
/src
  /core
    scanner.py
    downloader.py
    auth.py
  /ui
    menus.py
  /utils
    logger.py
    i18n_helper.py
  /i18n
    en.json
    pt.json
    ...
  main.py
```

---

# Conventions

## Code
- Mandatory type hints.
- PEP 8 (code style).
- Pylint/Flake8 for linting.

## Commits
Use Conventional Commits (feat:, fix:, etc.).

---

# Initial Priority (MVP) - COMPLETED

1. Scan Command (URL + Extensions).
2. Interactive selection menu.
3. Asynchronous download with progress bar.
4. Saving to local folder.
5. Interactive loop and search refinement.
6. Multi-language support and configuration file.

---
> Source: [samurodrigo/file-hunter](https://github.com/samurodrigo/file-hunter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
