## electroanalyzer

> **Python Version:** Python 3.12 is required for this project.

# ElectroAnalyzer Workspace Rules

## Python Environment Configuration

**Python Version:** Python 3.12 is required for this project.

**Virtual Environment Setup:**
- **Windows:** Use `venv` (not `.venv`)
  ```bash
  python -m venv venv
  venv\Scripts\activate
  ```

- **Linux/macOS:** Use `.venv` (with dot prefix)
  ```bash
  python3.12 -m venv .venv
  source .venv/bin/activate
  ```

**Rationale:**
- Windows: `venv` is the standard convention and avoids potential issues with hidden directories
- Linux: `.venv` follows the convention of keeping virtual environments hidden and is consistent with existing scripts

**When creating or referencing virtual environments:**
- Always specify Python 3.12 explicitly: `python3.12 -m venv` or `py -3.12 -m venv`
- Use platform-appropriate venv directory name (`venv` for Windows, `.venv` for Linux)
- Update documentation and scripts to reflect this convention

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kmoon0001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
