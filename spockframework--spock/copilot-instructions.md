## spock

> - The `*.yaml` workflow files are generated from the `*.main.kts` Kotlin DSL files, **never** edit the YAML directly.

- The `*.yaml` workflow files are generated from the `*.main.kts` Kotlin DSL files, **never** edit the YAML directly.
- After changing a `*.main.kts` file, regenerate the YAML with `./gradlew preprocessWorkflows`, then commit both; CI fails if the committed YAML is out of sync. See `README.adoc` for details.

---
> Source: [spockframework/spock](https://github.com/spockframework/spock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
