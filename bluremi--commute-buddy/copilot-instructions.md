## commute-buddy

> This file describes how to work in this project. Follow these instructions exactly.

# Project Rules

This file describes how to work in this project. Follow these instructions exactly.

## Project Documents

- **`prd.md`** — The living product document. Starts as a requirements blueprint; grows into full documentation of what was actually built. Contains: problem statement, approach, key features, technical architecture, and the feature/bug backlog.
- **`plan.md`** — The active development document. Contains only the story currently being worked on: the expanded user story and its step-by-step implementation plan.

## Development Workflow

### Executing Implementation Plans

When working on implementation plans (multi-increment plans in `plan.md`):

**CRITICAL: ONE INCREMENT AT A TIME.**

For each increment:
1. Implement the changes described for that increment ONLY
2. Commit with a descriptive message
3. Stop — tell the user "Ready for testing" and WAIT
4. Do NOT start the next increment
5. Wait for the user to explicitly confirm testing passed
6. Only then: mark the increment complete (`[x]`) in `plan.md` and proceed to the next

**DO NOT:**
- Implement multiple increments in one response
- Assume testing passed and move on
- Continue without explicit user confirmation ("looks good", "test passed", "continue", etc.)
- Start the next increment just because the user is silent
- Say "I'll proceed with the next increment" and then do it

**The user will explicitly tell you when to proceed to the next increment.**

### Completing a Story

When all increments in a story are marked complete:

**This step is mandatory. Do not skip it.**

1. **Update Key Features in `prd.md`** — Revise the Key Features section to accurately describe what the app now does. Add new features; update or remove features that changed.

2. **Update Technical Architecture in `prd.md`** — Update Tech Stack, System Design, Key Files, and Commands to reflect what was actually built. Add new libraries, files, patterns, and architectural decisions. Remove anything no longer accurate. This section must always describe the real current state of the codebase — not a plan.

3. **Mark the backlog item complete** — Check off the item (`[x]`) in the `prd.md` backlog.

4. **Clear `plan.md`** — Remove the completed story so `plan.md` only ever contains active work.

prd.md is the ground truth of what the project is. Keeping it accurate is as important as writing the code.

## General Principles

- Read existing code before modifying it. Do not propose changes to code you haven't read.
- Prefer editing existing files over creating new ones.
- Avoid over-engineering. Only make changes that are directly requested or clearly necessary.
- Do not add features, refactor code, or make "improvements" beyond what was asked.
- Do not add error handling, fallbacks, or validation for scenarios that can't happen.
- Do not create abstractions for one-time operations.

## Windows / PowerShell Notes

- **No `gradlew` scripts in the repo.** `android/gradlew` and `android/gradlew.bat` are not committed. Do NOT try `.\gradlew` or `.\gradlew.bat` — they don't exist. Use the Gradle binary from the cached wrapper distribution:
  ```powershell
  $gradle = (Get-ChildItem "$env:USERPROFILE\.gradle\wrapper\dists\gradle-8.13-bin" -Recurse -Filter "gradle.bat" | Select-Object -First 1 -ExpandProperty FullName)
  Set-Location "a:\Phil\Phil Docs\Development\commute-buddy\android"
  & $gradle :app:testDebugUnitTest
  ```
- **No heredoc in PowerShell.** `$(cat <<'EOF' ... EOF)` is bash syntax and will fail. For `git commit` messages, use `-m "single line message"`.
- **Path spaces require quoting.** Always quote paths with spaces. `cd "path" && command` chaining is unreliable in PowerShell — use separate `Set-Location` + `& command` calls.

## Project-Specific Notes

- **Two IDEs required:** Android Studio for the Kotlin/Android app, VS Code for the Garmin/Monkey C app
- **Phase 1 testing** (UI/logic): Use Connect IQ Simulator in VS Code — no hardware needed
- **Phase 2 testing** (BLE): Physical phone + Garmin Venu 3 via USB sideloading
- **BLE payload must stay under 1KB** — Garmin Glance memory limit is ~32KB
- **Never parse protobuf on the watch** — all heavy lifting happens on Android
- **Connect IQ Android SDK:** Use `getDeviceStatus()` not `getStatus()`, and `IQDevice.IQDeviceStatus` not `ConnectIQ.IQDeviceStatus`. See `docs/garmin/android-sdk-api-notes.md` before modifying device/app discovery code.

---
> Source: [bluremi/commute-buddy](https://github.com/bluremi/commute-buddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
