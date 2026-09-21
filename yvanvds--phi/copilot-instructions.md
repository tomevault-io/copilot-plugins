## phi

> Workflow rules for Claude Code sessions on the Phi project. Read at session

# CLAUDE.md

Workflow rules for Claude Code sessions on the Phi project. Read at session
start. Pair with [docs/PROJECT_OVERVIEW.md](docs/PROJECT_OVERVIEW.md).

## How we work

1. **Issues first.** Every bug, feature, or enhancement is filed as a
   labelled GitHub issue *before* code is written. Branch from `develop`
   as `<issue-number>-<short-slug>` and PR back into `develop` through CI;
   `main` is protected and release-only, reached via a `develop → main`
   PR. The only exception is a trivial doc fix.
2. **Tests where sensible** (not dogmatically). Pure domain logic, the
   engine bridge, and end-to-end paths get tests first. Widget tests follow
   widget implementation. One-off scripts and animation tweaks don't need
   tests.
3. **One class per file.** Filename = `snake_case(ClassName)`. Folders are
   cheap; bundling classes is friction.
4. **Layered structure.** `core/` → `domain/` → `design/` → `engine/` →
   `shell/`+`surfaces/` → `app/main`. One-way dependencies. Update
   `docs/PROJECT_OVERVIEW.md` after structural changes.
5. **Design tokens come from `design system/colors_and_type.css`.** Dart
   tokens under `lib/design/tokens/` are derivations — keep them in sync.

## Label taxonomy

Created during Phase 0 and applied to every issue.

- **type:** `type:bug`, `type:feature`, `type:enhancement`, `type:task`,
  `type:docs`
- **layer:** `layer:scene`, `layer:dsp`, `layer:spatial`,
  `layer:time-domains`, `layer:patcher`, `layer:live-coding`, `layer:midi`,
  `layer:state-machine`, `layer:mix`, `layer:shell`, `layer:design`,
  `layer:engine-bridge`, `layer:infra`
- **priority:** `priority:p0-now`, `priority:p1-soon`, `priority:p2-later`,
  `priority:p3-maybe`
- **status:** `status:triaged`, `status:blocked`, `status:in-progress`

Pick at least one `type:` and one `layer:`. Priority and status are
maintainer-applied.

## Running locally

Requires Windows, Flutter ≥ 3.44, and `libyse.dll` discoverable via
`YSE_DLL_PATH` or sitting next to the executable. See [README.md](README.md)
for the build recipe.

```powershell
flutter pub get
flutter analyze
flutter test
flutter run -d windows
```

## Boundaries

- **Don't** push to `main` without confirmation.
- **Don't** skip hooks (`--no-verify`) or bypass signing.
- **Don't** import `package:yse/...` outside `lib/engine/bridge/` — the
  engine façade is the only place the FFI surface is touched.
- **Don't** put Flutter imports into `lib/domain/` — domain stays pure Dart.
- **Do** file an issue on `yvanvds/dart-yse` when the bridge needs new
  capabilities, rather than working around the wrapper.

## Memory

There is a Claude Code memory store for this project under
`~/.claude/projects/d--phi/memory/`. It captures durable preferences and
project facts that survive between sessions.

---
> Source: [yvanvds/phi](https://github.com/yvanvds/phi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
