## phpstorm-prereview

> PhpStorm plugin for local, PR-style code review: inline comments in the editor, written by the user or imported from AI agent runs (`codex` and `claude` CLIs reviewing the current branch against the default branch).

# Prereview — development notes

PhpStorm plugin for local, PR-style code review: inline comments in the editor, written by the user or imported from AI agent runs (`codex` and `claude` CLIs reviewing the current branch against the default branch).

## Build & run

- `./gradlew build` — compile + tests
- `./gradlew runIde` — sandboxed PhpStorm with the plugin
- The Gradle wrapper defaults `GRADLE_USER_HOME` to `./.gradle-user-home` (unless already set) to isolate plugin-SDK caches.
- Releases are driven by GitHub releases: the tag `vX.Y.Z` becomes the plugin version; `pluginVersion` in `gradle.properties` is only a local fallback.

## Architecture decisions

- The core UI primitive is a **component inlay card** (a real Swing panel embedded in the editor via `ComponentInlayAdapter`), not a gutter-icon-plus-tool-window combo. The quality of the inline UX is the whole point of the plugin; prefer improving the card over adding secondary surfaces.
- Comment display has two states: compact (range highlight + gutter marker) and expanded (the inlay card). The tool window is a fallback/overview, not the primary surface.
- **Anchoring** is deliberately best-effort: file URL + offsets + line, plus a dedicated anchor line (card renders under the end of the selection) and a small context snippet. Anchors track document edits (`ReviewCommentDocumentSyncService`) and file renames/moves (`ReviewCommentsVfsSyncService`); imprecision after heavy refactors is accepted, don't over-engineer recovery.
- Persistence is project-local via `PersistentStateComponent` (`ReviewCommentsService` is the source of truth). No remote sync by design.
- Agent runs (`AgentReviewService`) execute the `codex` and `claude` CLIs in parallel through `StructuredCliReviewRunner` subclasses, with a JSON output schema. Each agent's comments replace that agent's previous comments only when its run succeeds. CLI binaries are auto-discovered in `PATH` and common install locations — no user configuration.

## Non-goals

Remote sync, multi-user, comment threading/resolution, GitHub-style full diff review UI, other JetBrains IDEs, compatibility with older PhpStorm versions, MCP/realtime agent protocols.

## Known fragile areas

Test around these when touching the inlay/editor code:

- Keyboard focus inside the embedded card, and macOS system shortcuts
- Component lifecycle when an editor is recreated (tab reopen, split editors)
- Rendering with scroll, soft wraps, and folding
- Anchor drift during document edits and undo/redo

---
> Source: [mnapoli/phpstorm-prereview](https://github.com/mnapoli/phpstorm-prereview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
