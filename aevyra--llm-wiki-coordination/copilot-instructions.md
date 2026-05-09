## llm-wiki-coordination

> This file is for AI agents working on the LLM Wiki Coordination repo itself, not for end-user wikis that adopt the protocol.

# AGENTS.md — for agents working *on this repo*

This file is for AI agents working on the LLM Wiki Coordination repo itself, not for end-user wikis that adopt the protocol.

If you are an agent in a downstream wiki using this protocol, read your own wiki's `AGENTS.md` instead. The user copies the protocol fragments from `templates/` into their wiki and merges the relevant sections into their own `AGENTS.md`.

## What this repo is

A drop-in multi-agent coordination layer for [Karpathy-style LLM wikis](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). It is itself an LLM-maintained artifact, but the audience is the open-source community, not a single user's second brain.

## Hard rules for working here

1. The `templates/` folder must stay generalizable. No personal names, projects, or single-user assumptions. Use `<agent>`, `<user>`, `<topic>` placeholders.
2. The `docs/` folder explains *why* and *how*. Each protocol gets one page in `docs/protocol/`.
3. Never reference a specific second-brain implementation by its private name in committed files. Refer to "the source implementation" if needed.
4. English by default. Translations may live under `docs/i18n/<lang>/` later.
5. Update `CHANGELOG.md` for any change that touches `templates/` or protocol semantics.
6. The protocols described here have been tested in private usage. Do not invent new protocol features in this repo without first discussing in an issue or design doc.

## Layout

- `README.md` — entry point, value proposition.
- `LICENSE` — MIT.
- `docs/overview.md` — conceptual overview of the layer.
- `docs/installation.md` — concrete steps to add to an existing wiki.
- `docs/protocol/` — one file per protocol.
- `docs/concepts/` — memory tiers and typed relations explained.
- `templates/` — files the user literally copies into their `wiki/`.
- `examples/` — before/after illustrations, sample dialogue threads.

## Style

- Specs are precise and short. No marketing language.
- Examples are concrete. Show a real frontmatter block, not a description of one.
- When introducing a new term, define it on first use.

## License of contributions

Contributions are MIT-licensed by submission.

---
> Source: [AEVYRA/llm-wiki-coordination](https://github.com/AEVYRA/llm-wiki-coordination) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
