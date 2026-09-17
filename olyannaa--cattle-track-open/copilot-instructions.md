## cattle-track-open

> This repository is a public NDA-safe slice of the Cattle Track AI assistant. Keep it focused on the assistant layer only.

# Agent Guide

This repository is a public NDA-safe slice of the Cattle Track AI assistant. Keep it focused on the assistant layer only.

## Scope

Allowed:

- AI assistant architecture and C#/.NET 8 reference code.
- ASR, normalization, LLM tool-calling, safety gate, and confirmation flow.
- Public JSON contracts and example configuration.
- Documentation that explains the public slice.

Not allowed:

- Full Cattle Track backend or frontend source code.
- Real database schema, migrations, seed data, production endpoints, or internal service implementations.
- Private datasets, experiment artifacts, farm data, customer data, or secrets.
- Internal agent logs, handoff notes, task journals, or scratch documents.

## Repository Layout

- `ai-assistant/src` - C# reference assistant code.
- `ai-assistant/contracts` - public tool-calling contracts.
- `ai-assistant/config` - example runtime settings.
- `ai-assistant/examples` - sanitized examples.
- `docs` - human-readable technical notes.

## Editing Rules

1. Keep README focused on the public assistant slice.
2. Do not add fullstack application code.
3. Do not add real `.env` files. Use `.env.example` or files under `ai-assistant/config`.
4. Keep tool contracts deterministic and schema-first.
5. Write operations must stay behind safety gate and human confirmation.
6. If a change would require private Cattle Track internals, represent it as an adapter interface instead.

## Verification

Before finishing a change, run checks that match the touched files. At minimum, validate JSON files:

```bash
python3 -m json.tool ai-assistant/contracts/tool-registry.v2.1.json >/dev/null
python3 -m json.tool ai-assistant/config/assistant.example.json >/dev/null
```

---
> Source: [olyannaa/cattle-track-open](https://github.com/olyannaa/cattle-track-open) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
