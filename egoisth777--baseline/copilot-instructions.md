## baseline

> Control surface for the baseline drift-correction system — a UserPromptSubmit hook that periodically forces the agent to recite a small "baseline" of standing rules (Blade Runner style) so it stops drifting from CLAUDE.md conventions over a long session. Use this skill whenever the user wants to view, edit, add, or remove baseline rules; tune how often the recital fires (the interval N); change the recital prefix line; or install, verify, check the status of, or uninstall the baseline hook. Trigger on phrases like "edit the baseline", "add a baseline rule", "change baseline interval", "the baseline recital", "baseline drift", "make the agent recite X every N turns", or any request to manage the baseline-recital hook.


# baseline

A drift-correction system. Over a long session the agent forgets standing rules (e.g. "route file read/write/search through subagents, don't do them inline"). This installs a `UserPromptSubmit` hook that, every Nth prompt, injects a **baseline recital** — the agent must open its reply with a fixed prefix line and restate each rule verbatim before continuing. Reciting both *re-aligns* the agent (generating the rule primes the next action) and *exposes* drift (you see in the recital whether it got the rules right).

The baseline is the minimal invariant subset of rules the agent re-aligns to — like the Blade Runner 2049 baseline test.

## Architecture (what lives where)

| Piece | Path | Role |
|---|---|---|
| Hook program (portable) | `scripts/baseline-recital.zig` | Pure mechanism: count per session, fire on Nth. Cross-platform. Prebuilt binaries in `bin/`. |
| Hook program (Node fallback) | `scripts/baseline-recital.js` | Node JS fallback for systems without prebuilt binary or Zig toolchain. |
| Deployed hook | `~/.claude/hooks/baseline-recital.js` | Installed hook, invokes the portable binary or Node fallback. |
| Baseline data | `~/.claude/baseline.md` | **Everything tunable**: interval + prefix (frontmatter) + rules (body). Read live every fire — edits need NO reinstall. |
| Counter state | `~/.claude/.baseline-counters.json` | Per-session turn counts. Auto-pruned. Never edit by hand. |
| Wiring | `~/.claude/settings.json` | `UserPromptSubmit` → command entry. `install`/`uninstall` manage it. |
| Manager | `scripts/manage.js` | install / verify / status / uninstall. Run from repo root. |
| Platform installer | `install.sh` / `install.ps1` | Entry point for first install. |
| Builder | `build.sh` / `build.ps1` | Compile Zig binary from source (optional). |

**Key split:** rules/interval/prefix are *data* in `baseline.md` (edit anytime, takes effect next prompt). The hook program is *mechanism* (only changes when you alter how counting or injection works → must redeploy via `install`).

For internals (frontmatter parser, counting, hardening) read `references/architecture.md`.

## Runtimes

The baseline hook runs as:
1. **Prebuilt binary** (default) — fast, no dependencies. Ships in `bin/` for Windows, macOS, Linux.
2. **Compile from source** — run `build.sh` / `build.ps1` or pass `--build` to install script. Requires Zig toolchain.
3. **Node JS fallback** — automatic if no binary and no Zig. Portable, slower startup.

The manager picks the best available runtime automatically.

## The two most common tasks

### Edit the baseline (rules / interval / prefix) — no reinstall

This is the everyday case and it is just editing one file: `~/.claude/baseline.md`.

```
---
interval: 5                       # fire every 5th prompt
prefix: LI BASELINE ALIGNED:      # the line the agent must open with
---
# comments and blank lines ignored; one rule per line below
File read/write/search -> subagent (cavecrew-investigator/builder, Explore), not inline. Save main ctx.
```

- **Add a rule:** append a line. Keep it short — it's injected into context every fire, so caveman-compress it (drop articles/filler).
- **Change interval:** edit `interval:`. Lower = tighter leash, more tokens.
- **Change prefix:** edit `prefix:`. This is the literal line the agent recites under.
- Changes apply on the **next prompt** — the hook reads this file live. No reinstall.

After editing, you can confirm what's live with `status` (below).

### Manage the hook itself

Run the manager from the repo root:

```bash
node scripts/manage.js status      # what's installed, in-sync?, current rules
node scripts/manage.js install     # deploy hook + seed baseline.md + wire settings
node scripts/manage.js verify      # functional: confirm it fires on turn N
node scripts/manage.js uninstall   # unwire settings + delete deployed hook (keeps baseline.md)
```

Or use the platform installer scripts:
```bash
./install.sh       # Unix (sh/bash/zsh)
.\install.ps1      # Windows (PowerShell)
```

The commands are idempotent and preserve any co-resident `UserPromptSubmit` hook (e.g. caveman) — settings edits are surgical.

## Workflow guidance for the agent using this skill

1. **Identify the intent.** Editing rules/interval/prefix → it's a `baseline.md` edit, nothing more. Installing/repairing/removing → it's a `manage.js` run.
2. **For data edits:** read `~/.claude/baseline.md`, make the change, keep rules caveman-short, preserve the frontmatter fences. Then run `node scripts/manage.js status` (from repo root) to show the user the live result.
3. **For mechanism changes** (how counting/injection works): edit `scripts/baseline-recital.zig` or `scripts/baseline-recital.js`, rebuild if needed with `build.sh` / `build.ps1`, then run `node scripts/manage.js install` to redeploy, then `node scripts/manage.js verify` to prove it still fires.
4. **Always end with `status` or `verify`** so the user sees ground truth, not a claim. After any settings change, remind them: open `/hooks` once or restart so Claude Code reloads settings (the running session won't pick it up otherwise).

## Evolving the system

This skill is meant to grow. Likely future directions, and where they'd live:

- **More rules / rule sets:** just `baseline.md` body. If you ever want *named* rule sets (e.g. a "perforce" set vs a "subagent" set), that's a mechanism change — extend the frontmatter parser in `baseline-recital.zig` / `baseline-recital.js` to select a set, then redeploy.
- **Different cadence logic** (e.g. fire on token-count instead of turn-count): mechanism — edit the hook program. Note the harness gives the hook `session_id`, `transcript_path`, `cwd`, `prompt` on stdin; there is no context-percentage field, so turn-count is the reliable trigger.
- **Per-project baselines:** mechanism — have the hook read a project-local `baseline.md` (via `cwd` from stdin) and merge with the global one.

When you change mechanism, keep `scripts/baseline-recital.zig` (or `.js`) the source of truth and redeploy with `install`; never hand-edit the deployed copy in `~/.claude/hooks/`.

---
> Source: [egoisth777/baseline](https://github.com/egoisth777/baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
