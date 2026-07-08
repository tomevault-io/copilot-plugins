## lorepunk-specialist

> Custom marketing intelligence agent for Margaret (lorepunk). Built on the Kintsugi Engine with Apolaki marketing intelligence and a Claude Code-style tool scaffold.

# CLAUDE.md — Project Lorepunk

## What Is Lorepunk

Custom marketing intelligence agent for Margaret (lorepunk). Built on the Kintsugi Engine with Apolaki marketing intelligence and a Claude Code-style tool scaffold.

An AI marketing strategist that can read files, write copy, execute code, analyze data, research competitors, and build campaigns — all through conversation.

## Architecture

```
User (CLI or Web) → Scaffold Engine → LLM (fine-tuned 120B on Mac Studio)
                         ↓                        ↓
                   Tool Registry            Tool Calls
                    ├─ File tools     ←── read/write/edit/list
                    ├─ Code tools     ←── bash/python
                    ├─ Web tools      ←── search/fetch
                    ├─ Marketing tools ←── campaigns/content/analytics
                    └─ Crypto tools   ←── tokenomics/regulatory/community
```

## Key Directories

```
lorepunk/
  scaffold/    Engine + tool registry (the Claude Code pattern)
  tools/       File, code, web, marketing, crypto tools
  marketing/   Apolaki-derived marketing intelligence
  crypto/      Crypto/Web3 domain expertise
web/           Web UI (Flask)
config/        VALUES.json, model config
tests/         Test suite
```

## Usage

```bash
# CLI (Claude Code style)
python -m lorepunk --model lorepunk:latest --workspace ./my-project

# With custom Ollama
python -m lorepunk --api-base http://192.168.1.100:11434

# Disable code execution (safe mode)
python -m lorepunk --no-bash --no-python
```

## Hardware

Target: Mac Studio M3 Ultra (256GB RAM, 4TB disk)
Model: Fine-tuned 120B (Oracle training data via Vera)

## Running Tests

```bash
python -m pytest tests/ -x -q
```

## Team

- **CC (Coalition Code)** — Scaffold architecture
- **Vera** — Model fine-tuning
- **Thomas Edrington** — Project coordination
- **Margaret (lorepunk)** — Client + compute provider

---
> Source: [HumboldtJoker/Lorepunk-Specialist](https://github.com/HumboldtJoker/Lorepunk-Specialist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
