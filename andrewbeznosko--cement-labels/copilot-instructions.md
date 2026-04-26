## cement-labels

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a web-based cement label generator for creating printable PDF labels for cement packaging. The project generates 8 labels per A4 page for "ЦЕМЕНТ 500" (Portland cement with slag CEM II/B-S 42.5K) produced by "Подільський цемент".

## Project Structure

- `index.html` - Main selection interface for choosing label templates
- `labels-standard.html` - Standard label template with larger margins (5mm inner padding)
- `labels-with-date-field.html` - Label template with packaging date field
- `CLAUDE.md` - This documentation file

## Development Commands

This is a static HTML project with no build process required.

### To view template selection:
```bash
# Open main interface in any web browser
open index.html
# or
python3 -m http.server 8000  # then visit http://localhost:8000
```

### To generate PDF:
- Open index.html and select desired template
- Click "📄 Скачати PDF" button on the chosen template page
- Use browser's "Print to PDF" option

## Label Specifications

- **Format**: 8 labels per A4 page (2 columns x 4 rows)
- **Size**: Each label sized for 3kg cement packaging
- **Content**: Ukrainian text with cement specifications
- **Features**: 
  - "Keep dry" icon (umbrella with raindrops)
  - Product description and usage instructions
  - Manufacturer information
  - Weight and packaging date fields

## Key Features

- Template selection interface with two variants
- Print-optimized CSS with @page rules for A4 format
- Responsive grid layout for labels
- SVG icon for moisture protection warning
- Browser print integration for PDF generation
- Fixed download button (hidden during printing)
- Cut lines between labels for easy separation

## Notes

- All text is in Ukrainian
- Labels are designed for 3kg cement packaging
- Print margins optimized for standard A4 printers
- Uses Arial font for best print compatibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndrewBeznosko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
