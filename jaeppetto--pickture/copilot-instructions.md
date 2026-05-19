## pickture

> **iOS native gallery cleaning app** — helps users organize, review, and clean up their photo library.

# Pickture

**iOS native gallery cleaning app** — helps users organize, review, and clean up their photo library.

## Tech Stack

- **Platform:** iOS (native)
- **Language:** Swift
- **UI Framework:** SwiftUI
- **Architecture:** Clean Architecture (Domain / Data / Presentation) + MVVM
- **Concurrency:** Swift Concurrency (async/await, actors)
- **Dependency Management:** Swift Package Manager (SPM)
- **Minimum Target:** iOS 17.0

## Agent Team

This project uses 7 specialized agents:

| Agent | File | Role |
|-------|------|------|
| Architect | `.claude/agents/architect.md` | System design & architecture decisions |
| Infra | `.claude/agents/infra.md` | Build, CI/CD, environment setup |
| UI/UX | `.claude/agents/uiux.md` | Design system, components, UX |
| Developer | `.claude/agents/developer.md` | Core feature implementation |
| Tester | `.claude/agents/tester.md` | Test strategy & execution |
| Rules | `.claude/agents/rules.md` | Coding standards & linting |
| Docs | `.claude/agents/docs.md` | Documentation & knowledge management |

## Project Structure

```
Pickture/
  App/                # App entry point, App delegate, scene configuration
  Core/               # Shared utilities, extensions, constants
    Theme/            # Design tokens, colors, typography
    Extensions/       # Swift extensions
    Constants/        # App-wide constants
  Domain/             # Entities, repository protocols, use cases
    Entities/
    Repositories/     # Protocol definitions
    UseCases/
  Data/               # Repository implementations, data sources, models
    Repositories/
    DataSources/
    Models/
    Mappers/
  Presentation/       # SwiftUI views, ViewModels
    Screens/
    Components/       # Reusable SwiftUI views
    ViewModels/
  Resources/          # Assets, Localizable strings, Info.plist
    Assets.xcassets/
    Localizable/
PicktureTests/        # Unit & integration tests
PicktureUITests/      # UI tests
docs/                 # ADRs, guides
scripts/              # Build & automation scripts
```

## Core Design Principles

> These principles are **non-negotiable** and must guide every decision across all agents.

### 1. Distinctive, High-Quality UI/UX
- The app's visual identity must be **original and polished** — avoid generic, template-like aesthetics.
- Pursue a unique design language with carefully crafted micro-interactions, transitions, and visual hierarchy.
- Every screen should feel intentional. Typography, spacing, color, and motion must work together cohesively.
- Benchmark against best-in-class gallery and utility apps. The goal is an experience users want to show off.

### 2. Performance at Scale (Tens of Thousands of Photos)
- Users may have **30,000+ photos** in their library. Every feature must be designed with this scale in mind.
- **Mandatory patterns:**
  - Lazy loading and pagination — never fetch all assets at once.
  - `PHCachingImageManager` for efficient thumbnail pre-fetching and cache management.
  - Incremental `PHFetchResultChangeDetails` for reactive, diff-based updates instead of full reloads.
  - Background processing with `TaskGroup` / actors for heavy operations (duplicate detection, analysis).
  - Memory budget awareness — monitor and cap image cache sizes, release off-screen resources.
- **Forbidden patterns:**
  - Loading all `PHAsset` metadata into memory at once.
  - Full-resolution image loading for thumbnails or previews.
  - Blocking the main thread with photo library operations.
  - Unbounded in-memory collections of image data.
- When in doubt, profile first. Use Instruments (Allocations, Time Profiler, Core Animation) to validate.

## Cross-Cutting Conventions

- **Language:** Code and agent instructions in English. User-facing communication in Korean.
- **Naming:** `camelCase` for variables/functions/properties, `PascalCase` for types/protocols/enums, `SCREAMING_SNAKE_CASE` for global constants.
- **File naming:** `PascalCase.swift` — one primary type per file, filename matches type name.
- **Imports:** System frameworks first (`Foundation`, `SwiftUI`, `PhotosUI`), then SPM packages, then local modules. Each group separated by a blank line.
- **Architecture layers:** Domain has zero dependencies on Data or Presentation. Data depends only on Domain. Presentation depends on Domain (via ViewModels).
- **No circular dependencies** between features or layers.
- **Swift formatting:** Follow Swift standard style. Use SwiftFormat/SwiftLint for enforcement.
- **Analysis:** SwiftLint strict mode. Zero warnings policy.
- **Concurrency:** Prefer `async/await` over closures/Combine for async operations. Use `@MainActor` for UI-bound code.
- **Error handling:** Use Swift's `Result` type or typed throws. No force unwrapping (`!`) except in tests.
- **Access control:** Explicit access modifiers on all declarations. Default to most restrictive (`private`, `internal`). Use `public` only for module API boundaries.
- **Dependency Injection:** Protocol-based DI. ViewModels receive dependencies through initializer injection.

## Agent Workflow Rules

All agents **must** follow these rules during every session:

### 1. Skill 적극 활용
- Use `/find-skills` to discover relevant local and global skills before starting work.
- When a skill matches the task (e.g., `ui-ux-pro-max` for design, `git-commit` for commits), **always invoke it** instead of doing the work manually.
- Periodically check for new skills that may have been added.

### 2. Agent Team 적극 활용
- Delegate tasks to the appropriate specialized agent. Do not do another agent's job.
- Use the `Task` tool to spawn sub-agents for parallel, independent work.
- When a task crosses ownership boundaries, coordinate by spawning the relevant agent rather than doing it yourself.

### 3. Efficient Context Management
- Use sub-agents (`Task` tool) for research, exploration, and isolated tasks to protect the main context window.
- Avoid dumping large file contents into the main conversation — read only what's needed, use targeted searches.
- Prefer `Glob` and `Grep` for quick lookups; use `Task(subagent_type=Explore)` for deep exploration.
- Keep responses concise. Do not repeat information already established in the conversation.

### 4. Session Handoff Protocol (Context 15% Rule)
When the session context reaches approximately **15% remaining**:
1. **Stop** current work at a safe checkpoint.
2. **Create a handoff summary** in `/Users/jaeppetto/.claude/projects/-Users-jaeppetto-pickture/memory/HANDOFF.md` containing:
   - **Completed:** What was accomplished in this session (bullet list).
   - **In Progress:** Any partially completed work and its current state.
   - **Next Steps:** Specific tasks/phases to continue in the next session.
   - **Key Decisions:** Any important decisions made that affect future work.
   - **Blockers:** Any unresolved issues or questions.
3. **Notify the user** in Korean that context is running low and the handoff file has been written.
4. The next session should **read `HANDOFF.md` first** and continue from where the previous session left off.

---
> Source: [Jaeppetto/pickture](https://github.com/Jaeppetto/pickture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
