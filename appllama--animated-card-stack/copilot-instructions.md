## animated-card-stack

> This is the Appllama MCP prompt deck. The choreography in

# Working on this repository

This is the Appllama MCP prompt deck. The choreography in
`src/scene/motion.ts` is the product; everything else serves it. The reference
canvas is 1240 × 1240 logical units, projected to whatever size the container
is.

`src/remotion/` and `scripts/` are capture tooling for the README artwork, not
a feature. They read the same tables, so changes to the choreography must keep
both in step — but never let capture concerns drive the deck's design.

Rules that keep the deck coherent:

- Card copy stays live DOM text. No text baked into images.
- The content layer is never blurred. Only the glass surface layer takes the
  depth and velocity blur.
- Everything the cards display comes from `src/cards.ts`. Do not hardcode card
  copy in components.
- Timing is authored in whole 59.94 fps frames so live playback and rendered
  video stay frame-identical.
- Run `npm run verify` before proposing a change; run
  `npm run remotion:compositions` when touching anything under `src/remotion/`.

The Appllama masthead and the seven capability cards are the showcase — keep
them. The source-app icons and client marks are third-party trademarks used for
identification only: do not add new ones, do not imply affiliation, and keep
`NOTICE.md` and the README's intellectual-property section accurate whenever
the deck changes.

---
> Source: [Appllama/animated-card-stack](https://github.com/Appllama/animated-card-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
