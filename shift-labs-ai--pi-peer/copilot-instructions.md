## pi-peer

> A pi extension that lets pi sessions on one machine message each other through

# Working in this repo

A pi extension that lets pi sessions on one machine message each other through
a shared directory of mailboxes.

Start with [README.md](README.md) for what it is and
[ARCHITECTURE.md](ARCHITECTURE.md) for how it works and why.

## The gate

```bash
bun run check      # typecheck + lint + full suite
```

Run it before reporting work complete. Not "the tests I think are relevant" —
the gate. If it cannot run, say what blocked it.

## Rules that matter here

**Mail is never destroyed to tidy something up.** A sender that was told its
message was queued has to be right. Any change to `sweepPeers`, `removeRecord`,
or shutdown handling must keep that true, and the tests that pin it are not
negotiable.

**The test suite is the specification.** Each case states the guarantee it pins
and why. Changing behaviour means changing a stated guarantee, deliberately,
with the comment updated to explain the new one. Never weaken a case to make a
change pass.

**Verify against reality, not against your own summary.** The bug that ate a
letter passed every unit test — it only appeared when two real pi sessions ran
in split panes. Behaviour claims about the extension need a real session or a
transcript, not a green suite.

**The strings are the interface.** What `format.ts` produces is the whole of
what the model sees. Treat those tests like an API contract, and keep the
boundary preamble on every delivery.

**Read from disk before rewriting.** Files change between turns, including by
your own earlier commits. Rewriting a file from an in-context copy silently
reverts anything you have not seen.

**Comments explain why.** What the code does is visible in the code. Say what
would break if it were written the obvious way instead.

## Conventions

- Tabs, 120 columns, enforced by biome. Run `bun run format`.
- `src/peer/` has no pi dependency and is testable standalone. Keep it that way.
- `src/extension/` is the only file that knows about pi, and stays thin.
- Import pi types from `@earendil-works/pi-coding-agent`, and keep it and
  `typebox` as `"*"` peer dependencies — pi bundles both.
- Commit messages name the behaviour that changed and the reason, not the files.

---
> Source: [shift-labs-ai/pi-peer](https://github.com/shift-labs-ai/pi-peer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
