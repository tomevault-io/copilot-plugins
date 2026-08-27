## mood

> Run the formatter before testing:

## Verification

### Linting

Run the formatter before testing:

```shell
./gradlew fmt
```

### Testing

Run the checks relevant to the changes:

- Kotlin changes: `./gradlew test`
- Android-specific changes: `./gradlew :android:assembleDebug`
- Web-specific changes: `./gradlew :web:wasmJsBrowserDevelopmentDistribution`

---
> Source: [CharlieTap/mood](https://github.com/CharlieTap/mood) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
