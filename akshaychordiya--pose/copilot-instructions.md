## pose

> Guidance for AI agents (Claude Code, Cursor, Codex, etc.) working on the Pose codebase. Read this before you touch anything.

# AGENTS.md

Guidance for AI agents (Claude Code, Cursor, Codex, etc.) working on the Pose codebase. Read this before you touch anything.

## What Pose is

A KSP2 processor that auto-generates Jetpack Compose `@Preview` functions from `@Composable` source at build time. Deterministic, offline, never modifies user source. Ships with a companion IntelliJ plugin for editor-side gutter navigation.

Full user-facing docs: [`README.md`](README.md). This file is for people (and agents) *working on* Pose.

## Repository layout

| Path                 | What lives here                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `annotations/`       | Pure Kotlin JVM annotations JAR - `@Pose`, `@PoseProvider`, `@PoseIgnore`. Zero Compose deps by design. Published to Central.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `processor/`         | The KSP2 symbol processor. Reads `@Pose`, emits `<Source>__Preview.kt` files. Also published to Central.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `intellij-plugin/`   | The Marketplace plugin. Adds gutter icons in the IDE that jump from source composables to their generated previews.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `sample-app/`        | End-to-end smoke test. Every capability of Pose is exercised in one file each ([LoginContent](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/LoginContent.kt), [HomeContent](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/HomeContent.kt), [ProductCard](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/ProductCard.kt), [ArticleCard](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/ArticleCard.kt), [WellKnownTypesCard](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/WellKnownTypesCard.kt), [SelfThemedBanner](sample-app/src/main/kotlin/io/github/akshaychordiya/pose/sample/SelfThemedBanner.kt)). If a change plausibly affects generation, verify it here too. |
| `docs/refusals.md`   | Refusal-code catalogue keyed by `PG-xxx`. Every diagnostic message the processor emits links here - keep in sync.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `docs/images/`       | Screenshots for the README and Marketplace listing.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `.github/workflows/` | `ci.yml` runs on every push/PR. `release.yml` fires on `v*` tags (Central). `plugin-release.yml` fires on `plugin-v*` tags or manual dispatch (Marketplace).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `.run/`              | Shared IntelliJ run configurations. Add here rather than in `.idea/` so contributors see them too.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |

| `.githooks/` | `pre-push` runs full verification before any push. Needs one-off activation — see First-time setup below. |
| `CHANGELOG.md` | Keep a Changelog format. The plugin and the KSP artifacts version independently. |
| `PUBLISHING.md` | Release scaffolding for both cadences — Sonatype Central (KSP) and JetBrains Marketplace (plugin). |

## Where each concern lives (processor)

| Concern                                                                                | File                      |
|----------------------------------------------------------------------------------------|---------------------------|
| Drive the pipeline, filter symbols, bulk opt-in                                        | `PoseProcessor`           |
| Validate signature (visibility, receivers, generics, refuse-category params)           | `SignatureChecker`        |
| Decide emission shape (inline / companion / sealed fan-out) + match `@Pose(providers)` | `PreviewPlanner`          |
| Recursive structural synthesis of values                                               | `SampleResolver`          |
| Well-known FQN table (Compose value classes, Flow, java.time, refuse list)             | `FqnTable`                |
| Function-type lambda placeholders (`() -> Unit`, scoped composable lambdas)            | `FunctionTypeSynthesizer` |
| KotlinPoet emission + wrapper layers (inspection mode / preview wrapper / theme)       | `PreviewFileEmitter`      |
| KSP-arg parsing                                                                        | `Options`                 |
| Diagnostic codes + `strict` routing                                                    | `Diagnostics`             |

## First-time setup

Activate the shared pre-push hook (one-off, per clone - git does not do this automatically):

```bash
git config core.hooksPath .githooks
```

Every `git push` then runs full verification first and aborts on failure. Emergency bypass: `git push --no-verify` (CI still gates `main`).

## Build & test - the important gradle tasks

The one you want most of the time is **full verification** - same command the pre-push hook runs, and available in the IDE as `Run → Verify Everything`:

```bash
./gradlew build :intellij-plugin:buildPlugin
```

Everything else is a subset for tighter feedback loops:

```bash
# Fast-loop test cycle (all processor tests)
./gradlew :processor:test

# Verify sample-app builds and previews generate
./gradlew :sample-app:assembleDebug
# then inspect: sample-app/build/generated/ksp/debug/kotlin/io/github/akshaychordiya/pose/sample/

# Rebuild the IntelliJ plugin zip
./gradlew :intellij-plugin:buildPlugin
# zip lands at: intellij-plugin/build/distributions/intellij-plugin-<version>.zip

# Plugin unit tests (light IntelliJ platform tests)
./gradlew :intellij-plugin:test

# Verify plugin against multiple IDE builds - slow (downloads distributions), run before Marketplace publish
./gradlew :intellij-plugin:verifyPlugin

# Publish annotations + processor to ~/.m2 for local Marshmallow-style testing
./gradlew :annotations:publishToMavenLocal :processor:publishToMavenLocal
```

## Common tasks

### Add an FQN emitter (make Pose synthesize a value for a new type)

1. Extend `FqnTable.SimpleEmitters` (fixed literal) or `FqnTable.GenericEmitters` (recursive over a type argument like `Flow<T>`).
2. Add a coverage test to `FqnTableTest.kt`. Follow the existing pattern - declare the type in a stub `SourceFile.kotlin`, compile via `CompileHarness`, assert the generated file contains the expected literal.
3. If the type only exists in a specific artifact, the test's stub covers you locally, but real users will need that artifact on their classpath - no action needed on our side.

