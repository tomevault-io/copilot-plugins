## chrysalis-ioc-triage

> This project checks a Windows host for **Indicators of Compromise (IoC)** related to the **Chrysalis backdoor** and **Lotus Blossom (Billbug)** campaign described in Rapid7’s write-up:

# Triage – Chrysalis / Lotus Blossom IoC Investigation

## Purpose

This project checks a Windows host for **Indicators of Compromise (IoC)** related to the **Chrysalis backdoor** and **Lotus Blossom (Billbug)** campaign described in Rapid7’s write-up:

- **Blog:** [The Chrysalis Backdoor: A Deep Dive into Lotus Blossom's toolkit](https://www.rapid7.com/blog/post/tr-chrysalis-backdoor-dive-into-lotus-blossoms-toolkit/)
- **Threat:** Chinese APT Lotus Blossom; initial access via abused Notepad++ update (e.g. `update.exe` from 95.179.213.0).

## What Gets Checked

- **Paths** – Known install paths (e.g. `%AppData%\Bluetooth`, `C:\ProgramData\USOShared`).
- **File hashes** – SHA-256 of known malicious files (installers, DLLs, loaders, shellcode) in those paths and optionally under extra directories.
- **Mutex** – `Global\Jdhfv_1.0.1` (Chrysalis single-instance).
- **Registry** – Run keys (HKCU/HKLM) for values referencing BluetoothService, update.exe, or `-i`/`-k` style arguments.
- **Services** – Services named or pointing to BluetoothService/update.exe.

## Project Layout

- **`iocs.json`** – Machine-readable IoCs (hashes, paths, mutexes, registry keys, network).
- **`docs/chrysalis-iocs.md`** – Human-readable IoC reference and MITRE mapping.
- **`scripts/Check-ChrysalisIoC.ps1`** – PowerShell script that runs the checks and writes a JSON report.

## How to Run the Check

From the repo root (or with correct relative path to `iocs.json`):

```powershell
# Default: paths, hashes in known dirs, registry, mutex, services
.\scripts\Check-ChrysalisIoC.ps1

# Also hash files under given paths (slower)
.\scripts\Check-ChrysalisIoC.ps1 -ScanPaths 'C:\Users','C:\ProgramData'

# Skip registry or mutex
.\scripts\Check-ChrysalisIoC.ps1 -NoRegistry -NoMutex
```

Reports are written under the same directory as `iocs.json` as `chrysalis-scan-YYYYMMdd-HHmmss.json`. Exit code 1 if any finding, 0 if none.

## Agent / AI Guidelines

- When adding IoCs, update both `iocs.json` and `docs/chrysalis-iocs.md` and keep them in sync.
- Prefer the script’s existing categories (Path, FileHash, Mutex, Registry, Service) for new findings; add new categories only when they don’t fit.
- Do not modify live system state (e.g. delete files or change registry) from the script; keep it read-only.
- For network IoCs (IPs/domains), use external tools or manual review; the script focuses on host-based IoCs.

---
> Source: [CreamyG31337/chrysalis-ioc-triage](https://github.com/CreamyG31337/chrysalis-ioc-triage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
