## windows-dev-environment

> Windows development environment setup - activate venv before running Python/pip commands


# Windows Dev Environment

Before running any Python, pip, or pytest commands in the terminal, activate the virtual environment first:

```
.\.env_borg\Scripts\Activate.ps1
```

Chain it with the actual command, e.g.:

```
.\.env_borg\Scripts\Activate.ps1; pip install -e .[dev]
.\.env_borg\Scripts\Activate.ps1; pytest tests/ -v
```

---
> Source: [mlapaglia/Borgitory](https://github.com/mlapaglia/Borgitory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
