## carboncode

> Carbon Code is a new terminal coding agent project in `Yapie0/carboncode`. The goal is to build a Chinese-first, DeepSeek-powered personal developer CLI similar to Codex.

# AGENTS.md

## Project Context

Carbon Code is a new terminal coding agent project in `Yapie0/carboncode`. The goal is to build a Chinese-first, DeepSeek-powered personal developer CLI similar to Codex.

The repository is intentionally early. The main design reference is:

- `docs/superpowers/specs/2026-05-18-carboncode-cli-design.md`

## Current Status

- Local git repo is initialized on `main`.
- Remote is `git@github.com:Yapie0/carboncode.git`.
- Git identity for this repo:
  - `user.name=Yapie0`
  - `user.email=cloydeyzxqxsv@gmail.com`
- Initial design doc commit exists locally:
  - `403a731 Add Carbon Code CLI design`
- `origin/main` may not exist until the first push.
- Current implementation direction:
  - 导入 Reasonix 主体，然后做 Carbon Code 产品化。

## Product Direction

Build a personal developer CLI for v1.

The intended user flow:

```bash
npm install -g @carboncode/cli
cd path/to/project
carboncode
```

The CLI should read the repository, plan work, edit files, request permission before shell commands, run validation when approved, and show diffs and a concise outcome summary.

## Chosen Base

Use DeepSeek-Reasonix as the preferred implementation base because it is TypeScript/Node, DeepSeek-native, npm-friendly, and already contains useful agent primitives.

DeepSeek-TUI and Deep Code are references for UX and feature ideas, but they are not the preferred v1 base.

Do not start the agent engine from scratch unless the user explicitly changes this decision.

## Naming Decisions

- Product name: Carbon Code
- npm package: `@carboncode/cli`
- Main command: `carboncode`
- Short command: `ccode`
- Optional alias: `carbon`

Do not install `cc` by default. It commonly points to the system C compiler and may break native builds if shadowed.

## MVP Scope

Include:

- DeepSeek API configuration.
- DeepSeek V4 Pro / Flash model profiles, after verifying the current official model IDs.
- Interactive terminal session.
- Repository search and file reading.
- Safe file editing with visible diffs.
- Shell command approval.
- Test/build command execution when approved.
- Session resume.
- Project rules through `AGENTS.md` or `CARBON.md`.
- Chinese default prompts, errors, and command explanations.
- Basic token/cost visibility.
- MIT license compliance for upstream code.

## License And Attribution

If code is copied or derived from DeepSeek-Reasonix, preserve upstream MIT notices.

Expected files:

- `THIRD_PARTY_NOTICES.md`
- `LICENSES/DeepSeek-Reasonix-MIT.txt`

Attribution can be low-profile, but must remain discoverable in the repository and npm package. Do not remove upstream copyright notices from derived source files.

## Engineering Rules

- Prefer TypeScript/Node patterns for this project.
- Use existing upstream abstractions where they are sound.
- Keep early changes tightly scoped to branding, packaging, DeepSeek defaults, config, permission flow, and documentation.
- Do not add unrelated rewrites while importing the base.
- Keep local CLI code readable for local debugging and user trust.
- Before claiming a feature is done, run relevant verification commands and report what passed or failed.

## Next Recommended Steps

1. Continue Carbon Code productization of imported Reasonix surfaces.
2. Replace remaining user-visible Reasonix wording in prompts, docs, dashboard, and desktop surfaces.
3. Finish `AGENTS.md` / `CARBON.md` project-rule loading behavior.
4. Verify current official DeepSeek model IDs before public release copy.
5. Tag a release only after npm Trusted Publishing is configured for `@carboncode/cli`.

## Important Open Decisions

- Public open-source release vs. private/internal alpha first.
- Exact current DeepSeek model IDs.
- Whether `carbon` is installed by default or only created through opt-in alias setup.

---
> Source: [Yapie0/carboncode](https://github.com/Yapie0/carboncode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
