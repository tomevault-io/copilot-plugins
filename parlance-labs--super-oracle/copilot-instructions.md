## super-oracle

> Rationale and foot-guns behind `scripts/super-oracle.sh`. Keep this out of the

# Dev notes

Rationale and foot-guns behind `scripts/super-oracle.sh`. Keep this out of the
README (user-facing) — it's for maintainers.

## Repository layout

```
super-oracle/
├── .agents/plugins/marketplace.json      # Codex marketplace
├── .claude-plugin/marketplace.json       # Claude Code marketplace
├── plugins/super-oracle/
│   ├── .codex-plugin/plugin.json         # Codex manifest
│   ├── .claude-plugin/plugin.json        # Claude manifest
│   └── skills/super-oracle/              # the portable skill (used by Amp too)
│       ├── SKILL.md
│       ├── reference/briefing-template.md
│       └── scripts/super-oracle.sh
└── scripts/smoke-test.sh
```

## Test

A minimal-token smoke test confirms `codex-fugu` works and runs on `fugu-ultra`.
Cheap, not necessarily fast. Exits 0 on success, skips if `codex-fugu` isn't
installed.

```bash
scripts/smoke-test.sh
```

## Always Fugu Ultra
`codex-fugu` defaults to plain `fugu`. The script forces `-c model=fugu-ultra`
and intentionally exposes no model flag.

## MCP left on by default
Unauthenticated MCP servers print harmless `Auth required` warnings at shutdown
but don't block — measured overhead ~2s, immaterial for a long-running oracle.
So we leave MCP on and just filter the cosmetic stderr noise. `-n` turns it off
for a server that genuinely hangs.

`-c mcp_servers="{}"` does NOT disable MCP in `exec` mode (codex deep-merges
config). `-n` instead runs with a temp `CODEX_HOME` whose `config.toml` has all
`[mcp_servers.*]` tables stripped (requires `python3`).

## Permission posture is approximated, not inherited
Codex does not expose the parent's approval/sandbox policy to children — only
`CODEX_SANDBOX` and `CODEX_SANDBOX_NETWORK_DISABLED`. `CODEX_SANDBOX` is a
backend marker (`seatbelt` on macOS), NOT a `--sandbox` policy value, so it can't
be passed through. Logic:

1. `SUPER_ORACLE_SANDBOX` (validated) wins.
2. `SUPER_ORACLE_BYPASS=1` → bypass; `=0` → `workspace-write`.
3. Else any `CODEX_SANDBOX` / `CODEX_SANDBOX_NETWORK_DISABLED` signal → stay
   conservative (`workspace-write`), never escalate.
4. Else (unattended) → `--dangerously-bypass-approvals-and-sandbox`.

## Success = output produced, not exit code
A broken/expired MCP server can make `codex-fugu` exit non-zero even when the
answer is fine. The script judges success by whether the `-o` file is non-empty.
The `-o` file is truncated before any failure-prone step (missing dependency,
`-n` rewrite, `cd`, run) so a stale file can never look like success.

The post-run tail intentionally runs under `set +e` and ends with an explicit
`exit 0`. Reason (found by dogfooding): with `set -e`, a benign `[ test ] && echo`
short-circuit can make a *successful* run exit non-zero, so callers that chain on
the exit code break. Because the tail has no `set -e`, every operation that must
not fail silently (recovering `answer.md` into `-o`, enumerating artifacts,
appending the manifest) is guarded with an explicit `if ! ...; then ... exit 1`.
Keep that pattern when editing the tail; do not re-add `set -e` there.

## Two-channel output (control plane vs data plane)
`codex-fugu`'s `-o` captures ONLY the final assistant message. A long answer the
oracle writes to a file would otherwise be lost (and an earlier review run was
wasted when such a file got deleted as "stray output"). Fix:

- **Control plane** = `-o`: the script prepends an output contract telling the
  oracle to put its full answer in the final message.
- **Data plane** = `SUPER_ORACLE_ARTIFACTS_DIR` (default `OUTPUT.artifacts`):
  the only sanctioned place for supporting files. Cleared/recreated each run,
  removed at the end if empty so it doesn't litter.

Robustness: if `-o` is empty but `$ARTIFACTS_DIR/answer.md` exists it's recovered
into `-o`; any artifacts are listed as a manifest appended to `-o` with a
"do not delete before reading" note. Never treat oracle-emitted files as
disposable.

Footgun (found by real runs, not fakes): the oracle runs with our perms and sees
the exported `$SUPER_ORACLE_ARTIFACTS_DIR`, so it can delete its own artifacts dir
mid-run. The tail therefore treats a MISSING artifacts dir as "no artifacts" (not
an error); only a `find` failure on a dir that still exists is fatal. An earlier
"wrapper exits 1 on a successful real run" mystery was exactly this.

## Progress heartbeat (UX)
A long run feels dead without feedback, so the script keeps the user informed on
stderr (never `-o`):
- a startup line that names Fugu Ultra, sets the "few minutes" expectation, and
  states the answer path (repeated in plain language because a harness's small
  output window may scroll the technical announce line away);
- a background `heartbeat` that prints `still working — Xm Ys elapsed · …normal`
  every `SUPER_ORACLE_PROGRESS_INTERVAL` seconds (default 30; `-p` overrides; `0`
  disables), with the FIRST beat within ~15s so it never looks hung;
- a `done in Xm Ys` line at the end (uses `RUN_START`).

Design choices learned from testing real Codex/Claude TUIs:
- Each heartbeat is self-contained (elapsed + "this is normal") because harnesses
  show only a tail/window of a running command, so a one-time startup note isn't
  enough.
- We deliberately DROPPED the old "latest oracle line" feature: codex-fugu's
  stderr is mostly internal `WARN`/log noise, which looked like errors. A clean
  elapsed counter is clearer and never alarming. (This also removed the progress
  temp-log + `tee`.)
- `grep --line-buffered` is still required so codex-fugu's own stderr streams live
  rather than clumping at the end (BSD/macOS and GNU both support it).
- The stderr filter also drops all `WARN codex_` lines (codex-internal warning
  namespaces, e.g. a 503 plugin-catalog fetch or model-personality notice) so a
  real run doesn't open with a wall of scary-looking-but-benign warnings. ERROR
  lines are never filtered. The one blind spot Fugu flagged: a `WARN codex_` could
  hide a consequential-but-non-fatal notice (silent model fallback off fugu-ultra,
  context truncation) that still yields a non-empty `-o` and so escapes the
  empty-output check. Escape hatch: `SUPER_ORACLE_VERBOSE=1` swaps the filter for
  `cat` to show raw stderr. Don't hand-narrow the regex namespace-by-namespace
  (brittle as codex adds more); the verbose hatch is the intended recovery path.
- The heartbeat runs `sleep` as an explicit child killed via a TERM/INT trap, so
  `stop_heartbeat` never orphans a `sleep` that would hold stderr fds open.

## Releasing
Marketplace installs key updates off the plugin `version`. Bump `version` in both
`plugins/super-oracle/.codex-plugin/plugin.json` and
`plugins/super-oracle/.claude-plugin/plugin.json` on every published change and
tag the release.

---
> Source: [parlance-labs/super-oracle](https://github.com/parlance-labs/super-oracle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
