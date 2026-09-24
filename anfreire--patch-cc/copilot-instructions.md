## patch-cc

> Interactive patcher for the Claude Code native binary. What it does and how to

# patch-cc

Interactive patcher for the Claude Code native binary. What it does and how to
use it: [README.md](README.md).

Before changing anything, read [docs/CONDUCT.md](docs/CONDUCT.md) — how we
build here: the mindset, the guardrails, and who commits (the user does). Then:

- [docs/PLAYBOOK.md](docs/PLAYBOOK.md) — matcher rules, the patch reference,
  how to repair a patch after a Claude update, and the map of the bundle's
  native surfaces (the model registry and its consumers).
- [docs/INTERNALS.md](docs/INTERNALS.md) — the Bun container format and how
  the binary is rewritten in place.

Verify with `uv run patch-cc doctor` (every patch against a clean bundle;
point it at the pristine copies in `~/.local/share/patch-cc/backups/` to sweep
older builds). There is no test suite by design — doctor against real bundles
is the check.

Be exact about what that covers: doctor runs the **matchers**, parses the
bundle they produce, then **bakes it into a temp binary and executes
`--version`** — so the sweep also proves every build writes and boots (the
container layer included; 2.1.246 is why). It still never runs the gateway,
the translator, the OAuth flow or the menu, so a change under
`src/patch_cc/codex/` (the bridge — all of it runtime, none of it a patch; the
Codex *patch* is `patches/codex.py`, which the sweep does cover) or in
`menu.py` is checked by exercising it — a real `codex serve` against a real
turn — and a green sweep says nothing about it. Nor does `--version` exercise
a patch's *feature* at runtime: it proves the patched code loads and runs, not
that a streamed turn renders — behaviour changes still want a real turn
watched.

---
> Source: [anfreire/patch-cc](https://github.com/anfreire/patch-cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
