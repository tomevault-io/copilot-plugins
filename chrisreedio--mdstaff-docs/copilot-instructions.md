## mdstaff-docs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an ASM MD-Staff documentation repository that contains Python tools for converting HTML documentation into organized Markdown files. The main purpose is to process ASM (MD-Staff) HTML documentation and create structured documentation for Query Endpoints and Triggered Webhooks.

## Core Architecture

### HTML to Markdown Converter (`html_to_markdown_converter.py`)
- **Purpose**: Converts ASM HTML documentation into organized markdown structure
- **Key Dependencies**: BeautifulSoup4, html2text, pathlib
- **Output Structure**: Creates organized directories under `asm_docs/` with categorized markdown files

### Documentation Structure
The converter creates two main documentation types:

#### Query Documentation (`asm_docs/query/`)
- 01_overview: Basic Query Endpoint concepts and security
- 02_options: Fields, filters, sorting, counts, pagination  
- 03_provider_records: Core provider data types and examples
- 04_enrollment: Provider enrollments, entities, networks
- 05_additional: Supplementary provider information
- 06_security: Security models and permissions
- 07_other_records: Lookups, reference sources, history

#### Triggered Webhooks Documentation (`asm_docs/triggered_webhooks/`)
- 01_setup: API keys, webhook endpoints, triggered messages
- 02_operations: Testing, logging, troubleshooting
- 03_examples: Sample configurations and payloads

## Development Commands

### Running the Converter
```bash
# Convert specific HTML file
python3 html_to_markdown_converter.py "Query Endpoint – ASM.html"

# Convert triggered webhooks HTML
python3 html_to_markdown_converter.py "Triggered Webhooks – ASM.html"

# The script will auto-detect document type and create appropriate structure
```

### Required Dependencies
Install required Python packages:
```bash
pip install beautifulsoup4 html2text
```

## Key Features

### Image Processing
- Automatically copies images from HTML asset directories to `images/` folders
- Updates image paths in markdown to use relative paths
- Handles various image formats (PNG, JPG, SVG, etc.)

### JSON Detection and Formatting
- Automatically detects JSON code blocks in HTML content
- Wraps them with proper markdown code fence syntax
- Validates JSON before wrapping to avoid false positives

### Content Organization
- Creates README.md index files for each section
- Tracks created files to avoid generating empty placeholders
- Maintains hierarchical structure based on HTML table of contents

## File Structure Conventions

- **HTML Files**: Place source HTML files in root directory
- **Asset Directories**: HTML asset folders (e.g., "Query Endpoint – ASM_files/") contain images and resources
- **Output**: All generated markdown goes under `asm_docs/` with appropriate subdirectories
- **Images**: Consolidated into `images/` folders within each documentation section

## Content Processing

### Sub-Records Support
The Query documentation includes examples of accessing sub-record data:
- Lookup sub-records: `LicenseTypeID.Description` 
- Reference source sub-records: `ReferenceSourceID.Name`
- Automatic ID field handling with/without suffix

### Security and Permissions
All documentation respects MD-Staff permission models - API access follows security settings configured in the application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrisreedio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chrisreedio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
