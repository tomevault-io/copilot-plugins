## wife

> Wife is a local memory layer for coding agents. It has zero runtime

# Wife — notes for the agent reading this (Codex reads this file)

Wife is a local memory layer for coding agents. It has zero runtime
dependencies and no build step: the source in `src/` is what runs.

## If the user wants to install it

Run these, in this order, from this folder:

```bash
npm link            # puts the `wife` command on their PATH
wife init           # creates ~/.wife
wife attach claude  # or: wife attach codex
wife status         # confirm, then show them the output
```

Then tell them to **restart their agent** so the hooks load.

`wife attach` merges into the existing settings file and removes nothing.
If `npm link` fails with a permission error, report it rather than reaching
for sudo — the usual cause is that npm's global bin directory is not on PATH.

## If the user wants to change something

Run `npm run check` before and after. It is 90 unit tests plus a 77-check
end-to-end run and it must stay green. New behaviour needs a new test.

Where things live:

- `src/core/extract.js` — the pattern rules that turn prompts into candidate
  facts, plus the rejection filters. Most feature requests land here.
- `src/core/store.js` — markdown file plus metadata sidecar, scoring, pruning.
  Hand edits to the markdown always win over the sidecar.
- `src/core/harvest.js` — the curator: extract, gate, reconcile, prune.
- `src/agents/` — one file per agent integration.
- `src/commands/hooks.js` — code that runs inside a hook. It must never exit
  non-zero and must never print anything unexpected to stdout.

## Rules that are not negotiable

- Never mine the agent's own output. Wife reads user prompts only.
- Never let a credential reach disk. `src/core/redact.js` screens every
  candidate; a match drops the whole candidate rather than masking it.
- Never remove the token ceiling. It is the reason the memory stays trustworthy.

---
> Source: [ma-nucho-pro/wife](https://github.com/ma-nucho-pro/wife) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
