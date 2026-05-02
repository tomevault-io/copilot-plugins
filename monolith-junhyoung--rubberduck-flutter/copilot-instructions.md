## rubberduck-flutter

> DuckPilot is the mobile controller app for `청소난투(욕실의 난)`, a four-player bathroom battle game.

# DuckPilot

## Project Overview

DuckPilot is the mobile controller app for `청소난투(욕실의 난)`, a four-player bathroom battle game.

It is a one-hand portrait controller that sends tilt-based input to the game via Azure Web PubSub.

Current app identity:

- App: `RubberDuck Pilot`
- Android: `com.rubberduck.pilot`
- iOS: `com.rubberduck.pilot`
- Launch: `https://duckpilot.vercel.app/launch`
- Install: `https://duckpilot.vercel.app/install`

Core flow: join a session, hold to steer with gyro input, and use left / stop / right controls.

Runtime connection is based on launch URL bootstrap and no longer depends on a hardcoded client URL.

## Code Conventions

- Use single quotes for strings (enforced by linter)
- Prefer const constructors and final variables
- Use package imports (always_use_package_imports)
- Avoid print statements (use proper logging)
- Follow Riverpod patterns with providers in separate files
- Use Freezed for data models with immutability
- All Freezed model should be an abstract class.
- Implement proper error handling with try-catch blocks

## Key Rules (AI must follow)

- [MANDATORY] Avoid using deprecated APIs or method and legacy patterns. Use modern and latest coding patterns.
  **Prefer modern syntax:** Always use the most recent stable Dart/Flutter features (e.g., Records, Pattern Matching, Class Modifiers).
- [MANDATORY] **DO NOT USE** \_handle* or handle* to callback method name. Use \_on* or on*.
- [MANDATORY] Avoid helper methods like `_buildSomething()` in `build()` for child widgets.
  - Do not do: `build(...) => Column(children: [_buildHeader(), _buildItem(...)])`.
  - Do instead: use inline widget code, local constants/widgets, or extracted `StatelessWidget` classes.
- [MANDATORY] For list UI, keep item rendering as dedicated widget classes (`Widget` subclasses) instead of build-method helpers.

---
> Source: [monolith-junhyoung/rubberduck-flutter](https://github.com/monolith-junhyoung/rubberduck-flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
