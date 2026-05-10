## swiftagent

> **Native Swift SDK for building autonomous AI agents with Apple's FoundationModels design philosophy**

# AGENTS.md

**Native Swift SDK for building autonomous AI agents with Apple's FoundationModels design philosophy**

SwiftAgent simplifies AI agent development by providing a clean, intuitive API that handles all the complexity of agent loops, tool execution, and adapter communication. Inspired by Apple's FoundationModels framework, it brings the same elegant, declarative approach to cross-platform AI agent development.

---

## How to work in this repo

These are the defaults and conventions that keep changes consistent and easy to review.

### Expectations

- **After code changes:** build the app to make sure it still compiles.
- **After test changes:** run the relevant unit/UI tests (and the suite when appropriate).
- **Text & localization:** use the repo’s SwiftUI localization approach (String Catalog with plain `Text` / `LocalizedStringKey`).
- **Style bias:** readability beats cleverness; keep types and files small where possible.
- **Commits:** only commit when you’re explicitly asked to.

### Before you wrap up

- Always build the project for all supported platforms (and run tests if your changes touch them or could reasonably affect them).
- If you changed Swift files, always run: `swiftformat --config ".swiftformat" {files}`

---

## Project guidelines

### Documentation

- If you touch it, give it solid doc strings.
- For anything non-trivial, leave a comment explaining the "what" and “why”.

### Swift & file conventions

- Prefer descriptive, English-like names (skip abbreviations unless they’re truly standard).
- If a file is getting large or multi-purpose, feel free to split it into reusable components when that improves clarity.

### SwiftUI view organization

- In view types, declare properties as `var` (not `let`).
- Use `#Preview(traits: .tesseraDesigner)` for previews.
- For state-driven animation, prefer `.animation(.default, value: ...)` over scattered `withAnimation`.
  - Put `.animation` as high in the hierarchy as you can so containers/scroll views animate naturally.
- Prefer `$`-derived bindings (`$state`, `$binding`, `@Bindable` projections).
  - Avoid manual `Binding(get:set:)` unless it genuinely simplifies an adaptation (optional defaults, type bridging, etc.). If you do use it, leave a short note explaining why.
- Prefer `.onChange(of: value) { ... }` with no closure arguments; read `value` inside the closure.
- Push `@State` as deep as possible, but keep it as high as necessary. Don’t default to hoisting everything to the root.

### Layout, spacing, and styling

- Use `Layout.Spacing` and `Layout.Padding` tokens from `DesignSystem/Layout.swift`.
- Use the `HStack`/`VStack` custom initializers that accept spacing tokens.
- Use the `.padding(_:)` extension that takes `Layout.Padding`.
- For consistent visuals, reach for `Card`, `CollapsibleCard`, and `cardStyle` (material backgrounds + borders).

### Control views pattern

- `ControlView` is the standard wrapper for label/subtitle + content + trailing accessory.
- Use `Divider()` between grouped control rows; when needed, pad dividers so they align with card edges.

### Localization and text

- Use string literals in `Text` and `LocalizedStringKey` (String Catalog).
- Use `String(localized:)` outside SwiftUI or when a `LocalizedStringKey` initializer isn’t available.
- Use `.help(...)` for macOS tooltips when it adds value (it’s cross-platform, but only displays on macOS).

---

## Build & test commands

- Build SDK
  - `xcodebuild -workspace SwiftAgent.xcworkspace -scheme ExampleApp -destination "platform=iOS Simulator,name=iPhone 17 Pro,OS=latest" build -quiet`
- Build AgentRecorder (CLI)
  - `xcodebuild -workspace SwiftAgent.xcworkspace -scheme AgentRecorder -destination "platform=macOS" build -quiet`
- Build Tests
  - `xcodebuild -workspace SwiftAgent.xcworkspace -scheme SwiftAgentTests build -quiet`
- Run Tests
  - `xcodebuild -workspace SwiftAgent.xcworkspace -scheme SwiftAgentTests -testPlan SwiftAgentTests test -quiet`
- Always pass `-quiet` to `xcodebuild` to keep logs readable. If something fails and you need more logs, drop it temporarily.

### Record HTTP fixtures (AgentRecorder)

- Output is printed to stdout (Xcode: Debug console).
- Project skill: `.codex/skills/agent-recorder-fixtures/SKILL.md` (workflow for updating test fixtures).
- Keys: set `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` or point `AGENT_RECORDER_SECRETS_PLIST` at a local `Secrets.plist` containing `OpenAI_API_Key_Debug` / `Anthropic_API_Key_Debug`.

For example: 
```bash
# Apple Silicon: add `arch=arm64` to avoid “multiple matching destinations” warnings.
xcodebuild -workspace SwiftAgent.xcworkspace -scheme AgentRecorder -destination "platform=macOS,arch=arm64" -derivedDataPath .tmp/DerivedData build -quiet
./.tmp/DerivedData/Build/Products/Debug/AgentRecorder --list-scenarios
./.tmp/DerivedData/Build/Products/Debug/AgentRecorder --secrets-plist Secrets.plist --provider openai --scenario openai/streaming-tool-calls/weather --no-include-headers
```

---
> Source: [SwiftedMind/SwiftAgent](https://github.com/SwiftedMind/SwiftAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
