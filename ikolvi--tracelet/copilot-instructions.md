## tracelet

> You are working in a Flutter/Dart workspace managed by Melos.

You are working in a Flutter/Dart workspace managed by Melos.

CRITICAL WORKFLOW RULE:
Before EVERY commit and push, you MUST run the following commands:
1. `melos run format` or `melos run format:fix`
2. `melos run analyze`

If these commands fail, you MUST fix the formatting and analyzer errors before committing. Failure to do so will cause the CI workflows to fail.

RELEASE/PUBLISHING WORKFLOW:
When bumping versions or generating changelogs, DO NOT manually edit `CHANGELOG.md` or version numbers in `build.gradle.kts` / `podspec` files.
Instead, use `melos version`. This command will:
1. Automatically update versions and generate changelogs for all Flutter packages (using Conventional Commits).
2. Trigger the `scripts/sync_native_versions.py` hook to automatically synchronize versions and changelogs for Android/iOS Native SDKs.

---
> Source: [Ikolvi/Tracelet](https://github.com/Ikolvi/Tracelet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
