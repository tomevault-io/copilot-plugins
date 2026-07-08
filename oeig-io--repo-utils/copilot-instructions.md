## repo-utils

> The purpose of this file is to orient any actor (human or AI) working in this

# AGENTS.md

The purpose of this file is to orient any actor (human or AI) working in this
workspace and route them to the repo that owns the detail they need. It is
important because the detail lives close to the code it describes — this file
stays a thin, DRY map, not a copy of those docs.

If someone asks your name, your nickname is Stewy.

## Workspace Shape

This directory is a **meta-workspace**: each top-level folder is its own
independent git repository, not a submodule. The root `AGENTS.md` is a symlink
to `repo-utils/AGENTS.md` (recreated by `repo-utils/agents-symlink.sh`), so edit
the real file under `repo-utils/`.

Repos are cloned and kept current by `repo-utils` — see its `README.md` for the
sync scripts and prerequisites.

**External repos are welcome.** Not every folder here belongs to `oeig-io`
(some are upstream sources we keep for reference). Clone any external repo you
need into the workspace: `repo-utils/git-clone-and-pull-all.sh` only *creates*
repos from the `oeig-io` org, but its `git pull` pass keeps **every** repo
present in the workspace up to date — external ones included. So add what you
need; sync will maintain it. See "Additional Helpful Repos" in
`repo-utils/README.md`.

## How We Write and Communicate

`wi-base/WORK_INSTRUCTIONS.md` is the governing standard for everything we write
and communicate — documentation, work instructions, skills, READMEs, commit
messages, and the prose in answers. Read it before authoring or editing any of
these, and treat it as authoritative when its guidance conflicts with habit.

Its core tenets apply everywhere in this workspace:

- **DRY** — remove redundancy; reference related information by name instead of
  repeating it (this AGENTS.md is itself an example).
- **Get to the point, then the big picture, then the details** — open with
  "The purpose of this ... is to ...", explain why it matters, then supply only
  the detail needed for success. Stay friendly.
- **Reference over repetition** — link to the owning doc; never copy it.
- **No real user or company data** in anything committed to a public repo — use
  generic placeholders (`ACME`, `example.com`, `[CLIENT_ID]`).
- **Lowercase-dashed filenames** and **YAML frontmatter** for tool/skill files.

When in doubt about format, naming, structure, or tone, defer to
`wi-base/WORK_INSTRUCTIONS.md`.

## Finding Your Way Around

Repos are self-locating by name prefix rather than a maintained list:

| Prefix / name | What it holds |
|---------------|---------------|
| `wi-*` | Work instructions and AI skills (see `wi-base/WORK_INSTRUCTIONS.md`) |
| `install-*` | Application payloads built as a 1:N factory (e.g. `install-idempiere`) |
| `host-*` | Dedicated 1:1 application payloads with couriered secrets |
| `idempiere-*` | iDempiere core, REST, docs, plugins, and go-live deploy |
| `container-management` | Create/manage NixOS Incus containers (`install-*` and `host-*`) |
| `incus`, `netbird` | Infrastructure: containers/VMs and the mesh network |
| `corporate/` | Internal company documentation |

**Each repo owns its own instructions.** Before working inside a repo, read its
`AGENTS.md`, `CLAUDE.md`, and/or `README.md` — that is where the authoritative,
current detail lives. Start with these:

- `container-management/` — creating and orchestrating containers
- `idempiere-golive-deploy/` — deploying, resetting, and verifying iDempiere
  containers (deploy/reset workflows, timeouts, audit logs, status checks,
  destructive-action rules)

## Cross-Cutting Principles

These apply everywhere; repo docs hold the specifics.

- **Confirm before destructive operations.** Verify with the user before
  deleting containers (`incus delete`), resetting databases (`reset.sh`), or
  rebuilding systems (`nixos-rebuild`). See the owning repo for the exact rules.
- **A timeout is not a failure.** Long-running scripts can outlast a tool
  timeout while still completing. Verify actual state (service health, expected
  records, audit logs) before re-running — never re-run a completed command
  blindly.
- **Return to the default branch when a change is done.** The fork/branch/PR
  flow is welcome, but a repo left parked on a feature branch quietly stops
  receiving updates to its default branch. When your change is pushed and the PR
  is open, switch back to the primary repo's default branch (`git switch main`)
  so you keep pulling updates. `repo-utils/git-status-all.sh` colors this: a repo
  off its home branch, or a home branch that has fallen behind, shows red — see
  its signals and per-repo overrides in `repo-utils/README.md`.
- **Search, not just the filesystem.** Zulip is a system of
  record — decisions and research often live only in a thread. When a `grep`
  comes up short, or the answer may have been discussed rather than committed,
  also search Zulip (see the `zulip-api` skill / `pi-zulip` search tools).

## Enabling/Disabling Extensions and Skills

Use `pi config` to toggle extensions and skills on or off for the current
session. This opens an interactive TUI where you check/uncheck resources with
`space`.

```bash
pi config
```

Common use: enable/disable MCP (`@0xkobold/pi-mcp`) per session. MCP is installed
but disabled by default — enable it with `pi config` when needed.

---
> Source: [oeig-io/repo-utils](https://github.com/oeig-io/repo-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
