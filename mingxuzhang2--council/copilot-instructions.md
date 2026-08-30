## council

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Architecture, usage, commands, and configuration live in `README.md`. This file lists only what is
forbidden or required.

## Commands

```bash
npm test                                                   # node --test test/*.js
node --test test/council.test.js                            # one file
node --test --test-name-pattern "findSeat aliases" test/*.js # one test
npm start -- -C /path/to/repo                               # or ./bin/council
node src/cli.js --doctor                                    # local setup check, no network
```

There is no lint step and no build step.

## Constraints

- Node.js 22+ and ESM only. No transpiler, no `require`, no TypeScript.
- `bin/claude-ds` and `bin/claude-muse` must never reach Anthropic: they keep
  `ANTHROPIC_BASE_URL=$API_BASE` and their own `CLAUDE_CONFIG_DIR` under `.harness/`, and must not
  reuse `~/.claude`.
- `bin/claude-sub` must keep unsetting every `ANTHROPIC_*` variable and reading the user's own
  config directory, because the subscription's OAuth token lives there and a refresh writes back.
- Every seat stays read-only: the Claude `--disallowedTools` list, Pi's five `--no-*`
  startup-customization flags plus its `--tools read,grep,find,ls` allowlist, and Codex
  `sandbox_mode="read-only"`. Do not grant a seat a writing or shell tool.
- Codex takes its sandbox as a `-c` config override, never as `--sandbox`: `codex exec resume`
  rejects that flag and every debate round resumes.
- `--disallowedTools` is variadic, so another flag must follow it. A prompt placed directly after it
  is swallowed.
- Seat stdin stays `"ignore"`. `codex exec` appends stdin to the prompt and waits on an open pipe
  until the seat timeout.
- `bin/*` stays Bash 3 compatible. No associative arrays, no `${var,,}`, no `mapfile`.
- The generated `.harness/pi/models.json` and `.harness/codex/config.toml` are rewritten on every
  run from `.env`. Keep Pi's `User-Agent` override, which the gateway's filter requires, and Codex
  `wire_api = "responses"`, which 0.148 requires. An existing `.harness/codex/auth.json` wins, and
  `config.toml` is then deleted rather than left behind.
- `data/talks.json` is mutated only inside `withSessionLock`, and every JSON write goes through
  tmp-file plus `rename`. A corrupt history file is an error, never an empty store.
- Peer drafts reach a seat only as the untrusted `peer_claims_json` field, and seat drafts reach the
  chair only as `seat_evidence_json`. Neither may be presented as an instruction.
- A debate round reads the brief snapshot taken before the round starts, never a live `seat.log`.
- Only text that passes `isUsableClaim` is handed to peers or to the chair; a seat with no usable
  claim is dropped from the round, and a table with none skips the chair.
- All model output is passed through `sanitizeTerminal` before printing, and every width measurement
  uses `dispLen`/`wrapDisp` rather than `String.length`.
- `data/` and `.harness/` stay gitignored. They hold repository-derived text and credentials.
- `.env` is never committed and never read by `bin/claude-sub`.

---
> Source: [mingxuZhang2/Council](https://github.com/mingxuZhang2/Council) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
