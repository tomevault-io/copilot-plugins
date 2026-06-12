## nexgen-hermes-template

> | Agent | Status | Bot |

# AGENTS.md — nexgen.hermes.template
# NexGen IT Crew — Bill Carter

## Known NexGen Hermes Agents

| Agent | Status | Bot |
|-------|--------|-----|
| Amand | LIVE | @amand_nexgen_bot |
| Michael | PLANNED | — |
| Gabriel | PLANNED | — |

## Deploying a New Agent

  git clone https://github.com/billco70/nexgen.hermes.template <agent-name>
  cd <agent-name>
  python setup.py

## Rules That Never Change

1. MCP is the preferred protocol for all external integrations
2. All secrets via .env — never hardcoded
3. One container per agent — no shared containers
4. hermes-data/ is the agent brain — never delete
5. TELEGRAM_ALLOWED_USERS must be set — no open bots

---
> Source: [billco70/nexgen.hermes.template](https://github.com/billco70/nexgen.hermes.template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
