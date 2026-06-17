## start-your-python

> These rules are written as the shared baseline for this project family.

# AGENTS.md

## Reusable Rules

These rules are written as the shared baseline for this project family.

- Keep the `Reusable Rules` block aligned across sibling repositories unless the user explicitly asks for a deliberate deviation.
- Treat reusable-rule updates as bidirectional synchronization: when shared rules change in one repository, apply the same block to sibling repositories in the same task.
- Keep repository-specific product, packaging, signing, route, architecture, and handoff details under `Repository-Specific Rules`, not in this section.

### General Working Style

- Prefer minimal, targeted changes over broad refactors.
- Preserve existing product copy unless the task requires rewriting it.
- Keep user-facing docs concise and practical; avoid adding AI collaboration notes or marketing filler unless explicitly requested.
- Keep public root `README.md` files user-facing and polished: lead with value, use concise feature/usage framing, include externally useful examples, and avoid internal progress notes, AI handoff notes, operational constraints, or release-process guidance.
- Prefer the project-family README pattern when applicable: centered logo/title/value summary, badges and primary links, a `---` divider, screenshot preview, capability breakdown, quick start, usage/integration examples, feature details, local development, security reporting, license, Star History, and footer credit.
- Keep README structure user-journey oriented: what it is, why it matters, what it can do, how to start, how to use or integrate it, then contributor-facing local development notes.
- Keep README prose tight. Group capabilities by user-facing surface or scenario, use concrete statements and copyable minimal examples, and avoid repeating the same capability unless new context is added.
- README license text, license badge, root `LICENSE`, package metadata, and build metadata must agree; update them together or explicitly call out intentional differences.
- Contributor rules, AI guidance, handoff notes, release conventions, missing-work notes, and local helper commands belong in `AGENTS.md`, not public docs. Do not create `docs/`, `notes/`, `tmp/`, or similar directories just to store that material unless the user asks.
- In public docs, write commands with standard upstream tooling rather than local wrappers, aliases, shell functions, or private helper commands.
- For searches, prefer `rg`.
- Use `apply_patch` for manual edits when the environment is stable.
- Do not run destructive git commands unless explicitly requested.

### Validation And Hygiene

- Keep the working tree clean before handoff: do not leave local build outputs, dependency caches, debugging screenshots, or temporary troubleshooting files committed or untracked.
- When the environment lacks a required toolchain and the user does not need full local verification, skip heavy verification only when necessary and say so explicitly.
- Release notes are user-facing change logs. Do not include internal verification/process statements such as having run tests, builds, audits, or CI checks unless explicitly requested.
- When repository structure, commands, external capabilities, release process, or recurring engineering pitfalls change, update `AGENTS.md` in the same task. Keeping this file current is required, not optional.
- If newly learned guidance appears reusable across repositories, ask whether to scan sibling `AGENTS.md` files, apply the shared rule, and push those updates.
- For GitHub-hosted repositories, maintain the baseline repository-governance files consistently across projects unless the user explicitly asks for divergence. This baseline includes `LICENSE`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, issue templates, and similar repo-health/community files.
- "Consistently" does not mean every line must be identical. Keep the structure, tone, and policy baseline aligned, but make the necessary project-specific substitutions for repository name, product name, links, version fields, platform fields, security scope, issue-form fields, and other repo-specific facts.
- If one of those GitHub governance files changes in a way that should become the new shared baseline, ask whether to propagate it across sibling GitHub repositories and push the updates while preserving required project-specific substitutions.

### Security And Review

- Review code with a bug-risk mindset first. Prioritize functional regressions, security issues, breaking changes, and missing tests before style or cleanup suggestions.
- If code returns `text/html` built from server-side string templates, HTML-escape all text fields from settings, persisted data, and user-controlled input before interpolating them into tags such as `<title>`, headings, attributes, or inline scripts.
- Do not assume only frontend `innerHTML` paths are XSS-relevant; also inspect backend-rendered HTML, email templates, CMS fragments, and raw string formatting that bypasses auto-escaping.
- For admin permission checks, prefer no-side-effect probes against real resources.
- Do not use invalid create requests to probe permissions; validation failures can mask the real authorization result and create misleading server logs.

### Dependency And Upgrade Rules

