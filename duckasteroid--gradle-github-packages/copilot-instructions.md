## gradle-github-packages

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A Gradle plugin (published to the Gradle Plugin Portal as `io.github.duckasteroid.github-packages` and `io.github.duckasteroid.github-packages-settings`) that configures GitHub Packages as an authenticated Maven repository, in both build scripts and `settings.gradle`. Java 17, built with Gradle itself (`java-gradle-plugin`).

## Common commands

```bash
./gradlew clean check          # full build: unit tests + functional tests
./gradlew test                 # unit tests only (src/test)
./gradlew functionalTest       # functional tests only (src/functionalTest, uses Gradle TestKit)
./gradlew test --tests "GithubPackagesPluginTest.pluginRegistersExtension"
./gradlew functionalTest --tests "GithubPackagesPluginFunctionalTest.pluginAppliesAndAddsRepository"
```

There is no separate lint/format task configured. `check` depends on `functionalTest` (wired in `build.gradle`), so a single `./gradlew check` covers both suites — this is what CI (`.github/workflows/build.yml`) runs.

Prefer the `gradle-mcp` MCP tool over invoking `./gradlew` through a raw shell command when it's available — it gives structured pass/fail output instead of raw logs to grep through.

Versioning is derived from git tags via `duckasteroid-java` (from [duckAsteroid/gradle-convention-plugin](https://github.com/duckAsteroid/gradle-convention-plugin), applied in `build.gradle`) — there is no hardcoded version to bump. Between releases, the dev-build version is computed from Conventional Commits since the last tag (`feat:`→minor, `fix:`/`perf:`→patch, `!`/`BREAKING CHANGE:`→major); an explicit `-Prelease.version=X.Y.Z` (below) always overrides that. Publishing (`publish`, `publishPlugins`) only happens in CI on `v*` tags pushed to `main` (`.github/workflows/publish.yml`); don't run publish tasks locally.

Plugin resolution for `duckasteroid-java` itself is bootstrapped in `settings.gradle` via `io.github.duckasteroid.github-packages-settings` (this project's own settings plugin, pinned to a previously published Plugin Portal version) pointed at `owner=duckAsteroid, repository=gradle-convention-plugin` — `duckasteroid-java` is published only to GitHub Packages, never the Plugin Portal, so it can't be resolved without that bootstrap. The Java toolchain default from `duckasteroid-java` is Java 25; this project pins back to 17 via `duckasteroid.java.version=17` in `gradle.properties` to match the existing target. `duckasteroid-java` configures no publish repository itself (not even GitHub Packages) — this project's actual publish targets (Gradle Plugin Portal via `com.gradle.plugin-publish`, plus `maven-publish`) are untouched and independent of it.

### Releasing

Cutting a release means creating and pushing a `vX.Y.Z` tag on `main`, which triggers `publish.yml` (build, test, `publish`, `publishPlugins` to the Gradle Plugin Portal). Do this with axion-release's own `release` task rather than a manual `git tag` — it verifies the working tree is clean before tagging, and pushes the tag for you:

```bash
./gradlew release -Prelease.version=X.Y.Z
```

Without `-Prelease.version`, axion-release defaults to bumping the patch segment of the last tag. This project doesn't follow strict SemVer discipline for pre-1.0 releases (some past patch bumps included what would technically be minor features) — confirm the intended version number with the user rather than assuming either convention. The `release` task fails fast if there are uncommitted changes, so `git status` clean first.

## Architecture

Three source sets: `src/main` (plugin code), `src/test` (fast unit tests using `ProjectBuilder`), `src/functionalTest` (black-box tests using `GradleRunner`/TestKit that write real `build.gradle`/`settings.gradle` files to a temp dir and assert on build output — this is where most behavior is actually verified, since the plugins' effects are only observable through Gradle's repository/model APIs).

Two plugin entry points share the same credential/repository machinery:

- **`GithubPackagesPlugin`** (`io.github.duckasteroid.github-packages`) — applies to a *project* build script. Registers the `githubPackages { owner; repository }` extension and, in `project.afterEvaluate`, adds a Maven repo to `project.repositories` and (if `maven-publish` is applied) to `publishing.repositories`.
- **`GithubPackagesSettingsPlugin`** (`io.github.duckasteroid.github-packages-settings`) — applies to `settings.gradle`. Registers the same `githubPackages` extension on `Settings`, and in `gradle.settingsEvaluated` adds the repo to both `pluginManagement.repositories` and `dependencyResolutionManagement.repositories`.

Both plugins also inject a `gitHubPackages { owner; repo; ... }` closure/method (via `ExtraPropertiesExtension`, key `"gitHubPackages"`) that can be called *inside* a `repositories { }` block — this is `GithubPackagesRepositoryDsl`, which supports both a Groovy `Closure` and a Gradle `Action` entry point and works against any `RepositoryHandler` (project repos, publishing repos, or pluginManagement repos). Note the naming split: the block-scoped DSL uses `owner`/`repo`, while the `githubPackages` extension uses `owner`/`repository` — these are two different config surfaces, not a typo.

Gradle's `pluginManagement { }` block must be the first statement in `settings.gradle`, so `GithubPackagesSettingsPlugin`'s injected `gitHubPackages` DSL method cannot be used inside `pluginManagement.repositories { }` in the *same* settings file where the settings plugin itself is applied via `plugins { }` (see the functional test `pluginManagementDslCannotUseSettingsPluginDefinedMethodsInSameSettingsFile`, which asserts this fails). The documented workaround is to configure the top-level `githubPackages { owner; repository }` extension instead, which the settings plugin applies to `pluginManagement.repositories` itself after settings evaluation.

Credential resolution (`CredentialProviders`, keys centralized in `CredentialKeys`) is a three-tier `Provider.orElse()` chain, same for username and token, first available value wins:

1. Gradle property `gpr.user` / `gpr.key` (from `gradle.properties`, project or `~/.gradle`) — or, when an optional `profile` is set on the `githubPackages`/`gitHubPackages` block, `gpr.<profile>.user` / `gpr.<profile>.key` instead. Deliberately does *not* fall back to the unqualified `gpr.user`/`gpr.key` when a profile is set and unmatched, so a typo'd profile name can't silently leak the wrong identity — it falls through to tier 2 instead.
2. Env var `GH_PACKAGES_READ_USER` / `GH_PACKAGES_READ_TOKEN` (read-only org-wide credentials)
3. Env var `GITHUB_ACTOR` / `GITHUB_TOKEN` (GitHub Actions default)

This chain is the source of truth for precedence — reflected in `README.md` and in `CredentialProviders`.

Laziness matters here: `GithubPackagesExtension`'s username/token conventions must be wired with `Provider.orElse()` (lazy), not `.getOrElse()` (eager), because `profile` may be set later in the same `githubPackages { }` block, after the extension is constructed. Similarly, `GithubPackagesRepositoryDsl.call()` runs the user's closure *before* filling in credential defaults, so `owner`/`repo`/`profile`/explicit `username`/`token` are all known first — defaults only fill in whichever of username/token the closure left `null`.

`GithubPackagesExtension` is shared by both plugins (same class, applied to either `Project` or `Settings` extensions) and computes `mavenUrl()` as `https://maven.pkg.github.com/{owner}/{repository}`; explicit `username`/`token` set on the extension or DSL spec override the resolved-credential convention.

---
> Source: [duckAsteroid/gradle-github-packages](https://github.com/duckAsteroid/gradle-github-packages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
