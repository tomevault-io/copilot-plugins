## ai-context-kit

> Read this before writing any code.

# Project Rules

Read this before writing any code.

## What this is

[One line. Same as the PRD one-liner.]

## Stack

- Language / framework: [e.g. Flutter 3.29]
- State management: [e.g. Riverpod 2.x]
- Database / storage: [e.g. Drift, local]
- Key packages: [only the ones with opinions attached]

## Commands

```bash
# run
[e.g. flutter run -d chrome]

# test
[e.g. flutter test]

# lint / format
[e.g. dart format . && flutter analyze]
```

Always run the lint command before saying a task is done.

## Conventions

- Folder structure: feature-first, not layer-first. See ARCHITECTURE.md.
- Naming: [e.g. snake_case files, PascalCase classes]
- [e.g. Widgets over 150 lines get split.]
- [e.g. No business logic inside widgets.]

## Rules

- Do not add a new dependency without asking first.
- Do not edit anything in `[/generated, /build, *.g.dart]`.
- Do not refactor code you weren't asked to touch.
- If a task is ambiguous, ask instead of guessing.
- Match the style of `[path/to/gold-standard-file]` — that file is the
  reference for how code in this project should look.

## Known gotchas

- [e.g. The [X] package breaks on hot reload — full restart needed.]
- [e.g. Don't use [old pattern]; we migrated off it in March.]

---

**Keep this file under ~50 lines.** Long context buries the rules that
matter. If a section grows, move it to its own file and link it here.

---
> Source: [rifqieh/ai-context-kit](https://github.com/rifqieh/ai-context-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
