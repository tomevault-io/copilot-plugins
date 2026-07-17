## minley-budget-app

> Android budget tracking app built with Kotlin, Jetpack Compose, and Supabase.

# MinleyBudget Kotlin - Project Context

## Project Overview

Android budget tracking app built with Kotlin, Jetpack Compose, and Supabase.

## Tech Stack

- **UI**: Jetpack Compose with Material 3
- **Architecture**: MVVM with Hilt DI
- **Local prefs**: DataStore Preferences (no Room; persistent data reads go through Supabase)
- **Remote Database**: Supabase (PostgreSQL)
- **Auth**: Supabase Auth (Email + Google OAuth)
- **Serialization**: kotlinx.serialization

## Important Files

- **Data Models**: `app/src/main/java/com/minley/budget/data/model/`
- **Supabase Config**: `app/src/main/java/com/minley/budget/core/network/SupabaseClient.kt`

## Remember

- Supabase is the source of truth for the live database schema and RLS/RPC configuration.

---
> Source: [NesDevr/minley-budget-app](https://github.com/NesDevr/minley-budget-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
