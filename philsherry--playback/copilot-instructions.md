## playback

> Do not make any changes until you have 95% confidence in what you need to build. Ask the user questions until you reach that confidence level. Write the plan out and confirm with the user before starting.

# Playback

## Helping the user

Do not make any changes until you have 95% confidence in what you need to build. Ask the user questions until you reach that confidence level. Write the plan out and confirm with the user before starting.

## Testing

Run the relevant tests before finishing any edit:

```sh
npm run test:ts      # TypeScript tests (must pass)
npm run test:tui     # Go tests (must pass)
npm run typecheck    # type checking (must pass)
npm run lint         # all linting (advisory)
```

Read @.agents/TESTING.md

## Project structure

Read @.agents/STRUCTURE.md

## Pipeline

Read @.agents/PIPELINE.md

## Tapes

Read @.agents/TAPES.md

## TUI

Read @.agents/TUI.md

## TUI UI package

Read @.agents/TUI_UI.md

## Timeline

Read @.agents/TIMELINE.md

## Memory

Session memory lives in `.claude/memory/`. The index is `.claude/memory/MEMORY.md`.

## Applied learning

When something fails again and again, when the user has to re-explain or tell you off, or when you or the user find a workaround for a platform/tool limitation, add a one-line bullet point here. Keep each bullet point under 15 words. No explanations. Only add things that will save time in future sessions. Inform the user when you add new points.

- Stop rerunning validation after the user confirms it already passed.
- Never commit without explicit user instruction. One clean commit per PR.
- `__dirname ?? fallback` throws in ESM — guard with an existence check, not `??`.
- `test:smoke` runs built `dist/cli.js` against real tapes; `tsx` masks ESM-only bugs.
- VHS `Type` types characters verbatim — `\\` is two backslashes, not one. Don't escape `\` in `escapeVhs`.
- VHS `Type "..."` has no escape for `"` — schema rejects commands containing double quotes.
- `release:check` tape guard skips patch releases (`x.y.Z > 0`); only minor/major need a release video.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/philsherry) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
