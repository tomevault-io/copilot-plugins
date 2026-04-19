## brynhild-deno-plugin

> Commit plans - read BEFORE creating any commit plan


# Commit Plans

This project uses YAML commit plans instead of direct git commits.

## Key Rules

1. **NEVER run `git commit` directly** - Always use commit plans
2. **NEVER execute commits autonomously** - Only create/preview plans
3. **workflow/ is gitignored** - Planning docs don't get committed to the main repo
4. **Each file can only appear in ONE commit** - The tool stages whole files, not hunks

## Commit Plan Locations

There are **two separate commit plan directories**:

| Directory | For commits to... | Tracked in... |
|-----------|-------------------|---------------|
| `workflow/commit-plans/` | Main repo (public GitHub) | Not tracked (gitignored) |
| `workflow/commit-plans-workflow/` | Workflow mini-repo | Workflow repo only |

## Creating a Commit Plan

Location: `workflow/commit-plans/YYYY-MM-DD-description.yaml` (main repo)
Location: `workflow/commit-plans-workflow/YYYY-MM-DD-description.yaml` (workflow repo)

```yaml
repo: /optional/path/to/repo    # Optional: override repo path
tag: v1.0.0                      # Optional: create git tag after commits
tag_message: "Release v1.0.0"    # Optional: makes tag annotated

commits:
  - message: |
      type: short description

      Longer explanation if needed.
    files:
      - path/to/file1.py
      - path/to/file2.py
    deleted:                      # Optional: files to remove
      - path/to/removed.py
```

If a file needs changes across multiple logical commits, combine those changes into one commit or structure your work differently.

## Commit Types

`feat:` `fix:` `refactor:` `docs:` `test:` `config:` `security:` `improve:` `chore:`

## Correct Workflow

1. **Create the plan** - Write `workflow/commit-plans/YYYY-MM-DD-description.yaml`
2. **Preview** - `./scripts/commit-helper.py plan.yaml` (validates files exist)
3. **Execute** - `./scripts/commit-helper.py plan.yaml --execute`

## Using the Script

```bash
./scripts/commit-helper.py plan.yaml             # Preview (safe, always OK)
./scripts/commit-helper.py plan.yaml --execute   # Execute commits
```

## What You Can Do Autonomously

| Action                   | Allowed?                          |
|--------------------------|-----------------------------------|
| Create commit plan YAML  | ✅ Yes                             |
| Run preview (no execute) | ✅ Yes                             |
| Run execute              | ❌ Only with explicit user request |
| Add tag to plan          | ❌ Only with explicit user request |

**Tags are releases** — never add `tag:` to a commit plan unless the user explicitly requests a tag/release.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mandersogit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
