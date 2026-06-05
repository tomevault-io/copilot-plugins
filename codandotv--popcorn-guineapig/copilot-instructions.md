## popcorn-guineapig

> This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Project Overview

Popcorn Gradle Plugin enforces architectural rules in multi-module Kotlin projects (KMP, Java, Android). It validates module dependencies against configurable rules.

## Core Commands

- **Run tests + coverage**: `./gradlew popcornguineapigplugin:koverHtmlReport`
- **Run single test**: `./gradlew popcornguineapigplugin:test --tests "path.to.TestClass"`
- **Build plugin**: `./gradlew popcornguineapigplugin:build`
- **Clean**: `./gradlew clean`
- **Publish**: `./gradlew popcornguineapigplugin:publish --no-daemon`

## Architecture

Three-layer structure under `popcornguineapigplugin/src/main/kotlin/com/github/codandotv/popcorn/`:

- `presentation/` — Gradle plugin entry point, tasks, DSL
- `domain/` — Pure business logic: rules, use cases, models, repository interface
- `data/` — Repository implementation, report formatting
- `ServiceLocator.kt` — Wires dependencies together

See **SKILLS.md** for detailed architecture rules, project structure, build configuration, CI/CD, and development workflows.

---
> Source: [CodandoTV/popcorn-guineapig](https://github.com/CodandoTV/popcorn-guineapig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
