## pip-install

> Any `pip install` command MUST include one of these options:

# Pip Install Requirements

## ALWAYS use force or direct PyPI URL

Any `pip install` command MUST include one of these options:

```bash
# Option 1: Force reinstall
pip install --force-reinstall <package>

# Option 2: Direct PyPI URL
pip install --index-url https://pypi.org/simple/ <package>

# Option 3: Both (most reliable)
pip install --force-reinstall --index-url https://pypi.org/simple/ <package>

# Example with requirements
pip install --force-reinstall --index-url https://pypi.org/simple/ -r requirements.txt
```

## Why?

Network/proxy restrictions may interfere with default pip behavior. Using explicit PyPI URL ensures direct connection.

## Common Commands

```bash
# Install package
pip install --force-reinstall --index-url https://pypi.org/simple/ requests

# Upgrade package
pip install --upgrade --force-reinstall --index-url https://pypi.org/simple/ contentctl

# Install from requirements
pip install --force-reinstall --index-url https://pypi.org/simple/ -r requirements.txt
```

---
> Source: [MHaggis/Security-Detections-MCP](https://github.com/MHaggis/Security-Detections-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
