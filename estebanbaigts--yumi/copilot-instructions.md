## yumi

> Yumi is a native macOS app: a small animated character living in the MacBook notch that shows Claude Code sessions and a few integrations, and lets the user approve, answer, chat and drop files from the notch. It is built on the MIT source code of Coucou (see `ATTRIBUTION.md`).

# Yumi: guide for AI coding agents

Yumi is a native macOS app: a small animated character living in the MacBook notch that shows Claude Code sessions and a few integrations, and lets the user approve, answer, chat and drop files from the notch. It is built on the MIT source code of Coucou (see `ATTRIBUTION.md`).

## Read first
- `YUMI.md`: decisions, identity values, character design brief, file ownership per work session. It wins over any other document.
- `ARCHITECTURE.md`: analysis of the 18 systems of the original code, with refactor risks. Paths written `NotchBuddy/...` are `Yumi/...` here.

## Where things are
- `Yumi/Sources/App/`: all Swift code. `Yumi/Resources/sounds/`: the WAV sounds. `Yumi/project.yml`: XcodeGen project.
- `design/yumi/`: character concept sheet (concept 4, left half).
- `Yumi/Sources/App/AgentRuntime/`: the agent runtime (request, plan, tools, permissions, execution). It never acts without going through `PermissionManager`, and captures no context by itself.
- `Yumi/Sources/App/Core/`: typed event engine and session store. `Yumi/Sources/App/Modules/`: one folder per module, each producing a `ModuleSnapshot`. Neither may use a view, `AppState`, the hook server or a sound.
- `Yumi/Tests/`: unit tests (Swift Testing). The bundle is not hosted by the app: a file under test is listed in the `YumiTests` target of `project.yml` and must have no dependency on the rest of the app.
- `scripts/release.sh`: signed and notarized build. Never run it unless the user asks: with `--publish` it pushes a tag and creates a GitHub release.
- `.github/workflows/build.yml`: build and tests on every push.
- Branch `legacy-v0`: the first Yumi attempt (typed event engine, multi-session store, tests). Reference only.

## Build and test
```
cd Yumi && xcodegen && xcodebuild -scheme Yumi -configuration Debug build
cd Yumi && xcodebuild -scheme Yumi -configuration Debug test CODE_SIGNING_ALLOWED=NO
```
Also build the App Store target: `cd Yumi && xcodebuild -scheme YumiAppStore -configuration Debug build CODE_SIGNING_ALLOWED=NO`.

`YUMI_STUDIO=1` is the filming mode, in every build: nothing real starts and nothing personal is read (see `Core/StudioMode.swift`). Debug builds read two environment variables. `YUMI_SUPPORT_DIR=/some/short/path` moves the socket and the hook script there, so a development build never takes the socket of the installed app (keep the path short: a socket path is limited to 104 bytes). `YUMI_TRACE_MODULES=1` prints the module snapshots each time they change. `YUMI_TRACE_CONTEXT=1` prints each event of the Context Engine as JSON. `YUMI_TRACE_AGENT=1` prints each event of the agent runtime as JSON. `YUMI_CHAT_PROMPT="a||b"` sends chat messages at launch and prints the conversation; `YUMI_CHAT_ANSWERS=allow,deny` scripts the answers to that run's permission requests; `YUMI_TRACE_INITIATIVE=60` prints each remark Yumi makes on his own and, after that many seconds, how often the initiative driver was woken. `YUMI_CHAT_ARGS` appends arguments to the `claude` command (for instance `--setting-sources project,local` to keep your own hooks out of a test). The App Store scheme builds to the same product path as the Debug one: rebuild the `Yumi` scheme before running it.

Build in Debug before every commit. The Debug build has 3 known warnings (two unused values, one main actor call in `SettingsView`), the App Store build 2: do not add any.

## Rules
- Swift 6, SwiftUI + AppKit. No third-party dependencies unless truly unavoidable. The character is drawn in code (`Canvas` + `TimelineView`).
- The `.xcodeproj` and the `Info.plist` files are generated: change `project.yml`, never edit them.
- Never copy an icon, sound, image or the Mochi character design from Coucou. This repository is public.
- Secrets live in the Keychain, never on disk or in git.
- No telemetry. Network calls only to services the user configured.
- Never block Claude Code: if the app doesn't answer, the hook exits immediately.
- Never overwrite `~/.claude/settings.json`: dated backup, merge, show the diff, write only after the user confirms.
- Never send an email or approve a Claude Code permission without an explicit click.
- Keep `LICENSE` with the original copyright notice.
- Contributor rules and the review checklist are in `CONTRIBUTING.md`.

---
> Source: [estebanbaigts/Yumi](https://github.com/estebanbaigts/Yumi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
