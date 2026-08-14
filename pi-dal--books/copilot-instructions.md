## books

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a WeRead book management system that processes and organizes book notes and reading data. The project extracts metadata from markdown files containing book content and generates formatted outputs.

## Project Structure

- `Books/` - Raw book markdown files with metadata (title, author, reading time) and content
- `Books-Format/` - Processed book files with metadata removed (content after "---" separator)
- `Books.md` - Generated table of all books with title, author, and reading time
- `extract_book_info.py` - Extracts book metadata and generates Books.md table
- `format_books.py` - Processes raw books to create clean formatted versions

## Development Commands

This project uses PDM for Python dependency management:

```bash
# Install dependencies
pdm install

# Run book info extraction
python extract_book_info.py

# Format all books (remove metadata headers)
python format_books.py
```

## Book File Format

Raw book files in `Books/` follow this structure:
```
书名标题
作者姓名
笔记数量
阅读周期:
* 阅读时长：X 小时
* 开始时间：YYYY/MM/DD HH:MM:SS
* 结束时间：YYYY/MM/DD HH:MM:SS
---
[Book content starts here]
```

The `format_books.py` script removes everything before the "---" separator to create clean content files in `Books-Format/`.

## Dependencies

### Python (Legacy)
- `requests>=2.32.4` - HTTP library
- `web3>=7.12.0` - Web3 integration
- Python 3.13 (exact version required)

### TypeScript/Node.js (xLog Sync)
- `@crossbell/indexer` - Crossbell blockchain indexer
- `crossbell` - Crossbell SDK for xLog integration
- `typescript` - TypeScript compiler
- `fs-extra` - Enhanced file system operations
- `gray-matter` - YAML front matter parser
- `marked` - Markdown parser

## Data Processing

### Python Scripts
The `extract_book_info.py` script:
1. Reads all .md files from `Books/` directory
2. Extracts title (line 1), author (line 2), and reading time from metadata
3. Generates a markdown table in `Books.md` with all book information

### xLog Sync (TypeScript)
The sync tool (`src/`) provides:
1. Book parsing and content conversion
2. xLog platform integration via Crossbell protocol
3. Batch and selective synchronization
4. Preview mode for testing

## xLog Sync Commands

```bash
# Setup
npm install
npm run sync:dev init

# Usage
npm run sync:dev list          # List all books
npm run sync:dev test          # Test xLog connection
npm run sync:dev preview       # Preview sync (dry run)
npm run sync:dev sync          # Sync all books
npm run sync:dev sync "book.md" # Sync specific book
```

Configuration requires `.env` file with:
- `XLOG_PRIVATE_KEY` - Wallet private key
- `XLOG_ADDRESS` - Wallet address
- `XLOG_CHARACTER_ID` - xLog character ID
- `XLOG_SITE_DOMAIN` - xLog site domain

---
> Source: [pi-dal/books](https://github.com/pi-dal/books) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
