## instinct

> Security audit for software installed from the internet — npm packages, pip packages, executables, browser extensions, and git repos. Automatically scans for trojans, malware, obfuscation, credential theft, telemetry, and suspicious install hooks. Use when (1) installing any new npm or pip package, (2) downloading executables or installers, (3) cloning git repos with build scripts, (4) installing browser extensions, (5) user asks to check if something is safe, (6) user says "audit", "scan", "check for malware", "is this safe", or "instinct".


# Instinct — Security Gut Check

Audit anything installed from the internet before trusting it.

## Quick Start

### npm package
```bash
python scripts/npm_audit.py <package-name-or-path>
```
Accepts a global npm package name (e.g., `@tobilu/qmd`) or a direct path to the package directory.

### Python (pip) package
```bash
python scripts/pip_audit.py <package-name-or-path>
```
Accepts an installed package name (e.g., `requests`) or a path to the source directory.

### Everything else (executables, extensions, repos)
Use the manual checklist in `references/checklist.md`. Follow each section relevant to the software type.

## Workflow

1. **Identify type**: npm, pip, executable, browser extension, or git repo
2. **Run automated scan** if npm or pip (use scripts above)
3. **Review findings** — HIGH = block, MEDIUM = review, LOW = note
4. **For non-package software**, follow `references/checklist.md` manually:
   - Check digital signatures and hashes for executables
   - Review permissions for browser extensions
   - Inspect build scripts and hooks for git repos
5. **Report verdict** to user: CLEAN ✅, CAUTION ⚠️, or SUSPICIOUS ❌

## Severity Guide

- **HIGH**: Install hooks with code execution, outbound network calls, credential/wallet access, obfuscated code → do NOT trust without manual review
- **MEDIUM**: Telemetry, env var access, binary files, git URL deps → review but probably fine
- **LOW**: Missing repo field, compiled extensions (.pyd/.so) → informational

## Interpreting Results

Not every finding is malicious. Context matters:
- A web framework legitimately makes HTTP calls
- A CLI tool legitimately uses child_process/spawn
- A compiled extension (.pyd) is normal for performance-critical packages
- `process.env.NODE_ENV` is harmless; `process.env.AWS_SECRET` is not

Read the finding detail and check if the behavior makes sense for what the package does. Flag only what's out of place.

## When Scripts Are Not Enough

For deeper analysis beyond automated scanning:
- Manually read the flagged source files
- Check VirusTotal for executable hashes
- Compare published package against GitHub source (npm pack + diff)
- Search for CVEs or known vulnerabilities
- Check package download trends on npm/PyPI for anomalies

---
> Source: [Alphakil/instinct](https://github.com/Alphakil/instinct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
