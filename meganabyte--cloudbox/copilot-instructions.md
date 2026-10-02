## cloudbox

> The README is the primary source of truth, so read that first. Everything here isn't covered in the README.

# CLAUDE.md

The README is the primary source of truth, so read that first. Everything here isn't covered in the README.

## Code starting points

Always start here: the reconcile loop and the HTTP API live in `internal/boxd/`, the CLI commands in `cmd/`, the LangGraph agent loop in `agent/agent.py`, and box bring-up plus the embedded boxd binary in `internal/provision/`.

## Building and testing

`make build` cross-compiles boxd for the box's Linux architecture, embeds it into the CLI, and then builds the CLI, while `make test` runs `go test ./...`. The one caveat is that boxd has to be built before the CLI or the tests, since the CLI embeds it. The make targets already enforce that order, so a plain `go build` or `go test` will fail until the embedded binary exists. Stick to the make targets and it won't come up.

## Avoid risky changes

Quietly breaking one of the below properties risks the whole implementation. If one of these needs to be modified, ask me first.

- boxd only reconciles. It drives the box toward the desired-state file `agent.intent`. It never provisions a box and never decides what the desired state should be. A control plane should own provisioning and desired state, in this project the CLI orchestrates as a stand-in.
- Restore runs once and only when the orchestrator hands boxd a fresh box. boxd never kicks off a restore on its own.
- boxd's HTTP API is bound to `127.0.0.1`. The CLI only ever reaches it through the SSH tunnel. Nothing is meant to listen on a public port.
- There is one session per box, keyed by `session_id`. Could change in the future, but remains true today.
- The built vs. designed markers have to stay honest. `(Implemented.)` means it runs today and `(target design)` means it doesn't. Never label something built when it isn't.

## Comments/Code Style

- Lead with what something does, edge cases accounted for, and failure modes it prevents. Keep them to 2-3 lines and skip Claude jargon.
- Use idiomatic conventions. Match naming and style of existing code/comments. 
- Avoid leaking business logic across distinct packages and files

---
> Source: [meganabyte/cloudbox](https://github.com/meganabyte/cloudbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