### Add a refusal category (a new "this type can't be faked")

1. Extend `FqnTable.RefuseFqns` with the FQN → short category name.
2. Add a case to `RefusalTest.kt` - compile a composable that takes this type, assert the `PG-xxx` code fires.
3. If it's a new PG-code (usually PG003 covers refuse-category types), also update [docs/refusals.md](docs/refusals.md) with a section per code and rewrite pattern.

### Add a KSP option

1. Extend `Options` data class + `Options.from()` parser with a default value.
2. Cover in `OptionsTest.kt`.
3. Wire the option into whatever site consumes it (usually `PoseProcessor` or `PreviewPlanner`).
4. Document in the README's KSP options table.

### Add a sample-app composable that showcases a new feature

Follow the existing pattern in `sample-app/src/main/kotlin/…/sample/` - one composable per file with a `Showcase - <what this demonstrates>` comment at the top. Keep each file small and focused on one axis.

## Conventions

- **Deterministic output.** Sample values are seeded from `(composable FQN, parameter path, type FQN)` hashes. **Never `Random`, never clock, never classpath-iteration-order.** If you find yourself reaching for one of those, you're on the wrong path.
- **`explicitApi()` on both `annotations` and `processor`.** Declare visibility on every public symbol.
- **No comments on obvious code.** Comment WHY (a hidden constraint, a workaround for a specific KSP quirk), not WHAT. Descriptive names carry the load.
- **KSP can only emit new files, not modify existing ones.** Never propose "we could just modify the user's source" - that's outside KSP's contract. The IntelliJ plugin is the escape hatch for that whole class of UX work.
- **One file per source `KSFile`.** All previews for composables declared in `Foo.kt` land in `Foo__Preview.kt`. `PreviewFileEmitter` groups plans by containing file.

## Testing conventions

Tests use [`dev.zacsweers.kctfork`](https://github.com/tschuchortdev/kotlin-compile-testing) (a KSP2-aware fork). Every processor test:

1. Constructs one or more `SourceFile.kotlin` fixtures with a small composable + any stubs it needs.
2. Runs through `CompileHarness.compile(...)`, optionally with a KSP `options` map.
3. Asserts on either the generated file contents (`result.generatedFile("Foo__Preview.kt").readText()`) or the compiler messages (`result.messages` - for refusals).

Don't mock KSP internals - `CompileHarness` gives you real behaviour with a light in-memory compilation. Plugin tests use `BasePlatformTestCase` (light IntelliJ project) with `myFixture.tempDirFixture` to lay out mock file trees.

## Release process

**Two independent cadences.** Do not conflate them.

| What changed             | Tag                                 | Workflow             | Publishes to          |
|--------------------------|-------------------------------------|----------------------|-----------------------|
| Annotations or processor | `v0.5.0`, `v0.5.1`, …               | `release.yml`        | Maven Central         |
| IntelliJ plugin          | `plugin-v0.5.0`, `plugin-v0.5.1`, … | `plugin-release.yml` | JetBrains Marketplace |

Steps:

1. Bump root `build.gradle.kts` version (drop `-SNAPSHOT` if present).
2. Update `CHANGELOG.md` with a new `## [<version>]` section. Follow Keep a Changelog conventions.
3. If plugin changed, add a `<h3><version></h3>` block at the TOP of `intellijPlatform.pluginConfiguration.changeNotes` in `intellij-plugin/build.gradle.kts`. Historical entries stay below.
4. Commit with `Bump to <version>: <what changed>` style message.
5. Tag with the appropriate prefix and push:
   ```
   git tag v0.4.2         # KSP artifacts
   git tag plugin-v0.4.2  # IntelliJ plugin
   git push origin --tags
   ```

Publishing scaffolding (POM metadata, GPG signing, Sonatype credentials, Marketplace token) is fully documented in [`PUBLISHING.md`](PUBLISHING.md). Read that before touching release workflows.

## Things NOT to do

- ❌ Modify a user's source file. KSP only emits into `build/generated/`. There is no "just tweak the composable" escape.
- ❌ Publish `-SNAPSHOT` versions to Central. Central rejects them; they belong in the snapshots repo (or nowhere for now).
- ❌ Bypass `pose.strict` as a workaround for a broken emitter. If Pose refuses a type, either add a proper FQN entry or refuse it correctly with a helpful message.
- ❌ Reintroduce a `pose.variants` knob. Variant selection is the consumer's job via `kspDebug` / `kspRelease` / `kspCommonMainMetadata` etc.
- ❌ Add new dependencies to the `annotations` module. It's intentionally a zero-dep pure Kotlin JVM JAR so consumers don't accidentally pull in Compose tooling just by depending on the marker.

## When you're not sure

- The user's REAL goal is a preview showing up in Studio's preview panel next to their composable. Every design decision should trace back to that.
- If a change might affect generation, run the sample-app build and eyeball a generated file before claiming victory.
- If a change might affect the plugin, install the fresh `intellij-plugin-<version>.zip` in a real Android Studio via `Settings → Plugins → Install from Disk` and verify manually. `BasePlatformTestCase` catches most regressions but not visual ones.

---
> Source: [AkshayChordiya/Pose](https://github.com/AkshayChordiya/Pose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
