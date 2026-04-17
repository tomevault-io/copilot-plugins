## dart-rules

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**dart_rules** is a pure Dart package providing shared lint rules for all FastoGT Dart/Flutter projects. It defines a curated `analysis_options.yaml` that downstream projects include via `package:dart_rules/analysis_options.yaml`. Hosted on GitHub (github.com/fastogt/dart_rules).

## Build Commands

```bash
dart pub get
dart analyze
dart format . --line-length=100
```

There are no tests since this package contains only lint configuration, no executable Dart code.

## Architecture

- **`lib/analysis_options.yaml`** — the shared lint ruleset. Builds on top of `package:lints/core.yaml` with ~80 curated rules covering correctness (e.g., `avoid_empty_else`, `await_only_futures`, `hash_and_equals`), style (e.g., `annotate_overrides`, `prefer_final_fields`, `slash_for_doc_comments`), and unnecessary code elimination.
- **`lib/dart_rules.dart`** — empty library declaration (required by Dart package conventions).
- **`analysis_options.yaml`** (root) — self-includes `lib/analysis_options.yaml` for dogfooding.

## Usage

Downstream projects add this as a dev dependency via git and include the rules:

```yaml
# pubspec.yaml
dev_dependencies:
  dart_rules:
    git:
      url: https://github.com/fastogt/dart_rules.git
      ref: main

# analysis_options.yaml
include: package:dart_rules/analysis_options.yaml
```

Projects can override specific rules locally in their own `analysis_options.yaml`.

## Formatting

All FastoGT Dart/Flutter projects enforce 100-character line length: `dart format . --line-length=100`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fastogt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
