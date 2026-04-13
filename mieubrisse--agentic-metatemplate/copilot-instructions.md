## agentic-metatemplate

> <!-- BEGIN METATEMPLATE — replace everything between these markers with your template's CLAUDE.md -->

<!-- BEGIN METATEMPLATE — replace everything between these markers with your template's CLAUDE.md -->
agentic-metatemplate
====================

A GitHub template repo for creating agentic template repos. This repo contains no application code — it is infrastructure files only (Claude Code config, git hooks, setup checklist, skills).

When working here, you are editing the metatemplate itself. Changes affect every future template created from it.

Repository Structure
--------------------

```
CLAUDE.md                           This file
README.md                           Public-facing docs (explains the pattern, how to use)
REPO_SETUP_CHECKLIST.md             Base setup steps for repos created from child templates
TEMPLATE_SETUP_CHECKLIST.md         Setup steps for template authors (you)
.source-templates.yml               Template lineage tracking (self-referential here)
.claude/
  settings.json                     Claude Code permissions and hooks
  skills/
    setup-repo/SKILL.md             Agent-driven project setup (reads the checklist)
    setup-template/SKILL.md         Agent-driven template setup (reads the template checklist)
    sync-templates/SKILL.md         Pull upstream template improvements
.githooks/                          Git hook shells (Beads populates via bd init)
  pre-commit                        Quality gates placeholder
  prepare-commit-msg                Beads populates on init
  post-checkout                     Beads populates on init
  post-merge                        Beads populates on init
  pre-push                          Beads populates on init
```

How the Pieces Fit Together
---------------------------

Template authors create a repo from this metatemplate, then run `/setup-template` to configure it. This:

1. Replaces metatemplate-specific content in CLAUDE.md with their template's context
2. Replaces the README with one describing their template
3. Updates `.source-templates.yml` to point to their own repo
4. Removes the template setup infrastructure (this checklist and skill)

After that, the template author adds technology-specific code, extends `REPO_SETUP_CHECKLIST.md`, and customizes `settings.json`.

End-users of child templates run `/setup-repo`, which reads `REPO_SETUP_CHECKLIST.md` and walks through each item.

The `/sync-templates` skill lets downstream repos pull improvements from their upstream template by diffing `.source-templates.yml`'s `lastSyncedCommit` against HEAD.

Editing Guidelines
------------------

Changes to these files propagate to every child template via `/sync-templates`:

- **REPO_SETUP_CHECKLIST.md** — base items should be universal (placeholder replacement, beads init, license, hooks). Do not add technology-specific items.
- **setup-repo skill** — keep it generic. It reads the checklist; the checklist is what varies.
- **sync-templates skill** — the triage/apply/verify workflow. Changes here affect all downstream sync operations.
- **settings.json** — base permissions only. Template authors add their own tool permissions.
- **.githooks/** — minimal shells. Beads injects its blocks; template authors add quality checks to pre-commit.

Required Workflows
------------------

Invoke `/technical-writer` when modifying the README — it is the public face of this project.
<!-- END METATEMPLATE -->

Issue Tracking
--------------

This project uses **bd** (Beads) for issue tracking in shared Dolt server mode.

```bash
bd prime          # Load workflow context
bd ready          # Find unblocked work
bd create "Title" # Create an issue
bd close <id>     # Complete an issue
```

Run `bd prime` for full workflow details. Install with `brew install steveyegge/beads/beads`.

Syncing with Upstream Templates
-------------------------------

This project tracks its upstream template(s) via `.source-templates.yml`. Invoke `/sync-templates` to pull in improvements from the template repos this project was generated from. The skill diffs upstream changes, triages them for relevance, and applies the ones that fit — without overwriting your customizations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mieubrisse)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mieubrisse)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
