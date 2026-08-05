## fabric-loom

> - Do not run the full Gradle build or the entire test suite at once; they take many hours in this repository.

- Do not run the full Gradle build or the entire test suite at once; they take many hours in this repository.
- Prefer targeted Gradle tasks that validate only the area you changed.
- When verifying final changes, run `./gradlew build -x test`.

---
> Source: [FabricMC/fabric-loom](https://github.com/FabricMC/fabric-loom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
