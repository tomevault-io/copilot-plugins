## homeplate

> This file is the runbook for AI agents. It is deliberately procedural.

# AGENTS.md — operating Homeplate autonomously

This file is the runbook for AI agents. It is deliberately procedural.
The marketing summary is at https://homeplate.matthew-a46.workers.dev/llms.txt.

## Mission shape

The user will ask for some variant of: "install this CLI and point all my
projects' Actions at this computer." The complete, correct execution is:

```bash
brew install sikmabull/tap/homeplate        # or go install github.com/homeplate-ci/homeplate/cmd/homeplate@latest
brew install act                                # optional: offline engine
HOMEPLATE_GITHUB_TOKEN=<token> homeplate auto --json
```

`homeplate auto` is non-interactive and idempotent. It authenticates, scans
the disk for GitHub clones, links every repo the identity can admin, opens one
`runs-on:` PR per repo, installs the login daemon, and prints a JSON summary.
Report that JSON back to the user, translated to plain language.

## Hard rules (violating these is a bug)

1. **Never call it free.** Self-hosted runtime on private repos bills
   $0.002/min (GitHub control-plane fee, March 2026). Public repos are free.
   Say "up to ~31x cheaper than hosted macOS."
2. **Never spoof reserved labels.** `ubuntu-latest` etc. route to GitHub's
   hosted pool no matter what. The adoption path is `homeplate adopt`.
3. **Never link public repos** without the human explicitly approving
   `--i-understand-public-repo-risk` (fork PRs execute strangers' code).
4. **Never print tokens.** They live in the OS keychain. If a command fails
   with a keychain error, say so; do not ask the user to paste a token into
   a chat.

## Auth decision tree

1. `homeplate auth list` — if a suitable profile exists, use `--profile`.
2. Else if the user provides a token: `HOMEPLATE_GITHUB_TOKEN=... homeplate auto`
   (saved as profile `agent`, keychain-stored).
3. Else run `homeplate auth add personal` and hand the device-flow URL+code to
   the user, or ask for a fine-grained PAT and use `--pat`.

Fine-grained PAT permissions: **Administration: write** (runner registration),
**Commit statuses: write**, **Contents: write**, **Workflows: write** (adopt
PRs). Classic PAT scopes: `repo workflow admin:org`. GHES:
`homeplate auth add work --host git.example.com`.

## After setup

- Health: `homeplate status --json` and `homeplate doctor`.
- Queue/logs: `homeplate logs`, `homeplate logs --follow`.
- Caps: `homeplate limit --cpus 4 --memory 8g` (hot-reloaded).
- Offline behavior is automatic: no network → jobs run via act locally →
  commit statuses posted on reconnect (labelled as locally run).

## Verifying success

1. `homeplate status` shows linked repos and the daemon running.
2. The adopt PRs merged → push a commit → the PR check runs locally
   (`homeplate logs` shows the job; the container is destroyed after).
3. `homeplate status` savings counter is non-zero and **net of the $0.002 fee**.

## Repo layout (for contributors)

- `homeplate/` — the Go CLI + daemon (module `github.com/homeplate-ci/homeplate`)
- `homeplate/docs/RESEARCH.md` — verified-facts dossier; check it before
  changing anything that touches GitHub APIs, pricing, or macOS power
- `site/` — Next.js static export, deployed via `wrangler deploy`
- `Formula/` — Homebrew formula

Build: `cd homeplate && go build ./... && go test ./...`

---
> Source: [sikmabull/homeplate](https://github.com/sikmabull/homeplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
