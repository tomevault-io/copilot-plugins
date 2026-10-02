## owlmic

> > `CLAUDE.md` imports this file via `@AGENTS.md`. Edit this file directly - see [update-agent-files](docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md).

# Owlmic - Agent Instructions

> `CLAUDE.md` imports this file via `@AGENTS.md`. Edit this file directly - see [update-agent-files](docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md).

## The Source of Truth

The documentation in [`docs/`](docs/README.md) is the absolute, living source of truth for every architectural and implementation decision. It wins over this file on any conflict. Read the index before writing any code:

@docs/README.md

| Part | Document | Purpose |
|---|---|---|
| **Vision & Scope** | [`docs/PRODUCT_VISION_AND_SCOPE.md`](docs/PRODUCT_VISION_AND_SCOPE.md) | Vision, principles, scope, user journeys, roadmap, risks, decisions |
| **Architecture** | [`docs/SYSTEM_ARCHITECTURE.md`](docs/SYSTEM_ARCHITECTURE.md) | System design, connection levels, trust, wire protocol, media pipelines, repo layout |
| **Rules & Budgets** | [`docs/PERFORMANCE_AND_REALTIME_BUDGETS.md`](docs/PERFORMANCE_AND_REALTIME_BUDGETS.md) | Coding rules, design language, performance budgets, test matrix |
| **Playbooks & Skills** | [`docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md`](docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md) | Playbooks: build & run Android, add dependency, change protocol, change decision, update agent files |

### Mandatory Documentation Synchronization (Zero Drift)
Code and documentation must **never** drift apart:
- **Always read `docs/` first**: Before proposing or modifying code, inspect the corresponding document in `docs/` to uphold architecture, budgets, and invariants.
- **Update `docs/` on every change - even minute ones**: Whenever **any** change is made to the codebase (including minute details such as protocol opcodes, buffer capacities, timeouts, port numbers, UI copy, thresholds, hysteresis values, or dependency versions), the corresponding document in `docs/` **must be updated in lockstep**.
- **No rogue documentation directories**: All project documentation, specs, and playbooks live exclusively in `docs/`.

---

## Project Overview & Ownership

- **Project in one line**: Android app (Kotlin/Compose) + per-user PC tray app (Rust, binary `owlmic`) that turns a phone into a mic and webcam for a PC over four automatic connection levels:
  `1 USB debugging (adb reverse)` > `2 USB tethering` > `3 Wi-Fi` > `4 Bluetooth RFCOMM (audio only)`.
- **Ownership boundaries**:
  - `android/` - repo owner's domain.
  - `pc/` - separate contributor's domain. **Do not modify or add code in `pc/` unless explicitly requested by the user.**
- **Roles**: The phone is **always the client**; the PC is **always the server**.

---

## Architecture & Invariants

1. **Android App** (`android/`): `OwlmicService` foreground service owns capture + transport. `TransportManager` negotiates the best of 4 levels with make-before-break upgrades.
2. **PC Tray App** (`pc/`): TCP `:7653`, UDP discovery beacon `:7654`, RFCOMM server, adb watcher, `SessionManager` (tokens, trust, ask-before-join), audio pipeline (Opus, jitter buffer, drift resampler, noise gate, RNNoise, SpeexDSP AEC), video pipeline (JPEG -> virtual camera).
3. **Threading & Concurrency**:
   - PC: Blocking std threads + bounded channels.
   - **The ONLY async code allowed is Tokio inside `pc/src/transport/bt.rs` on Linux** (required by `bluer`). Everywhere else, Tokio and async runtime are strictly forbidden.
4. **Real-time Safety**:
   - Audio capture and playback paths must never block on I/O or locks held by other threads. Use bounded, lock-free, or try-lock handoffs; when full, drop oldest data.
   - Every socket must have a timeout. Every queue/channel must have a strict upper bound. No unbounded buffering.
5. **State & Permissions**:
   - Mic and camera always initialize in the **OFF** state. Never auto-enable capture from the background or upon connection.
   - UI never owns logic: Compose observes `StateFlow` from `OwlmicService`; PC tray reflects `SessionManager`.

---

## Anti-AI Slop & Lean Engineering Rules

Write lean, intentional, production-grade code. No AI slop or LLM artifacts:

