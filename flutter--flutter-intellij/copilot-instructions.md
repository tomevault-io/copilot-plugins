## flutter-intellij

> @.gemini/styleguide.md

# Flutter IntelliJ Plugin — Claude Code Guide

@.gemini/styleguide.md

## Additional Rules

- No I/O or heavy computation on the EDT (IntelliJ Threading Model).
- All `AnAction` subclasses must be stateless (no mutable instance fields).
- Use `io.flutter.logging.PluginLogger` (or IntelliJ's `Logger`) for all logging; never `System.out`.
- All new files must include the standard Chromium Authors copyright header.
- Zero-Formatting Policy: do not comment on indentation, spacing, or brace placement.
- Categorize code suggestions with `[MUST-FIX]`, `[CONCERN]`, or `[NIT]` severity prefixes.

## Agent Skills & Workflows

- For standard project maintenance workflows and automated procedures (PR cleanup, verification fixes, unit testing, platform version removal, etc.), refer to the executable skills in `.agents/skills/`.

---
> Source: [flutter/flutter-intellij](https://github.com/flutter/flutter-intellij) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
