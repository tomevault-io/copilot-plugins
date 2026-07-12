## context-management

> Keep agent context focused across tasks and stale threads


# Context Management

## Do

- Use `/clear` between unrelated tasks or features.
- Reference files with `@path` instead of pasting entire file contents.
- Use `@Past Chats` to pull in prior work instead of copy-pasting old conversations.
- Start fresh after **2 failed corrections** on the same issue: `/clear`, then write a better prompt that incorporates what you learned.
- Point website editors at `@docs/website.md` for file locations and shortcodes.

## Don't

- Let context accumulate across unrelated features in one long thread.
- Describe files vaguely when an `@` reference exists.
- Keep correcting the same mistake in a degrading context — reset instead.

## When context is stale

Signs you should `/clear` and re-prompt:

- Repeated fixes on the same bug without progress
- Agent confuses requirements from an earlier, unrelated task
- Plan has drifted significantly from what was approved

After clearing, restate: goal, approved plan (or link to `.cursor/plans/*.md`), relevant `@` files, and acceptance criteria.

---
> Source: [OWASP/MOSAIC](https://github.com/OWASP/MOSAIC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
