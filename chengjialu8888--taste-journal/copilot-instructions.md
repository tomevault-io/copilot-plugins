## taste-journal

> Treat this repository as a local-first visual memory system, not a generic asset manager.

# Agent operating contract

Treat this repository as a local-first visual memory system, not a generic asset manager.

Before changing memory, read:

1. `docs/product-contract.md`
2. `docs/architecture.md`
3. `docs/design-md-schema.md`
4. the relevant source, atom, scene, and style records

Rules:

- Preserve evidence before synthesis.
- Never infer the user's emotional meaning from pixels. Ask or leave an open question.
- Separate observation, inference, user meaning, and uncertainty.
- Never promote a single-source pattern to active taste. Require three independent sources or explicit human confirmation.
- Preserve contradictions; they are signals, not lint failures.
- Every active claim must drill down to a source record.
- Treat `vault/private/` and `vault/workspace/` as private. Never commit their contents.
- Treat the user-selected local folder as the runtime source of truth. Never keep captured media only in React state, localStorage, IndexedDB, or a remote service.
- Do not report a capture as preserved until both the original and its `sources/*.md` record are written.
- Do not invent exact fonts, creator intent, cultural provenance, or unsupported Figma fidelity.
- For creator distillation, resolve identity and compare at least three named works; keep creator statements, criticism, and inference distinct.
- Never use a creator's name as a style shortcut. Translate evidenced construction principles into original, target-domain rules.
- Run `npm run vault:lint` after changing Markdown and `npm run artifacts:generate` after changing `design.md`.
- Run `npm run verify` before publishing.

When blocked, preserve the source and state, report the unsupported step, and leave a deterministic retry path.

---
> Source: [chengjialu8888/Taste_journal](https://github.com/chengjialu8888/Taste_journal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
