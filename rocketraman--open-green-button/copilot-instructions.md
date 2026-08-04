## open-green-button

> This repo's toolchain (JDKs, Gradle, Node, etc.) is managed by **mise** — pinned in

# Open Green Button — agent guide

## Toolchains: use and update mise (IMPORTANT)

This repo's toolchain (JDKs, Gradle, Node, etc.) is managed by **mise** — pinned in
[`mise.toml`](mise.toml). Always go through mise; do not assume the ambient `java`/`gradle` on
`PATH` is correct (the host may default to JDK 25, which breaks Gradle 8.14).

- **Run build tools through mise**: `mise exec -- ./gradlew …` (or rely on mise being active in
  the shell). The bare shell's `java` may be the wrong JDK.
- **When you add, remove, or change a tool or version — update `mise.toml`** (and run
  `mise install`). Never hardcode a tool path in scripts or docs when mise can provide it; wire it
  through `mise.toml` (`[tools]` for versions, `[env]` for derived paths) so it's automatic for
  everyone. Keep any version that appears in more than one place (e.g. a tool version and a path
  in `[env]`) in sync, and update the explanatory comments in `mise.toml` too.
- After editing `mise.toml`, verify with `mise install` + `mise env` (and a real build) before
  considering it done.

### Current JDK setup (and why)
- `java = ["temurin-21", "graalvm-community-21.0.2"]` — temurin-21 is the **default** `java` and
  the Gradle daemon JDK (matches CI; Gradle 8.14 can't run on JDK 25). GraalVM CE 21 is installed
  alongside it solely to provide `native-image` + the tracing agent.
- `GRAALVM_HOME` (set in `mise.toml` `[env]`) points the GraalVM Native Build Tools Gradle plugin
  at that GraalVM install, so `:app:nativeCompile` and `scripts/generate-native-metadata.sh` work
  with no manual `JAVA_HOME` juggling.

## Build & native image
- App lives under [`server/`](server/) (Kotlin, Ktor CIO, Gradle). Normal build: `./gradlew build`.
- The production deploy artifact is a **GraalVM native image** (Fly.io scale-to-zero). See
  [`Dockerfile`](Dockerfile), [`docs/deployment.md`](docs/deployment.md), and the native-image
  reachability metadata under
  `server/app/src/main/resources/META-INF/native-image/org.opengb/` (regenerate with
  `scripts/generate-native-metadata.sh`; `manual/` there is hand-maintained — read its README).
- A missing metadata entry only fails at **runtime**, so after regenerating, rebuild and confirm
  the binary/container actually boots and serves before committing.

---
> Source: [rocketraman/open-green-button](https://github.com/rocketraman/open-green-button) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
