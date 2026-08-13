## kotoba

> Kotoba is a language-learning Anki deck generator. Any agent host that loads

# Kotoba

Kotoba is a language-learning Anki deck generator. Any agent host that loads
`AGENTS.md` can use it: when the user asks to build flashcards / an Anki deck /
vocab cards for a language, follow `skills/kotoba/SKILL.md` as the operating
guide.

The pipeline is **plan → preview → generate → validate → package**:

1. **Plan** — lock the language pair, scope (count, level, topic, ordering),
   and extras (audio, images) with the user.
2. **Preview** — render ~5 sample cards and get explicit approval before bulk
   generation.
3. **Generate** — produce card data in batches, honoring the per-language
   reference (`references/<language>.md`).
4. **Validate** — run `scripts/validate.py`, then self-check readings and
   sentence naturalness.
5. **Package** — `scripts/build_deck.py` builds the `.apkg`.

Deep references exist for Mandarin (`references/mandarin.md`) and Japanese
(`references/japanese.md`). For any language without one, write a deep
reference first (see Phase 1 of `SKILL.md`) before generating.

Dependencies: `pip install genanki edge-tts`. Audio and images need network
access; everything else runs offline.

Full operating instructions: [`skills/kotoba/SKILL.md`](./skills/kotoba/SKILL.md).

---
> Source: [yufengliu15/kotoba](https://github.com/yufengliu15/kotoba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
