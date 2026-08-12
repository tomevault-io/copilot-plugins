## app-finanzas-movile

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Native **Android** app (Kotlin) — the mobile client of "Finanzas Independientes". It consumes a REST API (contract in `docs/API-CONTRACT.md`). Setup, run, and troubleshooting live in `README.md`; the target architecture spec is in `planeamiento/ARQUITECTURA.md`, and the phased roadmap in `planeamiento/PLAN.md`. Read those for anything not covered here.

## Build & test

Windows shell → use `gradlew.bat`. Unix → `./gradlew`.

```bash
gradlew.bat :app:assembleDebug        # compile debug APK — must stay green on every PR
gradlew.bat :app:installDebug         # install on connected device/emulator
gradlew.bat :app:testDebugUnitTest    # unit tests (domain use cases, ViewModels)
gradlew.bat clean
```

Run a single unit test:

```bash
gradlew.bat :app:testDebugUnitTest --tests "com.example.finanzas_independientes_app.domain.usecase.CalcularCuotaDiariaUseCaseTest"
```

Requires **JDK 17** (AGP 9). Toolchain versions are pinned in `gradle/libs.versions.toml` and dependencies use the `libs.*` version catalog — add deps there, not with hardcoded coordinates.

## UI stack — READ THIS FIRST

This project uses the **classic Android View system**, NOT Jetpack Compose. Concretely:

- `viewBinding = true`; screens are **Activities** (no Fragments, no Navigation Component) with **XML layouts** in `res/layout/`.
- Lists use **RecyclerView + Adapter** (see `TransaccionAdapter`, `CategoriaAdapter`, `HealthSignalAdapter`).
- Material Components (`com.google.android.material`) + ConstraintLayout. Colors/themes in `res/values/` and `res/values-night/` (dark mode supported).
- Charts via **Vico** (`com.patrykandpatrick.vico:views`, Cartesian only — line/column, no pie). In v3 the `views` AAR flattens everything under `com.patrykandpatrick.vico.views.*` (there is no `.core` package); solid fills use the `Fill(colorInt)` constructor, not a `fill()` helper.

Do NOT introduce Compose, React, Tailwind, or CSS. When building UI, apply the `mobile-app-ui-design` skill (spacing, color psychology, 8-point grid via `dp`, 60/30/10 color split) but implement it in **XML + ViewBinding**, using Material widgets and vector drawables for icons.

## Architecture

Layered Clean Architecture under `app/src/main/java/.../`:

```
core/network   ← Retrofit, ApiResponse envelope, AppError, interceptors, safeApiCall
core/session   ← SessionManager, tokens in EncryptedSharedPreferences
data/remote    ← FinanzasApi + DTOs + mappers (DTO → domain model)
data/local     ← Room (offline-first cache): entities, DAOs, AppDatabase
data/repository← *RepositoryImpl (implements domain/repository interfaces)
domain/model   ← business models (never DTOs)
domain/usecase ← use cases (each with a unit test)
di             ← Hilt modules
presentation/<feature> ← Activity + ViewModel (+ Adapter) per feature
```

Dependency rule: **UI → ViewModel → Repository → Retrofit/Room**. Nothing above `data` knows Retrofit or DTOs exist. DI is **Hilt** everywhere — no manual factories or service locators.

### Network result pattern (important)

Every API call returns a sealed `ApiResult<T>` (`Success(data, code)` / `Error(AppError)`), produced by `safeApiCall` / `safeUnitCall` in `core/network`. ViewModels/UI pattern-match on it and **never touch Retrofit or exceptions directly**. Branch on the response's `code` field (`ApiCode`), never on `message`. Use `onSuccess {}` / `onError {}` extensions.

`BASE_URL` is injected per build type via `buildConfigField` (debug → `http://10.0.2.2:9090/`, release → Azure) — never hardcode it. The `/api/v1` route prefix is added by the network layer, not the base URL.

## Conventions

- Domain identifiers and class names are in **Spanish** (Transaccion, Meta, Categoria, CalcularCuotaDiariaUseCase) — match the existing code. Comments/docs default to English per the surrounding file.
- DTOs stay in `data/remote/dto`; map to domain models before crossing into `presentation`.
- Every domain use case ships with a unit test.

---
> Source: [Jfra-13/app-finanzas-movile](https://github.com/Jfra-13/app-finanzas-movile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
