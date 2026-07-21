## agentic-coding-live

> This repo builds a presenter mode for `unbounded_contexts_slides/unbounded_contexts_slides.html`,

# Project: Presenter App (live workshop build)

This repo builds a presenter mode for `unbounded_contexts_slides/unbounded_contexts_slides.html`,
a single-file HTML slide deck. See `PLAN.md` for the feature scope and `commands.md` for the
command cheat sheet.

## Verify loop is required for every feature

A feature or bug fix is not done when the code looks right. It's done when a Playwright test
in `tests/` actually drives the real behavior in a real browser and passes.

- Before starting a feature, either extend an existing spec or add a new one in `tests/` that
  encodes the expected behavior (e.g. "pressing P hides the nav buttons").
- Run `npx playwright test` and confirm it's red for the right reason (the feature doesn't
  exist yet), then make it pass.
- A change isn't finished until `npx playwright test` is green. Don't declare something working
  from reading the diff alone.
- If a test fails in a way that isn't about the feature being built (like the pre-existing
  slide-counter bug), treat that as a real finding to report, not something to quietly work around.

## Commits

Never commit without being explicitly asked, each time. An earlier approval doesn't carry
forward to later changes in the same session.

---
> Source: [chelseatroy/agentic-coding-live](https://github.com/chelseatroy/agentic-coding-live) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