- Do not merge dependency or toolchain bumps just to clear security alerts or Dependabot PRs. First confirm the repository config is compatible and all required CI/build/test steps stay green.
- Treat build-tool upgrades such as `vite`, bundlers, editors, framework compilers, and test runners as compatibility work, not routine version bumps. If the upgrade breaks the build, defer it or patch it properly instead of merging a red PR.
- When a security alert applies only to dev tooling or an unused runtime mode, verify real exposure before escalating. Distinguish "reported in the dependency graph" from "actually exploitable in this repository."

### Release Rules

- Rewrite stable release notes from the commits actually included by the published tag. Do not mix in changes that landed only on `main` after that tag.
- When converting prereleases into a stable release, aggregate the effective user-visible changes across the prerelease cycle instead of copying beta notes verbatim.
- If replacing or deleting an older release in favor of a newer one, compare the old tag, the new tag, and the default branch separately so unreleased work is not accidentally documented.
- Do not promote a prerelease to a stable `vX.Y.Z` release unless the user explicitly asks for that exact stable release.
- GitHub release titles should default to the bare tag name such as `v0.1.0` or `v0.1.0-beta.1`, not `ProjectName v0.1.0`, unless the user explicitly asks for a product-prefixed title.
- Treat release signing assets as product-critical state. Before generating or replacing a mobile/desktop release signing key, ask the user for all identity fields the tool will embed, alias/key naming, password policy, storage location, and whether the key is intended for long-term upgrades.
- Do not treat a successful release-mode build as proof that an artifact is properly signed. Verify the final artifact with the platform verifier whenever one exists, such as `apksigner verify --print-certs` for Android APKs.
- Never commit private signing material, keystores, provisioning profiles, passwords, or generated local signing property files. Commit only non-sensitive examples or documentation, and ensure `.gitignore` covers the real local files before generating them.
- If a release signing key is lost or replaced, existing users may lose the normal upgrade path. Surface that risk explicitly before changing keys.
- Keep release architecture/package allowlists explicit. When the allowed architectures change, update every related build surface in the same task: native build config, packaging/copy scripts, package-manager scripts, release docs, and generated artifact cleanup.
- Windows release artifact architecture labels must use `amd64` for 64-bit Intel/AMD builds, not `x64`; keep required toolchain/platform identifiers unchanged, such as Rust target triples (`x86_64-pc-windows-msvc`), Android ABIs (`x86_64`), and npm platform package names (`win32-x64`).
- After changing release architecture/package rules, scan for removed architecture names and delete stale artifacts from local release output directories before handoff.

## Repository-Specific Rules

This repository is `start-your-python`, a desktop learning app for Python beginners.

## Project Summary

- Product goal: provide a local beginner-friendly Python learning workspace, not a full IDE.
- UX direction: PyCharm-style workspace with course tree, editor, lesson content, and real terminal execution.
- Main audience: Chinese-speaking Python beginners.
- Current app version: `1.5.0`

## Tech Stack

- Frontend: `Vue 3`
- Build tool: `Vite`
- State: `Pinia`
- Desktop shell: `Tauri`
- Mobile shell: `Capacitor`
- Testing: `Vitest`
- Language: `TypeScript`

## Important Paths

- App entry and UI source: `src/`
- Tauri shell: `src-tauri/`
- Course content: `lessons/`
- Course images: `public/course-images/`
- Web-visible static assets stay in `public/`; icon generation source assets stay in `resources/`.
- Tests: `tests/`
- Scripts: `scripts/`
- User docs: `README.md`

## README Rules

- Keep the README aligned with the Halo plugin family style:
  - centered logo/title/value summary
  - centered Release, License, and CI badges with color parameters
  - centered primary links
  - `---` divider after the hero block
  - emoji section headings
  - screenshot preview
  - GPL-3.0 license section
  - Star History chart
  - centered `Built with ❤️ by Sunny` footer
- The README license text must match the repository `LICENSE` file and package metadata.
- If the README structure changes, preserve the same public-facing order used by the Halo plugin family unless the user explicitly requests a different format.

## Product Constraints

- This app is not a cloud IDE.
- Do not introduce account systems, cloud sync, or AI chat unless explicitly requested.
- Prefer preserving the current local learning tool framing.
- Preserve Chinese product copy unless the task is specifically about rewriting copy.
- Course content is sourced from real `.py` files under `lessons/`.

## Runtime Model

- The app can render lesson content and code examples from lesson files.
- Code blocks can invoke the system Python interpreter in the desktop Tauri app.
- Terminal behavior should support:
  - stdout
  - stderr
  - `input()` style interaction
