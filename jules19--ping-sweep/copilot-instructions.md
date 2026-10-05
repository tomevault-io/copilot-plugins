## ping-sweep

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a defensive network discovery tool that performs ping sweeps with enhanced MAC address lookup and device classification. The tool is designed for network administrators and security professionals to inventory and monitor authorized networks.

## Running the Tool

Basic usage:
```bash
python pingsweep.py 192.168.1.0/24
```

With verbose output and device classification:
```bash
python pingsweep.py -v 192.168.1.0/24
```

With custom threading and timeout:
```bash
python pingsweep.py --workers 20 --timeout 500 192.168.1.0/24
```

Export results to JSON:
```bash
python pingsweep.py --export -v 10.0.0.0/24
```

Fast Senetas encryptor discovery (requires root and scapy):
```bash
sudo python pingsweep.py --find-encryptors 10.0.0.0/16
```

## Dependencies

The tool requires Python 3.6+ and these libraries:
```bash
pip install requests tqdm
```

For `--find-encryptors` mode (optional, lazy-imported):
```bash
pip install scapy
```

## Architecture

The codebase consists of two main classes in `pingsweep.py`:

### MACVendorLookup Class
- Downloads and caches the IEEE OUI database from `http://standards-oui.ieee.org/oui/oui.txt`
- Stores vendor mappings in `oui_database.json` for offline use
- Falls back to a minimal built-in database if download fails
- Performs MAC-to-vendor lookup and device type classification
- Classification categories: Encryptor, Network Equipment, Mobile Device, Computer, IoT Device, Printer, Gaming Console, Camera, Unknown Device

### EnhancedPingSweep Class
- Multi-threaded ping sweep using `concurrent.futures.ThreadPoolExecutor`
- Platform-specific ping commands (Windows: `ping -n`, Unix: `ping -c`)
- ARP table lookup for MAC address resolution (`arp -a` on Windows/macOS, `arp -n` on Linux)
- MAC address normalization to handle various formats
- Optional hostname resolution via `nslookup`
- Thread-safe statistics collection and result aggregation

### find_encryptors() Function
- Fast ARP-based discovery of Senetas encryptors using Scapy
- Filters ARP responses for OUI prefix `00:d0:1f` (SENETAS_OUI constant)
- Chunks large subnets into /24s to manage memory
- Requires root/admin privileges for raw socket ARP
- Scapy is lazy-imported only when `--find-encryptors` is used

## Key Implementation Details

### MAC Address Handling
The tool normalizes MAC addresses from various formats (colon-separated, dash-separated, single digits) to a standard 12-character hex string, then formats as colon-separated pairs.

### Device Classification
Classification is performed using vendor string pattern matching and hostname analysis. The system looks for keywords in vendor names (e.g., "cisco" → Network Equipment, "apple" → Mobile Device) and hostname patterns (e.g., "router" → Network Equipment).

### Cross-Platform Compatibility
The tool detects the platform using `sys.platform` and adjusts ping and ARP commands accordingly. Windows uses different command syntax and output parsing than Unix-like systems.

### Database Management
The OUI database is automatically downloaded on first run and cached locally. The tool includes a fallback minimal database with common vendors in case the download fails.

## Output Formats

- **Simple mode**: IP, response time, vendor in tabular format
- **Verbose mode**: Detailed device information with emojis and device type classification
- **JSON export**: Structured data with scan metadata and device details

## Security Considerations

This is a defensive security tool for authorized network scanning only. The tool:
- Only gathers network-visible information (IP, MAC, hostname)
- Includes ethical usage warnings in documentation
- Performs no exploitation or intrusion attempts
- Designed for network inventory and monitoring purposes

---
> Source: [jules19/ping_sweep](https://github.com/jules19/ping_sweep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
