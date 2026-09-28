## claude-workspace-meta-assistant

> Working rules for any AI coding agent in this repository. Portable by design: Claude Code loads

# AGENTS.md — claude-workspace-meta-assistant

Working rules for any AI coding agent in this repository. Portable by design: Claude Code loads
this through `@AGENTS.md` at the top of `CLAUDE.md`; other agent tools read it directly.

## What this project is

A scaffolder and a distribution system. `create-project.py` emits Claude-ready projects. The
five Context Memory protocols in `protocols/` are the canonical copies and reach consuming
projects by git submodule, never by vendor-copy (ADR-003). The repository dogfoods its own
framework (ADR-005).

## Setup, build, test

No build step and no dependencies — Python 3.8+, standard library only.

    python3 self-test.py
    # or: make -f Makefile.claude-meta-assistant verify-self

The checks split into I* (repository integrity) and U* (scaffold a throwaway project into /tmp
and assert the emitted tree). All must be green before a change is proposed. This is a gate,
not a report.

Operator targets, all invoked with `make -f Makefile.claude-meta-assistant <target>` so they
never collide with a developer's own Makefile: `init`, `verify`, `verify-self`, `help`.

## Hard rules

These are not preferences. Each exists because breaking it cost something.

- **Never run remote git.** No push, merge, tag, or branch delete. The operator runs all of it
  manually. Agents make local commits only, and propose rather than execute.
- **Never add a `Co-Authored-By` trailer.** Commit messages are plain imperative. Verify with
  `git log -1 --pretty=%B` after every commit — this rule held for 30 releases and then broke
  silently, so check rather than assume.
- **Destructive local operations are blocked and must stay blocked.** A PreToolUse hook exits 2
  on `git reset --hard`, `git clean -f`, `git checkout -- <path>`, `git checkout .`, and
  `rm -rf`/`rm -fr` unless every target resolves under `/tmp`. Do not work around it. Surface
  the operation and let the operator run it.
- **Read before writing; audit after.** Read the current file in full before changing it. After
  any modification, grep both directions — new content present, old content gone — before
  taking another action.
- **Changing `create-project.py` or anything in `templates/` triggers scaffold validation.**
  Source reading does not capture `{{...}}` substitution behaviour; templates are shipped
  content, so what the operator receives post-scaffold is what matters. Scaffold a throwaway to
  `/tmp/<project-name>/` with `--dest` and `--owner` passed, inspect the real emitted files,
  confirm the change landed and substitution still resolves, then clean up. Never `make init`,
  never commit the throwaway, never touch the meta-repo tree during validation.

## Conventions

- **Cross-references use bare backticks, not markdown links.** Inside a guide or protocol, name
  a sibling as `Claude-Guide-NewProjectSetup.md`. The README is the framework's single link
  layer and self-test check I4 validates every markdown `.md` link in it. Backtick references
  read cleanly as prose, never decay into dangling links, and never require computing a
  relative path. The README owns navigation; the guides own content.
- **Namespace every `/tmp` write** so writes do not collide and cleanup is unambiguous.
  Development scratch and drafts go under `/tmp/claude-workspace-meta-assistant-temp-files/`;
  scaffolded throwaway projects go under `/tmp/<project-name>/`. Never write to bare `/tmp/`.
- **ADRs are decisions recorded before the edits they authorise**, not after. `docs/decisions/`.

## Where things live

| Path | What it is |
|---|---|
| `protocols/` | The five Context Memory protocols. Canonical. Never duplicated inside this repo. |
| `templates/` | Developer-completed skeletons emitted to the project root by a glob. |
| `claude-code-surface/` | Claude Code adapter, including the scaffold skeleton. |
| `claude-ai-surface/` | claude.ai adapter — custom-instruction templates and the bundle manifest. |
| `docs/decisions/` | ADRs. |
| `self-test.py` | The invariants. I* = repo integrity, U* = emitted-project integrity. |

## Things that have gone wrong before

- **Designing against a stale copy.** Uploaded snapshots and earlier extracts expire within a
  release. Read the current file every time; never design from a summary of it.
- **A standing rule degrading silently.** The `Co-Authored-By` prohibition held for 30 releases
  and then reappeared. Long-held rules are the ones that stop being checked.
- **Confusing an annotated tag's object SHA with its commit SHA.** They differ; comparing them
  raises a false alarm.
- **Inline `<svg>` in the README.** GitHub sanitises it — the diagram is in the file and
  invisible on the page. Use a fenced `mermaid` block.
- **"It parses" mistaken for "it renders."** A structural check is not a visual one. Say which
  one you performed.

## [OPERATOR] Anything else

[Add what only you know: review expectations, what you want proposed versus executed, files an
agent should never touch.]

---
> Source: [lloydbrian/claude-workspace-meta-assistant](https://github.com/lloydbrian/claude-workspace-meta-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
