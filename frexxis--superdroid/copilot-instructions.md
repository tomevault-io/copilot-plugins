## superdroid

> This file provides guidance to Factory Droid CLI when working with this repository.

# AGENTS.md

This file provides guidance to Factory Droid CLI when working with this repository.

## 🎯 Project Overview

**SuperDroid** is a prompt-level framework that adapts SuperClaude's command/droid/mode architecture for Factory Droid CLI.

### What SuperDroid IS:
- 30 slash commands for structured workflows
- 20 specialized droids (subagents) for focused tasks
- 7 behavioral modes for different working styles
- CLI tools for easy installation (`superdroid install`)

### What SuperDroid is NOT (yet):
- A pytest plugin (SuperClaude has this, we don't)
- A Python execution engine (no pm_agent, parallel execution)
- MCP server integration code (prompt-level only)

## 📂 Project Structure

```
SuperDroid/
├── .factory/
│   ├── commands/        # 30 slash commands
│   ├── droids/          # 20 specialized droids
│   └── modes/           # 7 behavioral modes
│
├── src/superdroid/      # Python CLI package
│   ├── __init__.py
│   └── cli/
│       ├── main.py      # CLI entry point
│       ├── install_commands.py
│       └── doctor.py
│
├── bin/                 # npm CLI
│   ├── superdroid.js
│   └── postinstall.js
│
├── pyproject.toml       # Python package config
├── package.json         # npm package config
├── install.sh           # Shell installer
└── README.md
```

## 🚀 Installation

```bash
# Option 1: pipx (Python)
pipx install superdroid
superdroid install

# Option 2: npm (Node.js)
npm install -g @frexxis/superdroid
superdroid install

# Option 3: Shell script
git clone https://github.com/Frexxis/SuperDroid.git
cd SuperDroid && ./install.sh
```

## 📊 Statistics

| Component | Count | Lines |
|-----------|-------|-------|
| Commands | 30 | ~4,600 |
| Droids | 20 | ~2,200 |
| Modes | 7 | ~700 |
| **Total** | **57** | **~7,500** |

## 🔧 CLI Commands

```bash
superdroid install        # Install all components
superdroid install --force # Force reinstall
superdroid doctor         # Health check
superdroid update         # Update (same as install --force)
superdroid list           # List available commands
superdroid version        # Show version
superdroid info           # Show framework info
```

## 📦 Package Distribution

| Platform | Package | Install Command |
|----------|---------|-----------------|
| PyPI | superdroid | `pipx install superdroid` |
| npm | @frexxis/superdroid | `npm install -g @frexxis/superdroid` |
| GitHub | Frexxis/SuperDroid | `git clone ...` |

## 🔗 References

- **SuperClaude Framework**: https://github.com/SuperClaude-Org/SuperClaude_Framework
- **Factory Droid CLI**: https://droid.ai/code

## 📝 Future Roadmap

If full SuperClaude parity is desired:
1. Add pytest plugin (`superdroid.pytest_plugin`)
2. Add PM agent modules (`pm_agent/`, `execution/`)
3. Add MCP integration code
4. Add test suite

Currently, SuperDroid focuses on the prompt/behavioral layer, which is the most impactful part for daily development workflows.

---
> Source: [Frexxis/SuperDroid](https://github.com/Frexxis/SuperDroid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