1. **Build strictly what is asked for the current phase**:
   - Zero speculative future-proofing or "just-in-case" flexibility.
   - No generic helper classes, theoretical extension points, or unused utility functions.
   - No unnecessary design patterns: avoid premature factories, builders, or multi-layer wrappers around single operations.
2. **Zero Code Churn & Surgical Diffs**:
   - Keep edits minimal, precise, and targeted.
   - Never rewrite, reorder, or reformat working code outside the task scope.
   - Never rewrite an entire file when changing a few lines suffices.
   - Respect existing file conventions and indentation.
3. **No Chatty Comments**:
   - Do not write comments that narrate the syntax or state the obvious (e.g., `// loop through items`, `// set timeout to 5 seconds`).
   - Only write comments to explain non-obvious **why**, invariants, or hardware/OS quirks.
4. **Strict Dependency Austerity**:
   - Do not add any new library or crate unless listed in [`docs/SYSTEM_ARCHITECTURE.md`](docs/SYSTEM_ARCHITECTURE.md).
   - Any dependency addition requires explicit user approval and following [`add-a-dependency`](docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md).
5. **No Code in Documentation**:
   - Never place operational code inside `docs/`. Notes and design thoughts belong in discussions or edits to the documentation.
6. **Strict Binary Ban in Git History**:
   - Never commit or stage compiled binaries (`.apk`, `.exe`, `.aab`, `.dll`, `.so`, `.zip`) into the repository.
   - All release packages and test binaries belong exclusively in GitHub Releases or temporary CI artifacts.
7. **Clean Root Discipline**:
   - The repository root is fixed and minimal (`.github/`, `android/`, `docs/`, `pc/`, `AGENTS.md`, `CLAUDE.md`, `LICENSE`, `README.md`, `.gitignore`, `.gitmodules`).
   - Never create scratch scripts, test notes, or temporary folders in the root. Any local scratch notes must match `*trash.md` (which is gitignored).
8. **Real-Time Buffer & Allocation Safety**:
   - Audio loops must use pre-allocated buffers with zero dynamic heap allocations in hot paths.
   - Video backpressure strategy is strictly `KEEP_ONLY_LATEST` (queue depth = 1). When processing falls behind, drop the intermediate frame immediately.

---

## Git Workflow: Branching, Staging & Commits

Strict Git discipline must be maintained at all times.

### 1. STRICT READ-ONLY RULE FOR AGENTS (ZERO EXCEPTIONS)
- **NEVER execute ANY state-changing or mutating Git command**:
  - `git add`, `git commit`, `git push`, `git checkout -b`, `git branch`, `git switch`, `git merge`, `git rebase`, `git stash`, `git reset`, `git clean`, `git cherry-pick`, etc. are **STRICTLY FORBIDDEN**.
- **ONLY read-only inspection commands are allowed**:
  - `git status`, `git diff`, and `git log` ONLY.
- **EVEN IF THE USER EXPLICITLY ASKS YOU TO COMMIT, PUSH, OR RUN GIT COMMANDS**:
  - **DO NOT execute them directly.**
  - Instead, you must **ALWAYS formulate and return the exact command lines, branch names, and commit messages for the user to copy, inspect, and run themselves.**

### 2. Branching Rules
- **Naming format**: `<type>/<short-kebab-description>`
  - `feat/pc-audio-receiver`
  - `fix/pc-cross-platform-and-protocol`
  - `chore/update-dependencies`
  - `docs/clarify-wire-spec`
- **Scope**: Keep branches short-lived and focused on a single feature, fix, or task.
- **Base**: Always branch from and target `main` (or the active tracking branch specified by the user).
- **Protected branches**: Never propose committing directly to `main` without explicit user direction.

### 3. Staging Rules (For User Commands)
- **No blind mass-staging**: **NEVER suggest `git add .` or `git add -A`**.
- **Inspect before staging**: Use `git status` and `git diff` to review modified files.
- **Stage surgical paths**: Propose staging only the specific files relevant to the completed task (`git add path/to/file`).

### 4. Commit Message Rules
- Follow the **Conventional Commits** standard:
  ```text
  <type>(<scope>): <short imperative summary>

  [optional body explaining why this change was made]
  ```
