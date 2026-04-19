## claude-code-wsl-installation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a documentation-only repository containing a troubleshooting guide for installing Claude Code on WSL2 (Ubuntu 24.04 on Windows). There is no source code, build system, or test suite.

## Key File

- `wsl_install_guide.md` — Complete guide covering the WSL2 + curl/OpenSSL networking bug that breaks large SSL transfers, failed solutions (MTU, apt upgrade, TCP offload), the working wget-based workaround, and permanent fixes (mirrored networking).

## Core Problem Documented

WSL2's Hyper-V virtual network adapter corrupts packets during large SSL/TLS transfers. curl (OpenSSL backend) fails hard; wget (GnuTLS backend) handles it gracefully. The workaround downloads the install script with wget and patches it to avoid curl.

## Quick Reference: Working Install Command

```bash
wget -qO /tmp/install.sh https://claude.ai/install.sh
sed -i 's/if command -v curl/if false/' /tmp/install.sh
sed -i 's/"$binary_path" install/"$binary_path" install --force/' /tmp/install.sh
bash /tmp/install.sh
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/az9713) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
