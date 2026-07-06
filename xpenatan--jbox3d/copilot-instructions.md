## jbox3d

> Any change is not done until the affected workflow has been validated as working.

# AGENTS.md

## Validation Is Required

Any change is not done until the affected workflow has been validated as working.

Validation requirements:
- Validate the actual affected workflow end to end. Build or generate prerequisites when needed, but `build`, `jar`, `assemble`, or compilation-only tasks are not enough when the workflow is a runnable app or sample.
- Run the documented Gradle task, app, sample, or command that exercises the changed behavior.
- For visual apps and samples, capture proof for every affected runtime/architecture:
  - Capture the actual target window only, not the full desktop or an unrelated foreground window. Wait until the app, browser page, emulator, or device is visibly loaded and ready before taking proof. If the target window is hidden, minimized, behind another window, or launched invisible, restore/show it and bring it to the front before capturing.
  - Desktop: validate JNI, FFM, and C when affected. Launch each runnable desktop sample/app and capture a screenshot. If a runtime is compile/package-only, record that explicitly and run its strongest available validation.
  - Web: validate both JavaScript and WebAssembly. Launch both JS and WASM run tasks/servers, open each in a browser, and capture browser screenshots.
  - Android: start an Android emulator or use a connected device, install/run the sample/app, and capture a screenshot from the emulator/device.
  - iOS: on macOS with Xcode tooling available, start an iOS Simulator or use a connected device, install/run the sample/app, and capture a screenshot from the simulator/device.
- If validation fails because a generated/native artifact is missing, report the missing prerequisite and ask how to proceed before building or fixing it.
- Do not count an expected failure as success unless the requested work is explicitly partial setup.
- If full validation is blocked by a missing external toolchain or environment setup, report the exact blocker and the closest passing validation completed.
- Final responses must include the validation performed and screenshot paths or browser/emulator/device proof when visual validation succeeds.

---
> Source: [xpenatan/jBox3D](https://github.com/xpenatan/jBox3D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
