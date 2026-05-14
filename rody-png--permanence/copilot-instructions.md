## permanence-context

> Permanence product and architecture context


# Permanence Context

`permanence-pitch.md` is the product source of truth. Permanence is a local-first personal creative memory system for preserving first-person cultural encounters at the level of synthesis, not clipping.

The core domain object is a Pearl. Preserve these layers unless the user explicitly rethinks the product model:
- Envelope metadata: title, source, source type, encounter date, tags, and mood.
- Experiential record: first-person account of the encounter.
- Intellectual synthesis: analysis, questions, lessons, and connections to the user's practice.
- Professor transcript: archived AI dialogue from the creation or reflection process.
- Connection graph: annotated links between Pearls.
- Attachments: optional supporting media.

Current implementation boundaries:
- `src/lib/pearls/types.ts` defines the Pearl domain model.
- `src/lib/pearls/store.ts` owns local persistence and CRUD. It currently uses `localStorage` keys `permanence.pearls.v1` and `permanence.pearl-positions.v1`.
- `src/lib/professor/adapter.ts` is the seam for replacing the mock Professor with an OpenRouter/Hermes-backed session.
- `src/lib/search/adapter.ts` is the seam for replacing mock threading with semantic retrieval and generated briefings.

The current spatial/glass/pearl-table aesthetic has been archived as the "champagne" design for historical reference, not as the direction to continue. For redesign work, keep the backend/domain concepts above, but freely rethink the visual language, interaction model, layout, and information architecture unless the user asks to recover the archived UI.

---
> Source: [rody-png/Permanence](https://github.com/rody-png/Permanence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
