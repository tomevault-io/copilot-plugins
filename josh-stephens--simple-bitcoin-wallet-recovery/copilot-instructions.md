## simple-bitcoin-wallet-recovery

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Simple Bitcoin Wallet Recovery is a tool for recovering Bitcoin from old wallet.dat files. It detects Bitcoin wallets, extracts private keys using a modified PyWallet, checks balances across multiple blockchain APIs, and exports keys in formats compatible with modern wallets.

## Plugin Commands

This repository is a Claude Code plugin. When installed, these slash commands are available:

| Command | Description |
|---------|-------------|
| `/recover-wallet <path>` | Full recovery wizard on file or directory |
| `/scan-wallets <directory>` | Scan directory for wallet files |
| `/detect-wallet <file>` | Check if a file is a Bitcoin wallet |
| `/check-balance <address>` | Check Bitcoin address balance |

## CLI Commands

```bash
# Install dependencies
pip install -r requirements.txt

# System dependencies (Ubuntu/Debian - Berkeley DB required)
sudo apt-get install python3-bsddb3 libdb-dev

# Run recovery wizard (main entry point)
python3 recovery_wizard.py --scandir /path/to/scan     # Scan directory
python3 recovery_wizard.py wallet.dat                   # Single file
python3 recovery_wizard.py --scandir /path --dry-run   # Preview scan

# Direct pywallet dump (legacy)
python3 pywallet.py --wallet=wallet.dat --dumpwallet

# Test individual modules
python3 lib/wallet_detector.py /path/to/file.dat
python3 lib/balance_checker.py
```

## Architecture

### Module Flow
```
recovery_wizard.py (orchestrator)
    ├── lib/wallet_detector.py  → Identifies Bitcoin wallets via Berkeley DB headers + pattern matching
    ├── pywallet.py             → Extracts keys from wallet.dat (subprocess call with --dumpwallet)
    ├── lib/balance_checker.py  → Multi-API balance verification with caching
    └── lib/secure_exporter.py  → Exports keys in multiple formats
```

### Wallet Detection (`lib/wallet_detector.py`)
- Analyzes files for Berkeley DB magic bytes (versions 4.x, 5.x, 6.x)
- Pattern matches for wallet-specific strings: `defaultkey`, `bestblock`, `pool`, `key`, `wkey`, `ckey`, `mkey`
- Returns confidence score 0-100%; threshold for "is wallet" is 40%

### Balance Checking (`lib/balance_checker.py`)
- APIs: blockchain.info (batch 100), blockstream, blockcypher, mempool.space
- Rate limits: 0.1-1.0 seconds between requests per API
- Caching: 1-hour TTL in `.balance_cache/` directory
- Uses median consensus when multiple APIs return different values

### Key Export (`lib/secure_exporter.py`)
- Formats: `electrum`, `bitcoin_core`, `json`, `csv`, `qr`, `encrypted`
- Sets file permissions to 600 on Unix systems
- Encrypted export uses AES-256 via PBKDF2HMAC

### PyWallet (`pywallet.py`)
- Modified fork of jackjack's pywallet v2.2
- Python 3 compatibility: uses `BytesEncoder` for JSON serialization, `in` operator instead of `has_key()`
- Handles byte/string conversions via `Bdict` class

## Output Structure

The wizard creates a timestamped directory:
```
bitcoin_recovery_YYYYMMDD_HHMMSS/
├── scratch/                    # Temporary files (deletable)
├── output/
│   ├── summary_report.txt
│   ├── funded_wallets/         # Wallets with balance
│   │   └── wallet_XXX/
│   │       ├── private_keys.txt
│   │       ├── wallet_info.json
│   │       └── TRANSFER_GUIDE.txt
│   ├── empty_wallets/
│   └── logs/scan_log.json
```

## Development Notes

- Berkeley DB library (`bsddb3`) is the critical dependency - install system packages first
- Files under 10KB are skipped during directory scans
- Balance checker limits to first 1000 addresses per wallet
- Wallet processing runs pywallet via subprocess to isolate potential crashes

---
> Source: [josh-stephens/simple-bitcoin-wallet-recovery](https://github.com/josh-stephens/simple-bitcoin-wallet-recovery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
