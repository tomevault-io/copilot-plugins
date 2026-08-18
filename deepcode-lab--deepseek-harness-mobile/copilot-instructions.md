## deepseek-harness-mobile

> CI runs `./gradlew assembleDebug lintDebug ktlintCheck testDebugUnitTest` and

# AGENTS.md

## Quality gate

CI runs `./gradlew assembleDebug lintDebug ktlintCheck testDebugUnitTest` and
`./tests/run-local.sh` on every push to main and every PR — failing means
blocked. After changing Kotlin code, match the gate before claiming done:

- `./gradlew ktlintCheck` — style (android ruleset via `.editorconfig`);
  auto-fix with `./gradlew ktlintFormat`
- `./gradlew lintDebug` — Android lint errors block (`abortOnError`)
- `./gradlew testDebugUnitTest` — JVM unit tests (JUnit4 + Robolectric + mockk)
- `./tests/run-local.sh` — JS polyfill tests (node)

## Build

- Requires JDK 17+ and Android SDK; the runtime rootfs
  `app/src/main/assets/rootfs.tar.xz` must exist (CI downloads it from the
  dsh-io/dsh-arm64 release)
- `assembleRelease` is CI-only — the signing keystore lives in the repo
  secret, never run it locally

## Runtime model

Single embedded Debian rootfs (built in dsh-io/dsh-arm64): the engine runs
inside the rootfs via proot, and the agent's bash is the same container's
bash. No Termux snapshot, no exec hooks, no wrapper scripts, no downloads on
first run. See `docs/design.md`.

---
> Source: [deepcode-lab/deepseek-harness-mobile](https://github.com/deepcode-lab/deepseek-harness-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
