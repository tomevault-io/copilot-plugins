## edirouterpoc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a proof-of-concept EDI (Electronic Data Interchange) routing system for X12 documents. The system automates file transfer between an ERP system and trading partners via FTP/SFTP.

## Commands

### Setup
```bash
pip install -r requirements.txt
```

### Run Scripts
```bash
# Process outbound files (ERP → Partners)
python outbound_processor.py

# Process inbound files (Partners → ERP)
python inbound_processor.py

# With custom config file paths
python outbound_processor.py -m /path/to/master_config.json -p /path/to/partners.json
python inbound_processor.py -m /path/to/master_config.json -p /path/to/partners.json
```

## Architecture

### Two Independent Scripts
The system uses two standalone Python scripts that operate independently:

1. **outbound_processor.py** - Handles ERP → Partner flow
2. **inbound_processor.py** - Handles Partner → ERP flow

Both scripts are designed to be run via OS schedulers (Task Scheduler on Windows, cron on Linux).

### Configuration System
The application uses a two-file JSON configuration approach:

- **master_config.json** - Local system paths (pickup folders, archive folders, log folders)
- **partners.json** - Trading partner details (FTP/SFTP credentials, paths, protocol type)

Both scripts read these configurations at runtime. There is no shared state between script executions.

Config file paths can be specified via command line arguments (`--master-config`/`-m` and `--partners-config`/`-p`), defaulting to files in the current directory.

### X12 ISA Segment Parsing
The outbound processor determines partner routing by parsing the ISA (Interchange Control Header) segment:

- X12 files start with an ISA segment (line 1)
- Fields are delimited by `*`
- **ISA08** (index 8, 0-based - ISA itself is at index 0) contains the receiver ID = partner_id
- The partner_id is matched against partners.json to determine FTP destination

### FTP Protocol Handling
Both processors support dual protocols:

- **FTP** - Uses Python's `ftplib.FTP`
- **SFTP** - Uses `paramiko` for SSH-based transfers

Protocol selection is per-partner via the `protocol` field in partners.json.

### File Tracking Strategy
**Outbound**: Files are archived after successful upload using a template system:
- Archive path and filename can be customized via templates in master_config.json
- Templates support placeholders: `{filename}`, `{extension}`, `{partner_id}`, `{partner_name}`, `{timestamp}`, `{date}`, `{time}`, `{year}`, `{month}`, `{day}`, `{hour}`, `{minute}`, `{second}`
- Per-partner overrides available in partners.json (optional)
- Example: `{partner_id}/{year}/{month}` creates subdirectories by partner and date

**Inbound**: The system uses filename prefixing on remote servers:
- Download only files that DON'T start with "X"
- After successful download, rename remote file to "X{original_filename}"
- This marks files as processed without deletion

### Logging Architecture
Each processor creates separate daily log files:
- `outbound_YYYY-MM-DD.log`
- `inbound_YYYY-MM-DD.log`

Logs include file-level tracking, partner information, success/failure status, and detailed error messages. Both file and console output are enabled.

**FTP Debugging**: The `ftp_debug_level` setting (0-2) in master_config.json controls FTP protocol debugging. When set to 1 or 2, all FTP commands and server responses are logged, useful for troubleshooting connection issues.

## Important Implementation Details

- **Error Handling**: Both scripts continue processing remaining files if one file fails. Individual file failures are logged but don't stop the batch.
- **No Database**: The system is stateless. File tracking relies on filesystem operations (archives, filename prefixes).
- **Security**: This is a proof-of-concept with plaintext credentials in JSON. Production use requires credential encryption or secure storage.
- **Partner Enablement**: Partners can be disabled via `"enabled": false` in partners.json without removing their configuration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/harrbca) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
