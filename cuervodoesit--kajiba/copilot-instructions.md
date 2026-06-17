## kajiba

> - **Description:** Community data pipeline that collects, standardizes, scrubs, and exports real-world usage data from Hermes Agent sessions to produce training data for open-source local models.

# Kajiba — Hermes Agent Skill Manifest

## Skill info

- **Name:** kajiba
- **Version:** 0.1.0
- **Description:** Community data pipeline that collects, standardizes, scrubs, and exports real-world usage data from Hermes Agent sessions to produce training data for open-source local models.
- **License:** Apache 2.0

## Slash commands

| Command | Description |
|---------|-------------|
| `/rate [1-5] [tags...]` | Rate the current session. Tags from controlled vocabulary. Example: `/rate 4 task_completed minor_hallucination` |
| `/report [category] [text]` | Submit a pain point report. Example: `/report tool_call_failure Model tried to use docker_compose tool` |
| `/kajiba preview` | Preview what would be submitted for the current session |
| `/kajiba submit` | Submit the current session (after preview + confirmation) |
| `/kajiba history` | Show past submissions and their status |
| `/kajiba config` | View Kajiba configuration |
| `/kajiba export [path]` | Export current session as Kajiba-format JSONL to local file |
| `/kajiba stats` | Show local statistics |

## Required tools

None — Kajiba is a passive collector that hooks into the agent's session lifecycle.

## Configuration

Add to `~/.hermes/config.yaml`:

```yaml
kajiba:
  consent_level: full          # anonymous | trajectory_only | metadata_only | full
  auto_submit: false           # Never auto-submit; always preview first
  llm_pii_scrub: true          # Run LLM-based scrubber (uses local model)
  scrub_strictness: high       # low | medium | high
```

## Installation

```bash
pip install -e ".[all]"
```

## Integration

Kajiba registers hooks into the Hermes Agent session lifecycle via `register_hooks(agent)`. It captures turns, tool calls, and metadata non-intrusively. If Kajiba encounters an error, it logs the issue and never interrupts the host agent session.

---
> Source: [CuervoDoesIt/Kajiba](https://github.com/CuervoDoesIt/Kajiba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
