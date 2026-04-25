## rotatingartlauncher

> - Reuse existing code paths, components, and classes before creating new ones.

# Repository Guidelines

## Rules
- Reuse existing code paths, components, and classes before creating new ones.
- Do not introduce parallel implementations for behavior already present in `app`.
- Search the internet for existing solutions, references, and prior art first before implementing non-trivial changes, then adapt findings to this codebase.
- Create a new class only when no suitable extension point exists; keep it small and feature-scoped.
- Store configuration in JSON only. For new configuration data, do not introduce XML, YAML, or `.properties` formats.
- Keep configuration schemas backward-compatible when possible and validate migration impacts in PR notes.
- Respect package boundaries inside `app`: reusable infrastructure, repositories, navigation, theme, and platform services belong under `app/src/main/java/com/app/ralaunch/core`; user-facing product flows belong under `app/src/main/java/com/app/ralaunch/feature`.
- Use existing DI patterns (Koin modules, `core/di/contract`, and `core/di/service`) instead of ad-hoc singletons or hidden global state.
- Prefer extending existing feature packages over creating new top-level feature namespaces.
- Treat `app/src/main/cpp` vendored trees (for example `SDL`, `FNA3D`, `FAudio`, `gl4es`) as external code; avoid edits unless dependency work is intentional.
- Keep changes scoped to the task; avoid drive-by refactors in unrelated files.
- Preserve backward compatibility for persisted data and public contracts; document any required migration in the PR.

## Coding Style & Naming Conventions
- Follow Kotlin official style with 4-space indentation.
- Naming: packages `lowercase`, classes/objects `PascalCase`, methods/variables `camelCase`, constants `UPPER_SNAKE_CASE`.
- Keep boundaries clean: platform/runtime concerns in `app/core/platform`, app-wide infrastructure in `app/core`, and user-facing features in `app/feature`.
- Within a package area, keep file roles explicit: Compose-related code belongs under `.../ui`, ViewModels and state coordinators belong under `.../vm`, feature-owned models or DTO-style shapes belong under `.../model`, contracts and interfaces belong under `.../contract`, and concrete service/manager/repository implementations belong under `.../service`.
- No repo-wide ktlint/detekt is configured; run IDE reformat before opening PRs.

## Commit & Pull Request Guidelines
- Match existing history style: `feat(scope): ...`, `fix: ...`, `refactor: ...`, `chore: ...`.
- Keep commits focused and self-contained.
- PRs should include: what changed, why, how it was tested (commands), linked issue (if applicable), and screenshots/video for UI updates.
- For bug fixes, include repro details: device model, Android version, and relevant logs.

---
> Source: [FireworkSky/RotatingartLauncher](https://github.com/FireworkSky/RotatingartLauncher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
