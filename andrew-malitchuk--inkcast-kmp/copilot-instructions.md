## inkcast-kmp

> User onboarding flow for initial app setup and introduction.

# CLAUDE.md — presentation-feature-onboarding

## Purpose

User onboarding flow for initial app setup and introduction.

## Convention Plugins

- `dev.yamh.io.convention.feature`
- `dev.yamh.io.convention.di`

## Key Files

| File | Purpose |
|------|---------|
| `OnboardingScreen` | Entry point composable |
| `OnboardingViewModel` | Orbit MVI ViewModel |
| `OnboardingContract` | State / SideEffect / Intent definitions |

## Module Dependencies

- `domain-usecase-api`
- `presentation-core-localisation`
- `presentation-core-styling`
- `presentation-core-ui`
- `presentation-core-navigation-api`

## Build

```shell
./gradlew :presentation-feature-onboarding:build
```

---
> Source: [andrew-malitchuk/inkcast-kmp](https://github.com/andrew-malitchuk/inkcast-kmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
