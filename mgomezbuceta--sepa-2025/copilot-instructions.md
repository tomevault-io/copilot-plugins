## sepa-2025

> Generates ISO 20022 pain.008.001.09 SEPA direct debit XML files with:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SEPA is a Python GUI application for processing CSV files (semicolon-separated) containing payment data and generating SEPA XML files for direct debit collection.

## Architecture

The application follows a data flow pattern:
1. **Input**: CSV files with payment data (placed in `sin_procesar/`)
2. **Processing**: Parse CSV, validate data, transform to SEPA XML format
3. **Output**: Generated XML files (saved to `xml/`), processed CSVs moved to `procesados/`)

### Key Components
- **GUI Layer**: User interface for file selection and processing
- **CSV Processor**: Handles semicolon-separated CSV parsing and validation
- **SEPA XML Generator**: Creates ISO 20022 pain.008.001.09 compliant XML files
- **File Manager**: Handles moving processed files between directories

## Data Format

### Input CSV Structure
The CSV files use semicolon (`;`) separators and contain SEPA direct debit fields including:
- Message identification and timestamps
- Payment information (amounts, dates, references)
- Creditor details
- Debtor details
- Mandate information for direct debits

Example fields: `MsgId;CreDtTm;NbOfTxs;CtrlSum;InitgPty_Nm;PmtInfId;...`

### Output XML Format
Generates ISO 20022 pain.008.001.09 SEPA direct debit XML files with:
- Document root: `urn:iso:std:iso:20022:tech:xsd:pain.008.001.09`
- Customer Direct Debit Initiation messages
- Payment information blocks for each transaction

## File Organization

- `src/`: Python source code
  - `gui/`: GUI components and interface logic
  - `processors/`: CSV parsing and SEPA XML generation
  - `validators/`: Data validation and business rules
  - `utils/`: Common utilities and helpers
- `data/`: Data processing directories
  - `sin_procesar/`: Unprocessed CSV files
  - `procesados/`: Successfully processed CSV files
  - `xml/`: Generated SEPA XML files
- `docs/`: Documentation, examples, and reference files (contains sensitive project information)
- `tests/`: Unit tests and test data
- `config/`: Configuration files and settings

## Development Commands

Since this is a new project without existing build tools, common commands will be:
- `python src/main.py` - Run the GUI application
- `python -m pytest tests/` - Run unit tests (when tests are added)
- `pip install -r requirements.txt` - Install dependencies (when requirements.txt is created)

## Key Requirements

- Process semicolon-separated CSV files
- Generate ISO 20022 compliant SEPA XML
- Move processed files to appropriate directories
- Provide GUI for non-technical users
- Handle specific requirements for SEPA payments

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mgomezbuceta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
