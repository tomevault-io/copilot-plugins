## drivesense

> - Use JDK 17 and Android SDK Platform 34.

# Codex and automation rules

- Use JDK 17 and Android SDK Platform 34.
- Never commit or synthesize `local.properties`, credentials, signing files, downloaded models, APK/AAB files, IDE state, or build outputs.
- Run `./gradlew prepareModels` explicitly once when the model is absent, then `./gradlew verifyModels`.
- `preBuild` must depend on `verifyModels` and must never depend on `prepareModels` or download files.
- Preserve the package/application ID `io.github.anup42.drivesense` and pinned model SHA-256 unless the task explicitly changes the public contract.
- Before proposing a code change, run `./gradlew testDebugUnitTest lintDebug assembleDebug`.
- Add pure JVM tests for thresholds, timing, voting, rotation, and transition logic. Record physical-device validation separately from build success.
- Keep camera frames on-device and unrecorded. Update `PRIVACY.md`, `SAFETY.md`, and third-party notices when behavior or dependencies change.
- Treat drowsiness and road-object output as experimental; never add unsupported medical, safety, or driver-assistance claims.

---
> Source: [anup42/DriveSense](https://github.com/anup42/DriveSense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
