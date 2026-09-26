## genesis

> Gemini harness adapter -- coming soon.


:::caution[Coming soon]
The Gemini harness adapter is not yet documented. The Genesis corpus does not currently ship a `gemini.md` per-harness affordance file; this is tracked as an open Genesis corpus issue separate from the doc site launch.
:::

## Status

Gemini support is planned but not yet shipped. The substrate primitives ([primitives](/genesis/reference/primitives/)) are harness-agnostic and apply unchanged; only the per-harness file-path / frontmatter adapter is missing.

When the adapter lands it will follow the same shape as the other harness pages:

- File paths for each primitive.
- How `/genesis` resolves on Gemini's dispatcher.
- Frontmatter dialect for personas and rules.
- Tool surface preloaded by Gemini.
- Known gotchas.
- Verified link to official Gemini documentation.

## Official documentation (for now)

Until the adapter ships, consult the official Google sources directly:

- [Gemini API docs (ai.google.dev/gemini-api/docs)](https://ai.google.dev/gemini-api/docs) -- the Gemini API reference. (Verified 200 OK.)
- [Gemini CLI source (github.com/google-gemini/gemini-cli)](https://github.com/google-gemini/gemini-cli) -- canonical source for the CLI agent surface. (Verified 200 OK.)

## Track this gap

The missing adapter is tracked as an open Genesis corpus issue. If you have validated Genesis against Gemini in practice, contributions to the adapter are welcome -- see the [Genesis repository](https://github.com/danielmeppiel/genesis).

## See also

- [Harness setup overview](/genesis/reference/harnesses/) -- the matrix of all supported harnesses.
- [Primitives](/genesis/reference/primitives/) -- the substrate concepts that will apply once the adapter ships.

---
> Source: [danielmeppiel/genesis](https://github.com/danielmeppiel/genesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
