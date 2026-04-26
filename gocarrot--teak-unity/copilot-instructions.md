## teak-unity

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Teak Unity SDK — a cross-platform Unity plugin providing push notifications, deep linking, rewards, and marketing channel management. Ships as a `.unitypackage` and a UPM package. The C# runtime code wraps native SDKs for iOS (`libTeak.a`), Android (`teak.aar`), and WebGL (`Teak.jslib`).

## Build Commands

The build system uses **Ruby Rake** (Ruby 3.1.6 via rbenv).

```bash
# Install Ruby dependencies
bundle install

# Full build (default): downloads native SDKs, generates version files, creates Teak.unitypackage
bundle exec rake

# Individual build targets
bundle exec rake build:android   # Download Android AAR + generate Version.java
bundle exec rake build:ios       # Download iOS framework + generate teak_version.m
bundle exec rake build:package   # Generate TeakVersion.cs + create Teak.unitypackage

# Build UPM package (output in temp-upm-build/)
bundle exec rake upm:build

# Build against local native SDK checkouts (expects ../teak-android and ../teak-ios)
BUILD_LOCAL=true bundle exec rake
```

**Code formatting:**
```bash
bundle exec rake format
# Runs: astyle --project --recursive Assets/*.cs --exclude=Assets/Teak/Editor/iOS/Xcode
```

**Documentation generation** (requires Node.js, yarn, doxygen):
```bash
yarn docs
```

**Running tests:**
```bash
# Via Unity CLI (project version: see ProjectSettings/ProjectVersion.txt)
/Applications/Unity/Hub/Editor/<VERSION>/Unity.app/Contents/MacOS/Unity \
  -executeMethod TeakTestRunner.RunAll \
  -logFile unity.test.log -quit -nographics -batchmode \
  -projectPath .
```

Tests use a custom runner (`Assets/Tests/Editor/TeakTestRunner.cs`) with `[TeakTestFixture]` / `[TeakTest]` attributes and `TeakAssert` helpers. Test files live in `Assets/Tests/Editor/`. Since there are no `.asmdef` files, all Editor scripts (including build post-processors) compile into `Assembly-CSharp-Editor`, so tests can access `internal` members directly.

## Architecture

### Platform Abstraction via Preprocessor Directives

All platform-specific code uses `#if UNITY_EDITOR / UNITY_ANDROID / UNITY_IPHONE / UNITY_WEBGL` blocks. The `Teak` class and related types contain parallel implementations for each platform within the same files. Editor builds stub out native calls.

### Key Files

| File | Purpose |
|------|---------|
| `Assets/Teak/Teak.cs` | Main singleton `MonoBehaviour` — user identification, event tracking, rewards |
| `Assets/Teak/Teak.Channel.cs` | Notification channel opt-in/out, categories |
| `Assets/Teak/Teak.Notification.cs` | Schedule/cancel notifications |
| `Assets/Teak/Teak.Operation.cs` | Async operation wrapper around native SDK futures |
| `Assets/Teak/Teak.UserData.cs` | User data model (push/email/SMS status) |
| `Assets/Teak/TeakNotification.cs` | Notification metadata from native layer |
| `Assets/Teak/TeakReward.cs` | Reward claim model and status enum |
| `Assets/Teak/TeakSettings.cs` | `ScriptableObject` for editor config (App ID, API Key) |
| `Assets/Teak/TeakVersion.cs` | Auto-generated from `Templates/TeakVersion.cs.template` — do not edit |
| `Assets/Teak/MiniJSON.cs` | Bundled JSON serializer (namespace `MiniJSON.Teak`) |
| `Assets/Teak/Plugins/iOS/` | iOS native binary, init code, privacy manifest |
| `Assets/Teak/Plugins/Android/` | Android AAR, version class, androidlib wrapper |
| `Assets/Teak/Plugins/WebGL/Teak.jslib` | JavaScript interop loading `teak.min.js` from CDN |
| `Assets/Teak/Editor/` | Build post-processors for Xcode and Android manifest |

### Patterns

- **Singleton access:** `Teak.Instance` lazily creates a `TeakGameObject` with `DontSave` flag.
- **Partial classes:** `Teak` is split across `Teak.cs`, `Teak.Channel.cs`, `Teak.Notification.cs`, `Teak.Operation.cs`, `Teak.UserData.cs`.
- **Async operations:** `Teak.Operation` wraps platform-specific futures and fires `OnDone` callbacks with JSON results.
- **Version compat defines:** `TeakPreProcessDefiner.cs` adds `TEAK_2_0_OR_NEWER` through `TEAK_4_3_OR_NEWER` scripting defines at build time.

### Version Management

- `VERSION` file at repo root contains the Unity SDK version (e.g. `4.3.10`). Updated at release time, not during development.
- `native.config.yml` specifies native SDK versions for iOS and Android.
- Template files in `Templates/` are rendered by Mustache during `rake` build to generate `TeakVersion.cs`, `Version.java`, and `teak_version.m`. Do not edit these generated files directly.
- **Tagging**: The `teak/tag-promote` CI orb parses the HEAD commit message for `Promote to: X.Y.Z` and creates + pushes a git tag.

