## harmonyos-docs-fetcher

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a HarmonyOS developer documentation fetcher that scrapes the official Huawei HarmonyOS documentation site (https://developer.huawei.com/consumer/cn/doc/) and converts HTML content to Markdown format.

**Documentation Status**: All documentation has been fetched (9,695 Markdown files).
- **Guides**: 4,062 files in `harmonyos-docs-full/guides/`
- **References**: 5,633 files in `harmonyos-docs-full/references/`

## Directory Structure

```
harmony-developer-docs-fetcher/
├── harmonyos-docs-full/        # Fetched documentation (read-only)
│   ├── guides/                 # Development guides (按目录层级结构)
│   ├── references/             # API reference documentation (按目录层级结构)
│   ├── *_mapping.json          # URL-to-path mapping tables
│   └── *_progress.json         # Crawl progress tracking
├── parser/                     # HTML to Markdown conversion module
│   └── html_to_markdown.py     # BeautifulSoup-based HTML→MD converter
├── merge_docs.py               # Tool to merge docs into single large MD files
└── README.md                   # Project documentation
```

## Common Commands

### Merge Documentation into Single Files

Generate two large Markdown files (one for guides, one for references) with hierarchical heading structure based on directory paths:

```bash
cd /d/code/harmony-developer-docs-fetcher
python merge_docs.py
```

Output files:
- `harmonyos-docs-full/guides_all.md` (~28 MB)
- `harmonyos-docs-full/references_all.md` (~75 MB)

### Count Files by Category

```bash
# Count guides
find harmonyos-docs-full/guides -name "*.md" | wc -l

# Count references
find harmonyos-docs-full/references -name "*.md" | wc -l
```

### Search Documentation

```bash
# Search for specific API/feature in guides
grep -r "剪贴板" harmonyos-docs-full/guides/

# Search in references
grep -r "pasteboard" harmonyos-docs-full/references/
```

## Architecture

### Parser Module (`parser/`)

The `HTMLToMarkdown` class in `parser/html_to_markdown.py` converts HTML content to Markdown using BeautifulSoup:

- **Input**: Raw HTML from documentation pages
- **Process**:
  1. Cleans navigation elements (`<nav>`, `<header>`, breadcrumb classes)
  2. Converts HTML tags to MD equivalents (headings, code blocks, tables, lists)
  3. Preserves code language markers from `class="language-*"` attributes
  4. Strips boilerplate navigation lines
- **Output**: Clean Markdown text

Usage:
```python
from parser import HTMLToMarkdown

converter = HTMLToMarkdown()
markdown = converter.convert(html_content, title="Page Title")
```

### Merge Tool (`merge_docs.py`)

Recursively walks the `guides/` and `references/` directories and creates single-file outputs with:
- Document metadata header (generation time, file count)
- Hierarchical headings based on directory depth
- Separator (`---`) between documents

Heading levels: starts at `##` for top-level directories, increasing by depth.

## Documentation Organization

### Guides (`guides/`)
20 top-level categories including:
- 基础入门 (Getting Started)
- 开发基础 (Development Basics)
- 应用框架 (Application Framework)
- 系统 (System)
- 应用服务 (Application Services)
- 工具 (Tools)
- 测试 (Testing)

### References (`references/`)
10 top-level categories including:
- 应用开发 (Application Development)
- 应用框架 (Application Framework)
- 系统 (System)
- 媒体 (Media)
- 图形 (Graphics)
- 应用服务 (Application Services)

## Quick API Reference

### Screen Time Guard Kit (屏幕时间守护)

**Permission**: `ohos.permission.MANAGE_SCREEN_TIME_GUARD`

**Module**:
```typescript
import { guardService } from '@kit.ScreenTimeGuardKit';
```

**Key APIs**:
- `requestUserAuth()` - Request user authorization
- `addGuardStrategy()` - Add time management strategy
- `setAppsRestriction()` - Set app access restrictions

### Clipboard (剪贴板)

**Permission**: `ohos.permission.READ_PASTEBOARD` (required for reading)

**Module**:
```typescript
import { pasteboard } from '@kit.BasicServicesKit';
import { PasteButton } from '@kit.ArkUI';
```

**Key APIs**:
- `setData()` - Write to clipboard (no permission needed)
- `getData()` - Read from clipboard (permission required)
- `hasData()` - Check if data exists (no permission needed)

## Knowledge Base Configuration

When creating knowledge bases from the merged MD files:

**Guides KB**: `HarmonyOS 开发指南完整版`
- Focus: Development workflows, best practices, feature implementation, troubleshooting

**References KB**: `HarmonyOS API参考手册完整版`
- Focus: API specifications, parameters, return values, error codes, permissions, device support

---
> Source: [yonglunyao/harmonyos-docs-fetcher](https://github.com/yonglunyao/harmonyos-docs-fetcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