- Learning progress should update from explicit step actions, correct quiz answers, and successful lesson-step code runs that finish with exit code 0.
- Free editor runs should not update lesson progress.
- Python runtime is external to the app. The app does not bundle Python.
- Mobile mode is a reader-oriented course experience and does not depend on system Python.

## Development Commands

- Install dependencies:
  - `npm install`
- Web development:
  - `npm run dev`
- Type check:
  - `npm run typecheck`
- Tests:
  - `npm run test`
- Web build:
- `npm run build`
- Mobile web build:
  - `npm run build:mobile`
- Do not run `npm run build` and `npm run build:mobile` concurrently. Both commands write to the same `build/` output directory and can fail with Vite `ENOTEMPTY` cleanup errors when run in parallel.
- GitHub Actions workflows set `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24=true` to suppress Node 20 deprecation warnings from JavaScript-based actions; keep that flag on when editing CI/CD workflows unless GitHub changes the guidance.
- Tauri development:
  - `npm run tauri:dev`
- Tauri build:
  - `npm run tauri:build`
- Release packaging:
  - `npm run tauri:release`

## Windows Build Notes

- Tauri work requires Rust toolchain.
- Windows desktop build also requires Visual Studio Build Tools with C++ components.
- Existing npm scripts already append Cargo bin path before Tauri commands. Preserve that behavior.

## Codebase Notes

- Workspace-related UI is under `src/components/workspace/`.
- Content parsing and loading are under `src/services/content/`.
- Python execution and environment detection are under `src/services/runtime/`.
- Learning progress, recent study state, and terminal state are under `src/stores/`.
- Markdown lesson content is rendered through `src/components/content/MarkdownContent.vue`; keep `src/utils/sanitizeHtml.ts` in place before any `v-html` rendering.
- Keep changes targeted. Avoid broad UI rewrites unless requested.
- Icon source of truth: keep `resources/icon.png` as the canonical input for both Tauri and Capacitor. Use `npm run icons:desktop` for `src-tauri/icons/*`, `npm run icons:android` after `npx cap add android`, and `npm run icons:generate` to refresh both from the same source. Preserve the full source image composition; do not crop or redraw it when updating app icons.

## Repository Release Conventions

- Version tags follow this style:
  - stable: `v1.3.0`
  - prerelease example: `v1.2.1-beta.1`
- When releasing:
  - update `package.json`
  - update `src-tauri/Cargo.toml`
  - update `src-tauri/tauri.conf.json`
  - keep `README.md` current if packaging targets, screenshots, runtime behavior, or capability descriptions change
- GitHub release titles should follow the reusable rule and use bare tag names.
- Release notes should use the heading `## 更新内容`.
- Release history is maintained in GitHub Releases. Do not add a committed `CHANGELOG.md` unless explicitly requested.
- GitHub Actions CI is defined in `.github/workflows/ci.yaml` and runs on pushes and pull requests to `main`.
- GitHub Actions CD is defined in `.github/workflows/cd.yaml` and runs on the `Release published` event.
- CD uploads Windows installer/bundle outputs, portable Windows zips, and signed Android release APKs to the GitHub Release.
- Windows portable zip filenames must include version and platform, for example `StartYourPython-v1.3.0-win-amd64.zip`.
- Windows CD publishes only the amd64 and arm64 portable zips:
  - `StartYourPython-v1.3.0-win-amd64.zip`
  - `StartYourPython-v1.3.0-win-arm64.zip`
- Windows portable zip contents must be rooted directly at `Start Your Python.exe` plus `lessons/`; do not wrap them in an extra top-level folder.
- The executable inside the Windows portable zip must keep the fixed product name `Start Your Python.exe`, without a version in the executable filename.
- Android APK filenames must include the version tag and ABI, for example `StartYourPython-v1.3.0-android-arm64-v8a-release.apk`.
- Android CD builds separate signed release APKs only for `arm64-v8a` and `x86_64` instead of a debug APK or universal APK.
- Android release signing in CD requires GitHub Secrets named `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, and `ANDROID_KEY_PASSWORD`.
- Android package name is `com.sunny.startyourpython`.
- The Android release keystore is kept in the project-local ignored `release-signing/` directory and must not be committed. Keep this private local backup plus the matching GitHub Actions secrets so future APK updates can use the same signing identity.

---
> Source: [sunnyhmz7010/start-your-python](https://github.com/sunnyhmz7010/start-your-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