### Release Flow

```
# 1. Update VERSION file to new version (e.g. 4.3.10)
# 2. Move docs/modules/changelog/unreleased.yaml → versions/X.Y.Z.yaml
# 3. Create fresh unreleased.yaml
# 4. Inline relevant native SDK changes as regular bug/enhancement/new entries
git commit -m "Promote to: X.Y.Z.rc0"
git push
# CI: orb detects commit message → tags X.Y.Z.rc0 → tagged-build workflow → deploy to S3
```

RC → final release flow:
- **rc0**: Promotes with current native SDK versions (may be betas). Used for integration testing.
- **rc1+**: Update `native.config.yml` to finalized native SDK versions, rebuild, promote again.
- **Final**: When an RC passes testing, promote with the final version number (`Promote to: X.Y.Z`).

**Version numbers are immutable.** Once a promote commit is pushed and CI tags it, that version is permanently consumed. Tags cannot be moved or reused. When promoting, always check recent commit messages (`git log --oneline`) to determine the next available version number.

## Code Style

- **Brace style:** Attached (K&R), enforced by astyle via `Assets/.astylerc`.
- **Indentation:** Indent switches and namespaces.
- **One-line braces:** Added to single-statement blocks.
- **Padding:** Space after control keywords (`if (foo)` not `if(foo)`).
- `MiniJSON.cs` is excluded from formatting.

## CI/CD

CircleCI on macOS (Xcode 14.3.1). Two workflows:
- **un-tagged-build:** Builds package, optionally promotes to tagged release.
- **tagged-build:** Builds, deploys versioned UPM package to `GoCarrot/upm-package-teak`, uploads `.unitypackage` to S3 (`teak-build-artifacts` bucket). `deploy_latest` requires manual approval.

## Documentation & Changelog

Public documentation is built with **Antora** and lives under `docs/`. API docs are generated via doxygen (`yarn docs`).

### Changelog System

The changelog is published to the documentation site. Source of truth is YAML files in `docs/modules/changelog/versions/`. AsciiDoc partials and the main changelog page are **generated** by `doxygen2adoc` (via `yarn docs`) and gitignored — never edit `.adoc` files under `docs/modules/changelog/`.

- **Adding entries:** Edit `docs/modules/changelog/unreleased.yaml`. When a user-facing change is made, add a line under the appropriate category.
- **Important:** `unreleased.yaml` lives in `docs/modules/changelog/`, NOT in `versions/`. The `versions/` directory is read by `doxygen2adoc` which requires valid semver filenames — `unreleased.yaml` in that directory will break `yarn docs`.
- **At release time:** Move `unreleased.yaml` into `versions/<version>.yaml` and create a fresh `unreleased.yaml`.
- **YAML categories:** `new`, `bug`, `enhancement`, `breaking`, `deprecation`, `upgrade_note`, `known_issue`, `android` (native version), `ios` (native version)
- **Format:** Each category is a list of strings. Use backticks for code references (doubled in YAML: ` `` `).

## Branching

- `develop` — active development (default PR target)
- `X.Y-stable` — maintenance branches (e.g. `4.3-stable`)

## Commit Message Format

Every commit must include a full human-Claude interaction log. Placeholders
below are in angle brackets — replace them with actual content, do not include
the brackets. The interaction log only covers prompts since the previous commit,
not the entire session.

```
Brief description of what was done

Technical description of changes made

## Human-Claude Interaction Log

### Human prompts (VERBATIM - include typos, informal language, COMPLETE text):
**Include EVERY prompt since last commit - even short ones, corrections, clarifications**
1. "<copy-paste ENTIRE first prompt since last commit>"
   → Claude: <what Claude did in response>

2. "<copy-paste ENTIRE second prompt>"
   → Claude: <how Claude adjusted>

<continue numbering ALL prompts - don't skip any or judge importance>

### Key decisions made:
- Human guided: <specific guidance provided>
- Claude discovered: <patterns found>

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
```

## How We Work

If given a GitHub project, identify an issue that you can work on independently. Prefer to work on the smallest, most tightly-scoped issue.

If given a GitHub issue (or when choosing one from a project), read the full issue and all comments. If there's anything that you're unclear on or points that require decisions, **ask**. For decisions, present possible options with pros and cons.

When you start work, create a new branch with the appropriate prefix (`fix/`, `feat/`, `maint/`) and a short descriptive name.

When you work, prefer to make small single purpose commits. Particularly when addressing PR feedback, use separate commits for each piece of feedback that isn't clearly connected to other feedback. If you're unsure, present your commit plan and ask for feedback.

When you're done with your work, push the branch and open a PR if one is not already open. Do not merge without human approval. After opening the PR or pushing, use a Task agent to request a review of the PR with **exactly** `/review <PR Number>`. Consider the feedback -- if any is an obvious improvement address it, if any requires a decision and you're uncertain, present the decision with options and pros/cons.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GoCarrot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
