## myguava-business-payment-sdk-android

> - Use **2 spaces** for indentation (no tabs)


# Gradle Kotlin Scripts (.kts) Rules

## Code Style
- Use **2 spaces** for indentation (no tabs)
- Use modern Gradle API (version 8 or 9+)
- Write configuration cache compatible scripts using providers and lazy properties

## Configuration Management
- Public parameters come from [gradle.properties](mdc:gradle.properties)
- Private parameters come from [local.properties](mdc:local.properties) 
- Dependencies are declared in [gradle/libs.versions.toml](mdc:gradle/libs.versions.toml), not directly in build scripts

## Best Practices
- Use `Provider<T>` and `Property<T>` for configuration cache compatibility
- Prefer `val` over `var` for immutable configurations
- Use lazy initialization where appropriate
- Access version catalog via `libs` accessor

## Example Patterns
```kotlin
// Configuration cache compatible
val compileSdkVersion: Provider<Int> = provider { 
  libs.versions.compileSdk.get().toInt() 
}

// Using version catalog
implementation(libs.androidx.core.ktx)

// Provider-based configuration
android {
  compileSdk = compileSdkVersion.get()
}
```

## Dependency Management
- All dependencies referenced through version catalog: `libs.group.artifact`
- Version declarations only in `libs.versions.toml`
- Group related configurations using extension functions when needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GuavaPay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
