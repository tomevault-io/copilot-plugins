## clickerx

> - Use Android Studio (not Gradle CLI)

# AGENTS.md - ClickerX Developer Guide

## Build
- Use Android Studio (not Gradle CLI)
- Variants: mockDebug, mockRelease, prodDebug, prodRelease

## Structure
- `app/` - Main entry point
- `module_*/` - Feature modules
- `component_*/` - Business components (clickercore, script, base, user, ai)
- `lib_*` - Internal libraries (common, compiler, shell, network, clinj, a11y)

## Critical Constraints
- **luaj-jse-3.0.1.jar** in `component_clickercore/libs/` is CUSTOM - do NOT replace with Maven
- ARouter kapt configured in build.gradle files (AROUTER_MODULE_NAME arg)
- Many deps use `compileOnly` - builds work but IDE may not auto-resolve

## Key Config
- `gradle/libs.versions.toml` - version catalog
- `settings.gradle` - Aliyun mirrors, module list
- `app/build.gradle` - ENTRY_MAIN, SKIP_PERMISSION_CHECK buildConfig fields, flavors

---
> Source: [LittleFogCat/ClickerX](https://github.com/LittleFogCat/ClickerX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
