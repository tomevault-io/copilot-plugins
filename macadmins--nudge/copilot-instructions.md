## nudge

> Nudge is a native macOS application written in Swift and SwiftUI that encourages

# Nudge agent instructions

## Project

Nudge is a native macOS application written in Swift and SwiftUI that encourages
users to install macOS updates. Work within the existing Xcode project and
architecture. The project currently uses Swift 5 language mode and a macOS 12.0
deployment target; check `Nudge.xcodeproj/project.pbxproj` before choosing APIs.
Do not raise these requirements unless the task calls for it.

Read [.agents/swift-swiftui.md](.agents/swift-swiftui.md) before changing Swift
code or the user interface. The `.agents/` folder holds repository-specific
guidance; this file is the entry point.

## Repository map

- `Nudge/UI/`: app entry point, shared UI, and simple/standard layouts.
- `Nudge/Utilities/`: preferences, update behavior, OS versions, logging, and UI logic.
- `Nudge/Preferences/`: preference structures and defaults.
- `Nudge/3rd Party Assets/`: SOFA and GDMF integration code.
- `Localizable.xcstrings`: localized strings.
- `NudgeTests/` and `NudgeUITests/`: XCTest targets.
- `Example Assets/` and `Schema/`: example configuration and management schema.
- `build_assets/`, `Nudge/Scripts/`, and `build_nudge.zsh`: deployment and packaging.

## Build and validation

Run commands from the repository root with a full Xcode installation selected:

```sh
xcodebuild -project Nudge.xcodeproj -scheme 'Nudge - Debug' -configuration Debug -destination 'platform=macOS' build
xcodebuild -project Nudge.xcodeproj -scheme 'Nudge - Debug' -destination 'platform=macOS' test
```

For focused unit validation, append `-only-testing:NudgeTests` or an individual
test identifier to the test command. The Debug scheme supplies `-unit-testing`
to its test action. Report toolchain or signing failures explicitly.

For UI verification, use the shared `Nudge - Debug (-demo-mode)` and
`Nudge - Debug (-demo-mode, -simple-mode)` schemes. Check both layouts and light
and dark appearances when a change affects shared UI. UI tests currently contain
commented-out launch tests, so passing that target alone does not verify visuals.

Use direct Xcode commands for routine validation. `build_nudge.zsh` is a release
workflow: it changes versions, switches Xcode with sudo, and can sign and package
artifacts. Run it only when release or packaging work is in scope.

Run checks appropriate to the change. Documentation-only edits need link and
diff checks, not an application build. State what was tested and any gaps.

For deadline, grace-period, SOFA, preference, or startup changes, read
[.agents/simulation-testing.md](.agents/simulation-testing.md) and run relevant
configuration-driven app simulations with unified-log assertions in addition to
unit tests. Use `-simulate-os-version`, `-simulate-date`, and local JSON/SOFA
fixtures. Confirm the evaluated date and decision in logs. Keep test preferences
and caches isolated from production Nudge; the documented runner handles this.

## Working conventions

- Follow nearby code style and keep changes focused on the requested behavior.
- Preserve existing edits and avoid unrelated formatting or project-file churn.
- When adding Swift files, ensure they belong to the correct Xcode target.
- Keep preference names, defaults, decoding, examples, and schema consistent
  when configuration behavior changes.
- Preserve deferral, deadline, grace-period, and update behavior unless changing
  that behavior is part of the task. Add focused regression coverage for it.
- Use the MacAdmins Python shebang for repository Python scripts:
  `#!/Library/ManagedFrameworks/Python/Python3.framework/Versions/Current/bin/python3`.
- Keep generated build products, credentials, and machine-specific settings out
  of version control.

## Agent coordination

Use the capabilities available in the current agent environment. Work locally
when that is the simplest way to complete the task. Use supported parallel agents
for independent work when useful, with clear ownership and instructions to
preserve each other's changes. Do not require Claude-specific agent teams or
pause authorized work because a particular coordination feature is unavailable.
Prepared plans may be executed locally; no particular skill or delegation mode
is mandatory.

---
> Source: [macadmins/nudge](https://github.com/macadmins/nudge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
