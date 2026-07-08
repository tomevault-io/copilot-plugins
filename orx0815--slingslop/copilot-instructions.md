## slingslop

> This file provides context for the GitHub Copilot **cloud coding agent** working on this repository.

# GitHub Copilot Agent Instructions — Slingslop

This file provides context for the GitHub Copilot **cloud coding agent** working on this repository.

## Repo Overview

See [AGENTS.md](../AGENTS.md) at the repo root for a full description of the project layout, tech stack, and key conventions.

## Agent Smith — New Sling App Skill

When an issue is assigned to Copilot and its title or body references **"Agent Smith"** or **"new Sling app"**, the agent MUST follow the full workflow defined in:

```
docs/agent-skills/create-Sling-app-with-Agent_Smith.md
```

Read that file **before doing anything else**. It is the authoritative specification for scaffolding a new Sling application.

### Key points for cloud (non-interactive) execution

Because the cloud agent cannot hold a back-and-forth conversation, the issue body MUST contain all the inputs that Agent Smith would normally collect interactively. The issue template at `.github/ISSUE_TEMPLATE/agent-smith-new-app.yml` defines those fields.

The agent should:

1. Read `docs/agent-skills/create-Sling-app-with-Agent_Smith.md` in full.
2. Map every issue-template field to the corresponding Agent Smith variable (see table below).
3. Skip the Conversation Phase (§1) — the issue body replaces it.
4. Execute all 13 tasks from §2.0 in order, tracking them as a todo list.
5. Open a pull request when the build passes, using the PR description format from the skill.

### Field mapping: issue → Agent Smith variables

| Issue field | Agent Smith variable |
|---|---|
| Project name | `PROJECT_NAME` (human-readable) |
| Maven groupId | `GROUP_ID` |
| Maven artifactId prefix | `PROJECT_NAME` (lower-case-hyphenated) |
| Java package | `JAVA_PACKAGE` |
| `sling:resourceType` prefix | `RT_PREFIX` |
| Sample content JCR path | `CONTENT_ROOT` |
| Apps path | `APPS_ROOT` |
| Project kind | Design intent |
| What it does | Functional purpose / dummy text tone |
| Tone & feel | Copy tone |
| Favourite colours | CSS OKLCH base hues |
| Navigation style | Layout structure |
| Zen-editable | Whether to include HTMX + Tiptap editing stack |
| Visual mood | CSS direction |
| Inspiration URLs | Design references (optional) |

## Build & Validation

After scaffolding, validate with:

```bash
mvn clean install -DskipITs
```

Fix any errors before opening the PR. The PR should not be marked ready for review until the build is green.

---
> Source: [orx0815/slingslop](https://github.com/orx0815/slingslop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
