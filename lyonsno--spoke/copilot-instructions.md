## spoke

> The canonical repo and product name is `spoke`.

# Gemini CLI Instructions

## Repo Identity

The canonical repo and product name is `spoke`.

When writing or updating docs, reviews, Epistaxis notes, PR text, release
notes, or other outward-facing references for this repo, use `spoke` rather
than `donttype` or `dictate`.

Treat the repo as renamed for documentation purposes and keep naming
consistent with `spoke`.

## Branching

**`main` is the integration branch.** `main-next` is retired — it was
force-pushed onto `main` on 2026-04-04 and the two are now identical.
For `spoke`, all new feature branches, fix branches, worktrees, and
temporary integration-carrier branches must be sliced from current
remote `origin/main`, not from `main-next`, `dev`, or any other branch,
unless the human explicitly asks for some historical or non-trunk
surface.

Before creating a worktree: `git fetch origin main` and branch from
`origin/main`.

Treat remote `origin/main` as the source of truth rather than any older
local trunk witness or smoke worktree. Do not call an older local
worktree the current tip just because it was the last place a smoke run
happened.

When the user asks to land work on the integration branch, that means
remote `origin/main`. Any temporary carrier branch used for verification
should be cut fresh from current `origin/main`, verified there,
remote-merged, and then cleaned up.

## Commits

Unless the user explicitly says otherwise, push commits after creating them.
If commit/push is the required next step, needing sandbox/escalation approval is not a reason to defer it or leave work local-only. Request the permission and continue.

## Epistaxis

When reading Epistaxis, if recorded state conflicts with what you observe in
the code, repo, or thread, flag the mismatch before proceeding. Multiple
sessions may write concurrently; merge your changes without overwriting
entries you did not author.

## Launch target registry policy

When the launch-target menu feature is in play:

- for `spoke`, a surface is not smoke-ready unless it is present in `~/.config/spoke/launch_targets.json` and launchable from the visible launcher UI on that machine
- treat `~/.config/spoke/launch_targets.json` as the curated source for menu-visible launch targets
- agents may add, remove, or retarget entries there when preparing or retiring local smoke surfaces
- there is no dedicated `smoke_branch` slot; additional prepared surfaces should appear as their own explicit registry entries
- prefer stable ids and short human labels; the entry should identify a purposeful surface, not a temporary hunk of local reasoning
- when `⌃⌥⌘K` and the menu should refer to the same smoke surface, keep `~/.config/spoke/smoke-target` and the registry entry with id `smoke` aligned
- do not silently assume the selected target also carries the launch-target affordance; if the target branch lacks the feature, say so when preparing the surface
- smoke-worthy surfaces must carry the launcher/menu commits that make the target selectable and legible in the menubar; registry prep alone is not enough
- record durable registry conventions or machine-local target changes in `spoke` Epistaxis when another session would need them to resume coherently

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lyonsno) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
