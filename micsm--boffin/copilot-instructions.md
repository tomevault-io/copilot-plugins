## boffin-post-change-audit

> description: Pre-flight review and post-change kernel audit

---
description: Pre-flight review and post-change kernel audit
alwaysApply: true
---

# Pre-Flight Review and Post-Change Audit

## Before presenting code-task results

- Re-read the loaded pack index and all loaded leaf packs; do not rely on memory.
- For a focused change, confirm you stayed within the requested scope and proved
  the change with a check; the ledger checks below apply to refactor/review tasks.
- For a refactor/review task, confirm the Pass 1 audit ledger lists a row for
  every in-scope stage; a stage with no row is an incomplete audit, not a clean
  file.
- Confirm every ledger row is `done` or `skip`, that each `skip` carries a
  reason, and that no row recorded as `todo` was downgraded to `skip:clean`.
- Confirm every fix named in Pass 1 reasoning has its own ledger row; a fix
  discussed but left unrecorded is an incomplete audit.
- If a change touches ownership, lifecycle, completion semantics,
  state-machine transitions, concurrent access, or logic currently encoded
  through flags or sentinels, name the exact invariant from stages S01-S03 it
  must preserve before editing.
- On user request, render the ledger and the blocking earlier-stage invariants
  as concise ordered bullets in the user's language.

## After any code edit

- MUST verify the edit with an external check (the narrowest test or lint that proves it), not by re-reading your own reasoning or the diff alone.
- MUST review the change as if a different author wrote it: read the final file region, not only the diff, and ask what a fresh reviewer would flag.
- MUST re-read the relevant loaded leaf pack(s) and confirm no `## EXCLUDES` pattern was introduced and no earlier-stage (S00-S03) invariant was weakened to satisfy a later-stage (S04-S06) cleanup goal.
- MUST NOT silently drop a ledger finding: it ends as `done` (with its check) or `skip:<reason>`.
- MUST present the filled ledger as the result; do not add rows for kernels that found nothing, and never cite a kernel a row does not actually rest on.
- When a loaded kernel actually changed a code decision, leave one short host-language comment at that site: start it with the `boffin:` prefix and state, in the code's own vocabulary, what the code now does or refuses, never restating the kernel. The judgment verb is encouraged (`kept .../refused .../cut ...`); the `boffin:` prefix is the machine-readable key; the clause MUST NOT contain a kernel id, kernel scope-names, or any other pack vocabulary. Readability bar (the one mandatory wording check): mentally dropping the `boffin:` prefix, the remaining sentence MUST still read as a comment a senior engineer would write -- otherwise rewrite the clause or drop the mark; this is only a mental check, so always keep the `boffin:` prefix in the emitted comment. MUST NOT mark a purely stylistic rewrite of already-correct code that averts no failure and preserves no load-bearing invariant, nor emit anything when a `.boffin-trace-off` file exists at the repo root. Keep to about one mark per hunk and roughly three per changeset; no URLs, marketing, or tool jargon; only in venues that forbid tool marks, omit the `boffin:` prefix (keep the senior-voice clause alone).

---
> Source: [MicSm/boffin](https://github.com/MicSm/boffin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
