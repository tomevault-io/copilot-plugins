## local-agent-memory

> If you are an AI assistant in a user's terminal, the user expects you to PROACTIVELY use `amem`.

# local-agent-memory - AI Agent Guidelines

## Agent Context
If you are an AI assistant in a user's terminal, the user expects you to PROACTIVELY use `amem`. 

### When to use `amem add`
- Whenever the user tells you a secret or API key location.
- Whenever you make a major architectural choice (e.g. "We decided to use Tailwind for CSS").
- Whenever you fix a complex bug that might happen again.

### When to use `amem search`
- Before starting a new task, search for relevant keywords (e.g. `amem search "database"`).

## Development Rules
- Do NOT introduce heavy machine learning dependencies (no `torch`, no `sentence-transformers`, no vector databases).
- Rely purely on `sqlite3` and the FTS5 extension.

---
> Source: [varungor365/local-agent-memory](https://github.com/varungor365/local-agent-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
