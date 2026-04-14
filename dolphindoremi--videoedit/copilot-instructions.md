## videoedit

> "description": "Never rewrite test invariants; fix implementation instead.",

{
  "version": 1,
  "rules": [
    {
      "description": "Never rewrite test invariants; fix implementation instead.",
      "alwaysApply": true,
      "globs": ["app/src/test/**", "app/src/androidTest/**"],
      "editsAllowed": false
    },
    {
      "description": "When touching WhisperEngine, WhisperBridge, AsrFileScanWorker, or AsrTranscribeWorker, run instrumented tests.",
      "alwaysApply": true,
      "commands": ["./gradlew :app:connectedDebugAndroidTest"]
    }
  ]
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dolphinDoReMi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
