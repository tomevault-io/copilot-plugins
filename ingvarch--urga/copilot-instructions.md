## urga

> Guidance for Claude Code working in this repository.

# CLAUDE.md

Guidance for Claude Code working in this repository.

## What urga is

A terminal UI for HashiCorp Nomad. One binary, `urga`, that lists and acts on
cluster resources: jobs, allocations, tasks, task groups, logs, deployments,
namespaces, services, evaluations, nodes, variables and node pools.

The interface is English only. There is no translation layer and no plan for
one; write literals where they belong.

## Stack

- Go, module path `github.com/ingvarch/urga` (module path equals repo path, always)
- Bubble Tea v2 (`charm.land/bubbletea/v2`) for the program loop
- Lipgloss v2 (`charm.land/lipgloss/v2`) for styles
- `github.com/hashicorp/nomad/api` for the cluster

Charm moved the v2 modules to `charm.land/...`. The GitHub path resolves to v1
and does not build against this code.

## Architecture rules

These are settled, not open for redesign per change. They come from a fork of
hashicorp/damon where the opposite of each one caused a bug that took hours.

1. **One owner for the keyboard.** The root model decides who gets a key press.
   Overlays (prompt, filter, help, dialog) are fields of the model with an
   explicit order. No component installs a global key hook, and no key is
   handled in two places.
2. **The namespace travels with the request.** Every call into `internal/nomad`
   takes the namespace as an argument. Nothing reads a "current namespace" out
   of global state to build a request.
3. **Polling lives in `tea.Cmd`.** A poll returns a message. No goroutine writes
   to the model, no goroutine draws, no blocking send on a channel that the UI
   also reads. The event stream says *when* a screen is stale; *what* it now
   holds is read the usual way. Nothing is built out of event payloads, and a
   cluster that will not stream is polled the way it always was. `:events` is
   the one screen that shows the events themselves, as they came; no other
   screen is built out of them.
4. **Screen keys in the header, general keys in help.** The header lists only
   what the open resource can do. Navigation and global keys live in `?`.
5. **Render functions are pure.** A screen turns state plus a width into a
   string. That is what tests assert on.
6. **The table is ours** (`internal/ui/table.go`). Every screen is a list that
   needs three things at once: columns as wide as what is in them, a color per
   row for the state it is in, and a cursor row that reads. Nesting a cell
   color inside a highlight leaves the row unreadable, so the cursor row is
   painted end to end instead.

## Do not copy damon

hashicorp/damon is MPL-2.0 with an IBM copyright header in every file. Carry
over decisions and lessons, write the code here from scratch.

## Process

- TDD, no exceptions. A failing test that names the behaviour comes first, then
  the smallest code that passes it. This holds for "simple" functions too.
- `make check` (fmt, vet, lint, licenses, test, build) is green before every commit.
  It needs `golangci-lint` of the version the Makefile names, the one CI uses,
  and fails without it.
- Conventional commits: `feat(ui): ...`, `fix(nomad): ...`, `chore: ...`.
- No mention of other tools by name in code comments or commit messages.
- Comments are short and say why, never what the line already says. Existing
  comments are not deleted.
- Comments use plain words. Name what the code does ("closes the stream",
  "drops the answer", "shows the error") instead of a figure of speech ("lets
  go of it", "says so", "what it was left holding"). The terms of the code
  stay as they are: page, screen, stack, ask, poll, stream, watch.

---
> Source: [ingvarch/urga](https://github.com/ingvarch/urga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