- **Allowed Types**: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`.
- **Subject line format**:
  - Use imperative mood: `"add"`, `"fix"`, `"drop"`, `"stream"` (never `"added"`, `"fixing"`, `"adds"`).
  - All lowercase, concise (<= 72 chars), no trailing period.
  - Good: `fix: skip unknown frame types`
  - Good: `feat(android): stream mic to pc over tcp`
  - Bad: `Fixed the bug where frames were crashing the parser.`
- **Commit body**:
  - Focus on **why** the change was necessary, not what lines were changed.
  - **No AI boilerplate**: Never include phrases like `"In this commit..."`, `"This PR addresses..."`, or AI self-references.
  - Never mention AI, LLM, Claude, Gemini, or agent tool names in commit messages.

### 5. Git Safety Guardrails (Zero Tolerance)
- **NEVER execute or suggest destructive or history-rewriting commands**:
  - `git push --force` / `git push -f`
  - `git reset --hard`
  - `git clean -fd`
  - `git checkout .` / `git restore .` (clobbering uncommitted work)
- Never overwrite remote history or clobber local working tree changes.

---

## Never Generate (Strict Blacklist)

Do not introduce or propose any of the following:

- **Runtimes & Frameworks**: Electron, Tauri webviews, Node.js, Python, HTTP/REST/WebSockets for media streaming, GTK, or Qt.
- **Android Libraries**: Retrofit, Hilt/Dagger, Room, Firebase, analytics/telemetry SDKs.
- **Async on PC**: Tokio anywhere outside `pc/src/transport/bt.rs`.
- **System bloat**: Windows services or system-wide systemd units (Owlmic PC is strictly a per-user tray app autostarting at user login).
- **Hardcoded networking**: Hardcoded USB tethering subnets (always discover interfaces dynamically).
- **Visual bloat**: Gradients, shadows, glassmorphism, decorative animations, or emojis in code and technical documentation headers.
- **Git bloat**: Compiled binaries (`.apk`, `.exe`, `.aab`, `.dll`, `.so`, `.zip`) committed into the Git repository.

---

## Wire Protocol Reference

- **Framing**: `type u8 | len u32 BE | payload (max 4 MiB)`.
- **Control & State**:
  - `0x00` HELLO
  - `0x10` WELCOME
  - `0x11` PENDING
  - `0x12` REJECT
  - `0x03` HEARTBEAT
  - `0x04` CONTROL
  - `0x05` BYE
- **Media Frames**:
  - `0x01` AUDIO
  - `0x02` VIDEO
  - **Media Header**: `seq u32 BE | ts u64 BE µs | codec u8 | rsv u8` followed by raw media payload.
- **Ports**: TCP `:7653` (control & data), UDP `:7654` (discovery beacon).
- Full wire specification: [`docs/WIRE_PROTOCOL.md`](docs/WIRE_PROTOCOL.md). To modify, follow [`docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md`](docs/DEVELOPER_PLAYBOOKS_AND_SKILLS.md).

---

## UI & Design System (Android & PC Tray)

Follow [`docs/UI_AND_DESIGN_LANGUAGE.md`](docs/UI_AND_DESIGN_LANGUAGE.md):
- Pure functional design: zero shadows, zero gradients, zero rounded glassmorphism, zero decorative spring animations.
- Copy must be calm, concise, and actionable (e.g., *"Tap Allow on your phone"*, not *"ADB authorization error code 3"*).
- **Color Palette (Android OLED Black)**:
  - Background: `#000000` | Sheet: `#111111` | Tile: `#1C1C1C` | Hairline: `#2A2A2A`
  - Text: `#FFFFFF` | Text Secondary: `#8E8E93` | Inactive: `#3A3A3C`
  - Live (the PC is receiving): `#D71921`. A mic or camera that's on turns white.
  - Status OK: `#30D158` | Status Waiting: `#FFD60A` | Status Error: `#FF453A`

---

## Code Quality & Verification Commands

Before proposing changes, verify locally:
- **Android**: unit tests and lint (`./gradlew testDebugUnitTest lintDebug`).
- **PC**: `cargo fmt --check` and `cargo clippy -- -D warnings`.
- Verify performance constraints against [`docs/PERFORMANCE_AND_REALTIME_BUDGETS.md`](docs/PERFORMANCE_AND_REALTIME_BUDGETS.md).

---

## Current Status & Roadmap

Tracked exclusively in [`docs/README.md`](docs/README.md) and [`docs/ROADMAP_AND_TEST_MATRIX.md`](docs/ROADMAP_AND_TEST_MATRIX.md).

---
> Source: [diveshpatil9104/owlmic](https://github.com/diveshpatil9104/owlmic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
