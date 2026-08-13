## songbook

> A how-to-sing songs reference library: one guide per song under `guides/`

# songbook — working notes

A how-to-sing songs reference library: one guide per song under `guides/`
(`guides/<slug>.md`), drawn from a large karaoke catalogue. Cross-linked with
`aaroncodex` via `SINGING_LIBRARY_LINK.md`.

## The one hard rule

**No guide ever computes its own `/10` score.** Guides may reference raw measures
(cents, dB, %), but the only thing permitted to produce an overall score is the
aaroncodex engine (`voxanalysis/vox-analysis/engine/analyse_song.py`). See
`SINGING_LIBRARY_LINK.md`.

By convention, changes to `SINGING_LIBRARY_LINK.md` are made in both songbook and
aaroncodex in the same session.

## Memory

Durable cross-session facts live in `memory/` — read `memory/MEMORY.md` at the
start of a session. It carries the shared session memory (facts 021–029),
mirrored across the sibling repos; the aaroncodex copy is primary.

---
> Source: [goodermine/songbook](https://github.com/goodermine/songbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
