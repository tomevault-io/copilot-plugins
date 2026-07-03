## apple-dev-skills

> This file is the Apple Dev Skills compatibility-repo override. Follow the root `socket` guidance for general git, docs, release, branch, and maintainer workflow rules when changing the canonical Socket-hosted payload.

# AGENTS.md

This file is the Apple Dev Skills compatibility-repo override. Follow the root `socket` guidance for general git, docs, release, branch, and maintainer workflow rules when changing the canonical Socket-hosted payload.

## Scope

- This repository is a compatibility marketplace and README pointer for users who installed `gaelic-ghost/apple-dev-skills` directly.
- The canonical authored Apple Dev Skills payload now lives in `gaelic-ghost/socket` under `plugins/apple-dev-skills`.
- Treat `productivity-skills` as the default baseline maintainer layer for general repo docs and maintenance work; this repo is the narrower specialist layer when Apple-specific behavior should change the workflow.
- Preserve standalone-install guidance for public users who install only `apple-dev-skills`, while leading new users to the Socket marketplace.
- Keep `.agents/plugins/marketplace.json` as the compatibility surface that redirects to the Socket subdirectory.
- Do not add plugin payload, `.codex-plugin`, or `.agents/skills` surfaces back to this repository for new feature work; make payload changes in Socket.

## Apple Rules

- For Swift, Apple framework, Apple platform, SwiftUI, SwiftData, Observation, AppKit, UIKit, Foundation-on-Apple, or Xcode-related guidance, require reading the relevant Apple documentation before proposing implementation changes.
- State the documented Apple behavior being relied on before design or code changes are proposed.
- If Apple docs and current code disagree, stop and surface that conflict.
- If no relevant Apple documentation can be found, say that explicitly before proceeding.
- Keep `explore-apple-swift-docs` as the canonical docs-routing surface instead of re-embedding broad docs-source selection logic into execution skills.
- Prefer framework-provided behavior over custom wrappers, coordinators, glue, or renaming layers unless a concrete constraint requires them.
- For Xcode app repos, tracked `.pbxproj` changes are critical project state when produced by Xcode, XcodeGen, or another project-aware workflow.
- Treat `Package.resolved` and similar package-manager outputs as generated files. Do not tell maintainers or agents to hand-edit them.

## Install Guidance

- The public README should lead with `codex plugin marketplace add gaelic-ghost/socket` and `codex plugin marketplace upgrade socket` because Socket is the preferred catalog.
- Also document `codex plugin marketplace add gaelic-ghost/apple-dev-skills` and `codex plugin marketplace upgrade apple-dev-skills` for compatibility installs.
- Keep explicit refs scoped to pinned reproducible installs and manual local clone marketplace instructions scoped to development, unpublished testing, or fallback cases.
- Keep README license guidance aligned with Socket's current PolyForm Noncommercial and separate commercial licensing terms.

## Validation

```bash
bash .github/scripts/validate_repo_docs.sh
```

Use the docs validator when README, AGENTS, ROADMAP, active skill inventory, or maintainer docs change. Use pytest when skill behavior, scripts, validation helpers, or tested contracts change.
Use the docs validator when README, AGENTS, marketplace metadata, or licensing docs change. This repository no longer carries skill behavior or pytest-backed payload tests.

---
> Source: [gaelic-ghost/apple-dev-skills](https://github.com/gaelic-ghost/apple-dev-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
