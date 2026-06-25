## risper

> - Risper is a local-first Hebrew dictation utility for macOS.

# AGENTS.md

## Repository Context

- Risper is a local-first Hebrew dictation utility for macOS.
- Runtime audio processing must stay local for the MVP. Do not introduce cloud ASR, telemetry, or transcript upload paths unless the user explicitly changes the product direction.
- Treat `specs.md` as the product and architecture source of truth.
- The app is a SwiftPM-first AppKit macOS app. Use Swift, AppKit, AVFoundation, Carbon/CoreGraphics, and local `whisper.cpp` integration before adding new frameworks.

## Agent Workflow

- Before implementation, read the relevant parts of `specs.md`, `Package.swift`, scripts, and source files touched by the task.
- For macOS app development, always use the `Build macOS Apps` plugin and its relevant task-specific skills before planning or editing code.
- Keep changes small, coherent, and reversible. Separate broad structural refactors from feature or bug-fix changes unless the refactor is required to complete the task safely.
- When the user asks to fix several code-review findings, launch sub-agents to handle independent fixes in parallel where the work can be cleanly separated. Keep tightly coupled or blocking fixes local, and integrate and verify all results before finishing.
- Do not overwrite or revert unrelated user changes. If the worktree is dirty, work with the existing changes.

## Build And Verification

- Run `swift build` after Swift source changes.
- Run `script/build_and_run.sh --verify` when app-bundle behavior, launch behavior, permissions, menus, hotkeys, or lifecycle code changes.
- Use `script/asr_harness.sh` or the local ASR server workflow for transcription-path validation when ASR behavior changes.
- For microphone, hotkey, Accessibility, clipboard, or cross-app paste behavior, document any manual QA that could not be completed from the agent environment.
- Do not add production dependencies, package managers, generated projects, or new build systems without explicit user approval.

## Regression And Debugging Guardrails

- When the user reports that the last commit worked, compare current runtime source files against `HEAD` before changing behavior. Treat the last-known-good runtime path as the baseline.
- For regressions that appear after packaging, signing, bundling, or installer work, first inspect bundle identity, signing, `Info.plist`, installed app path, and TCC permission state before changing Swift runtime logic.
- Keep packaging fixes separate from app-behavior fixes. Do not replace a working event-monitoring, permission, recording, or paste mechanism unless direct evidence shows that mechanism is the cause.
- For microphone, Accessibility, hotkey, overlay, clipboard, or paste changes, verify the real workflow end to end: focus a text field, hold `fn`, confirm the recording indicator appears, release, wait for transcription, and confirm text is inserted at the original cursor.
- Do not treat partial logs as success. Logs that show monitor startup, recording, transcription, or paste events must be paired with manual workflow verification when the behavior depends on macOS focus, permissions, or cross-app input.
- Read `docs/debugging-macos-permissions.md` before diagnosing or changing macOS permissions, app signing, TCC, global hotkey, Accessibility, microphone, clipboard, or cross-app paste behavior.

## Engineering Standards

- Optimize first for clarity, then simplicity, then concision. Code should be easy for the next engineer or agent to read, test, and change.
- Keep responsibilities cohesive. Avoid god objects, god files, and utility dumping grounds. When behavior naturally splits into separate domains, extract along stable domain boundaries rather than by incidental implementation detail.
- Maintain a single source of truth for configuration, paths, hotkeys, permission state, model/server settings, transcript state, and temp-file policy. If data is cached, derived, or duplicated for performance, make the owner and lifetime explicit.
- Avoid speculative generality. Build the behavior required by `specs.md`; defer abstractions until they remove real duplication or clarify a real boundary.
- Prefer proven platform APIs and existing project scripts over bespoke DIY implementations. If custom machinery is necessary, keep it narrow and explain the reason in code or docs.
- When implementing or fixing behavior, identify the underlying invariant, lifecycle, or failure mode before editing. Solve the class of problem across the relevant local surface, not only the visible symptom. For resources, permissions, temporary state, user data, process/global state, and async flows, define acquisition, mutation, cleanup, cancellation, failure, and interruption behavior explicitly where relevant.
- Keep code secure by default: prefer least-privilege permissions, narrow local-only exceptions, validated inputs, safe process/network boundaries, and privacy-preserving failure behavior.
- Reduce toil deliberately. Repeated setup, verification, diagnostics, and recovery steps should become reliable scripts or app diagnostics when they have clear ongoing value.
- Keep documentation short, current, and linked to canonical sources. Update docs in the same change as behavior changes when the docs would otherwise become stale.

## Swift And macOS Conventions

- Use AppKit for menu bar app lifecycle, status items, global hotkeys, pasteboard insertion, permission surfaces, and process management.
- Use AVFoundation for recording and audio conversion.
- Use SwiftUI only when a real settings or richer UI surface is introduced.
- Prefer early exits with `guard` for invalid states and permission failures.
- Keep access levels tight. Default to `private` or file-local scope for implementation details.
- Avoid force unwraps and force casts unless the invariant is local, obvious, and failure would indicate a programmer error.
- Use macOS unified logging for diagnostics, but keep logs privacy-safe.

## Privacy And Safety

- Do not log transcript text or audio contents by default.
- Do not persist audio longer than the current task/spec requires. Temp audio belongs under `~/Library/Caches/Risper/recordings/` unless the spec changes.
- Preserve user clipboard contents around insertion flows.
- Permission-denied, missing-model, ASR-down, empty-audio, and paste-failure paths must fail without crashing and without losing user data.
- Treat Accessibility and microphone permission behavior as user-visible product behavior, not just plumbing.

## Review Lens

- Review design, functionality, complexity, tests, naming, comments, style, and documentation before finishing.
- Watch for code smells as signals to investigate, especially duplicated code, large types/files, hidden global state, mutable shared state, and shotgun-surgery changes.
- When addressing review findings, fix the underlying class of issue, not only the exact reported line. Re-scan adjacent paths for the same invariant, lifecycle, or failure-mode pattern before declaring the finding resolved.
- Prefer small changes with related tests. If a change is hard to test automatically, state the manual verification path.

## Reference Basis

- OpenAI Codex AGENTS.md guidance: https://developers.openai.com/codex/guides/agents-md
- Google Engineering Practices: https://google.github.io/eng-practices/
- Google Swift Style Guide: https://google.github.io/swift/
- Google SRE toil guidance: https://sre.google/sre-book/eliminating-toil/
- DORA technical capabilities: https://dora.dev/capabilities/trunk-based-development/
- AWS Well-Architected operational excellence: https://docs.aws.amazon.com/wellarchitected/2023-10-03/framework/oe-design-principles.html
- Azure Well-Architected automation guidance: https://learn.microsoft.com/en-us/azure/well-architected/operational-excellence/enable-automation
- Martin Fowler on code smells: https://martinfowler.com/bliki/CodeSmell.html
- StaffEng archetypes: https://staffeng.com/guides/staff-archetypes/

---
> Source: [shafnir/Risper](https://github.com/shafnir/Risper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
