## typesafe-mcp

> `evaluate`: a Go MCP stdio server that forwards typed questions to TypeSafe Jev. One static binary. README.md is user-facing. Read this before the first action; it overrides generic defaults.

# typesafe-mcp

`evaluate`: a Go MCP stdio server that forwards typed questions to TypeSafe Jev. One static binary. README.md is user-facing. Read this before the first action; it overrides generic defaults.

Load by branch:

| Branch | File |
|---|---|
| Editing under `cmd/` | [`cmd/evaluate/AGENTS.md`](cmd/evaluate/AGENTS.md) |
| Writing `evaluate` questions, interpreting answers, choosing noul/choice/score | [`docs/jev/README.md`](docs/jev/README.md) |
| Domain terms (System One, canonical surface, adapter) | [`CONTEXT.md`](CONTEXT.md) |

## Verification is the job

Writing the patch is the easy part. If you cannot demonstrate the change works, you have not finished.

Done here means `task check` (gofmt, `go vet`, `go test -race -count=1`). Tool-contract or MCP transport changes also need `task inspect` or a stdio round-trip against `./evaluate`. A bug is done when the reproduction is red, then the same steps are green.

## Before code

1. Reproduce. Get the bug red (test or a real `evaluate` call). For a feature, name the current behaviour you will change.
2. Restate the task: what you will do, what you will not, what you assumed.
3. One sentence for the proof you will attach later.
4. If asked to investigate, stop at the diagnosis. Do not start fixing because you spotted something on the way.

## While working

- One scoped concern per change.
- Run the binary when the change is in the tool, setup, or stdio path — `task check` does not click through MCP for you.
- Fix the cause. If you are about to work around, say why the real fix is out of scope.
- Use what this repo already chose (cobra, the MCP SDK, stdlib).
- Jev and TypeSafe facts come from `docs/jev` or https://docs.typesafe.ai. Do not invent endpoints, fields, or model behaviour.
- User-facing strings (README, MCP tool prose, errors) stay in the product language from `tools.go`. Prototype names, TODOs, and agent reasoning stay out.

## Proof

| Change | Proof |
|---|---|
| Bug fix | The reproduction, then the same steps passing |
| MCP / tool contract | `task check` plus inspect or a captured stdio exchange |
| Setup / pi adapter | The client path, or `node --check` / `pi -e` as in `cmd/evaluate/AGENTS.md` |
| Refactor | `task check` showing behaviour unchanged |
| Docs | The claim matches `tools.go` or live TypeSafe docs |

A PR with no proof is a draft. Also state what changed and why, what you did not change that a reviewer might expect, and anything unsure.

## Code

Comments for non-obvious *why* only. No dead code, no commented-out blocks, no debug leftovers.

`state` and `questions` are hostile input. The HTTP client, 16 MiB cap, and criteria `validate` are the boundary. Jev answers come from the API — do not fabricate probabilities. Secrets stay out of logs and MCP stdout. `evaluate mcp` stdout is MCP-only.

## When corrected

Apply the fix, then write the general rule here or in `cmd/evaluate/AGENTS.md`. Principle, not the incident.

Bad: "When editing `setup.go`, watch the Claude Desktop rollback because it failed on 2026-09-18."
Good: "A failed adapter write must leave the previous registration intact."

## Stop and ask

Keys and billing routes, destructive git, a production publish, a product call you would have to guess, two failed attempts at the same approach. Two failures means the approach is wrong — report what you tried.

Status is done / in progress / blocked. No change since last time → say so, or stay silent.

## Workflow

- `task check` before calling work done.
- `task build` writes `./evaluate` (gitignored). `task inspect` opens the MCP Inspector against that build.

## Commits

Conventional Commits drive release-please: `feat` minor, `fix` patch, `!` breaking (still minor pre-1.0), `docs`/`test`/`chore` no bump. Scopes in use: `tools`, `setup`, `client`, `cli`, `readme`. `CHANGELOG.md` and `.release-please-manifest.json` are generated — the release workflow owns them.

---
> Source: [PyModel/typesafe-mcp](https://github.com/PyModel/typesafe-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
