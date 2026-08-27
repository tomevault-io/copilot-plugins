## cross-platform-ai-harness

> This file is symlinked into every AI tool's config, so they all start with the same

# Agent Rules

This file is symlinked into every AI tool's config, so they all start with the same
rules. It's the top of the harness — the main **guide**.

Keep it short. Add a rule only after a real mistake makes you want it (the ratchet).

## Rules

- **Pull first.** Run `git pull` before you change anything.
- **No secrets in the repo.** Keys and tokens live in a secrets manager. Only
  non-secret `@key` values (IPs, ports) go in `CONSTANTS.md`.
- **Keep your copy private.** Once you fill in your leaves, this repo holds real
  inventory — hosts, addresses, project notes. That's personal data. Use a private
  remote or stay local, and ask before any push. (The public *starter* can be public;
  your working copy can't.)
- **No AI attribution** in commits or code. No "co-author" trailers, no `@author`
  line naming a model. Every change is the human author's. The `commit-msg` hook
  blocks it.
- **Config lives here first.** If a tool has config, keep a copy in this repo with
  secrets as placeholders. Edit the copy here, then apply it to the real machine.
- **Ask before anything hard to undo or facing outward** — deploys, deletes, sending.

## Bootstrap

- User says **"start"** / **"bootstrap"** → follow `HARNESS.md` → Bootstrap Protocol,
  in order.
- User says **"onboard me"** / **"set me up"**, or this is a **fresh clone with no
  leaf for this machine** → read `ONBOARDING.md` and build their leaves from the
  templates.

## Routing

Find the task below, read that leaf's `AGENTS.md`, then act. Full map in `INDEX.md`.

| Keyword | Read |
|---|---|
| machine, inventory, install status | `machines/<machine>/AGENTS.md` |
| a specific AI tool / CLI | `tools/<tool>/AGENTS.md` |
| how-to (docker, deploy, db, …) | `skills/<skill>/AGENTS.md` |
| a project | `projects/<project>/AGENTS.md` |
| ssh, alias, tunnel, reach a host | `skills/ssh-tunnel/AGENTS.md` · `machines/ssh-aliases.conf` |
| a repeating non-secret value (`@key`) | `CONSTANTS.md` |
| what's still open across machines | `bash scripts/dashboard.sh` |

Rule of thumb: find the canonical file before you decide. If a change touches several
areas, link them to each other.

---
> Source: [muzafferkadir/cross-platform-ai-harness](https://github.com/muzafferkadir/cross-platform-ai-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
