## agentified

> **Agentified** — Context Intelligence Layer for AI agents. This repo contains `agentified-core`, a Rust HTTP server for tool registration and context resolution (intelligent tool selection).

# CLAUDE.md

## Project Overview

**Agentified** — Context Intelligence Layer for AI agents. This repo contains `agentified-core`, a Rust HTTP server for tool registration and context resolution (intelligent tool selection).

## Project Structure

```
agentified/
├── src/
│   ├── core/             # Rust server
│   ├── ts-packages/      # TypeScript packages (pnpm workspace)
│   │   ├── sdk/
│   │   ├── fe-client/
│   │   ├── react/
│   │   └── mastra/
│   └── py-packages/      # Python packages
├── examples/             # Example applications
├── benchmarks/           # Benchmarks
├── scripts/              # Test scripts
└── README.md
```

## Environment Variables

- `OPENAI_API_KEY` — required for embeddings
- `AGENTIFIED_PORT` — server port (default: 9119)

---
> Source: [agentified/agentified](https://github.com/agentified/agentified) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
