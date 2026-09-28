## mikey

> > `CLAUDE.md` imports this file via `@AGENTS.md`. Edit this file directly — see [update-agent-files](vibe/skills/update-agent-files.md).

# Mikey — Agent Instructions

> `CLAUDE.md` imports this file via `@AGENTS.md`. Edit this file directly — see [update-agent-files](vibe/skills/update-agent-files.md).

## The Brain (Source of Truth)

The brain in [`vibe/`](vibe/index.md) is the absolute source of truth for every architectural and implementation decision. It wins over this file on any conflict. Read the index before writing any code:

@vibe/index.md

| Part | Folder | Purpose |
|---|---|---|
| **Core** | [`vibe/core/`](vibe/core/) | Vision, principles, scope, UX specs, roadmap, risks, decisions |
| **Architecture** | [`vibe/architecture/`](vibe/architecture/) | System design, connection levels, trust, wire protocol, media, tech stack, security, repo layout |
| **Rules** | [`vibe/rules/`](vibe/rules/) | Coding rules, design language, performance budgets, test matrix, agent files |
| **Skills** | [`vibe/skills/`](vibe/skills/) | Playbooks: build & run Android, add dependency, change protocol, change decision, update agent files |

---

## Project Overview & Ownership

- **Project in one line**: Android app (Kotlin/Compose) + per-user PC tray app (Rust, binary `mikey`) that turns a phone into a mic and webcam for a PC over four automatic connection levels:
  `1 USB debugging (adb reverse)` > `2 USB tethering` > `3 Bluetooth RFCOMM (audio only)` > `4 Wi-Fi`.
- **Ownership boundaries**:
  - `android/` — repo owner's domain.
  - `pc/` — separate contributor's domain. **Do not modify or add code in `pc/` unless explicitly requested by the user.**
- **Roles**: The phone is **always the client**; the PC is **always the server**.

---

## Architecture & Invariants

1. **Android App** (`android/`): `MikeyService` foreground service owns capture + transport. `TransportManager` negotiates the best of 4 levels with make-before-break upgrades.
2. **PC Tray App** (`pc/`): TCP `:7653`, UDP discovery beacon `:7654`, RFCOMM server, adb watcher, `SessionManager` (tokens, trust, ask-before-join), audio pipeline (Opus, jitter buffer, drift resampler, noise gate, RNNoise, SpeexDSP AEC), video pipeline (JPEG -> virtual camera).
3. **Threading & Concurrency**:
   - PC: Blocking std threads + bounded channels.
   - **The ONLY async code allowed is Tokio inside `pc/src/transport/bt.rs` on Linux** (required by `bluer`). Everywhere else, Tokio and async runtime are strictly forbidden.
4. **Real-time Safety**:
   - Audio capture and playback paths must never block on I/O or locks held by other threads. Use bounded, lock-free, or try-lock handoffs; when full, drop oldest data.
   - Every socket must have a timeout. Every queue/channel must have a strict upper bound. No unbounded buffering.
5. **State & Permissions**:
   - Mic and camera always initialize in the **OFF** state. Never auto-enable capture from the background or upon connection.
   - UI never owns logic: Compose observes `StateFlow` from `MikeyService`; PC tray reflects `SessionManager`.

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
   - Do not add any new library or crate unless listed in [`vibe/architecture/tech-stack.md`](vibe/architecture/tech-stack.md).
   - Any dependency addition requires explicit user approval and following [`add-a-dependency`](vibe/skills/add-a-dependency.md).
5. **No Code in Documentation**:
   - Never place operational code inside `vibe/`. Notes and design thoughts belong in discussions or edits to the brain.

---

## Git Workflow: Branching, Staging & Commits

Strict Git discipline must be maintained at all times.

### 1. Branching Rules
- **Naming format**: `<type>/<short-kebab-description>`
  - `feat/pc-audio-receiver`
  - `fix/pc-cross-platform-and-protocol`
  - `chore/update-dependencies`
  - `docs/clarify-wire-spec`
- **Scope**: Keep branches short-lived and focused on a single feature, fix, or task.
- **Base**: Always branch from and target `main` (or the active tracking branch specified by the user).
- **Protected branches**: Never commit directly to `main` without explicit user direction.

### 2. Staging Rules
- **No blind mass-staging**: **NEVER run `git add .` or `git add -A`**.
- **Inspect before staging**: Always run `git status` and `git diff` to review modified files.
- **Stage surgical paths**: Stage only the specific files relevant to the completed task (`git add path/to/file`).

### 3. Commit Message Rules
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
- **USER CONFIRMATION REQUIRED**:
  - **NEVER execute `git commit` or `git push` without asking the user for confirmation first.**
  - Show the proposed commit message and staged files when asking for approval.

### 4. Git Safety Guardrails (Zero Tolerance)
- **NEVER execute destructive or history-rewriting commands**:
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
- **System bloat**: Windows services or system-wide systemd units (Mikey PC is strictly a per-user tray app autostarting at user login).
- **Hardcoded networking**: Hardcoded USB tethering subnets (always discover interfaces dynamically).
- **Visual bloat**: Gradients, shadows, glassmorphism, or decorative animations in UI.

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
- Full wire specification: [`vibe/architecture/wire-protocol.md`](vibe/architecture/wire-protocol.md). To modify, follow [`vibe/skills/change-the-protocol.md`](vibe/skills/change-the-protocol.md).

---

## UI & Design System (Android & PC Tray)

Follow [`vibe/rules/design-language.md`](vibe/rules/design-language.md):
- Pure functional design: zero shadows, zero gradients, zero rounded glassmorphism, zero decorative spring animations.
- Copy must be calm, concise, and actionable (e.g., *"Tap Allow on your phone"*, not *"ADB authorization error code 3"*).
- **Color Palette (Android OLED Black)**:
  - Background: `#000000` | Surface: `#111111` | Divider: `#2C2C2E`
  - Text Primary: `#FFFFFF` | Text Secondary: `#8E8E93` | Icon Off: `#3A3A3C`
  - Mic Active: `#30D158` | Camera Active: `#0A84FF`
  - Status OK: `#30D158` | Status Waiting: `#FFD60A` | Status Error: `#FF453A`

---

## Code Quality & Verification Commands

Before proposing changes, verify locally:
- **Android**: ktlint check and unit tests (`./gradlew ktlintCheck test`).
- **PC**: `cargo fmt --check` and `cargo clippy -- -D warnings`.
- Verify performance constraints against [`vibe/rules/performance-budgets.md`](vibe/rules/performance-budgets.md).

---

## Current Status & Roadmap

Tracked exclusively in [`vibe/index.md`](vibe/index.md) and [`vibe/core/roadmap.md`](vibe/core/roadmap.md).

---
> Source: [diveshpatil9104/mikey](https://github.com/diveshpatil9104/mikey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
