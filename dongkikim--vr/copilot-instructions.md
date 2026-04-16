## vr

> This document provides a comprehensive overview of the VRApp project to serve as instructional context for Gemini CLI.

# VRApp (Value Rebalancing App) - Gemini Context

This document provides a comprehensive overview of the VRApp project to serve as instructional context for Gemini CLI.

## Project Overview

VRApp is an Android application designed to implement and manage the **Value Rebalancing (VR)** stock investment strategy. It automates the calculation of valuation bands (Low/High) and provides precise buy/sell guides based on current stock prices and user-defined parameters (Value, Gradient, Pool).

### Key Features
- **VR Calculation:** Automatic determination of buy/sell/hold actions and quantities.
- **Market Support:** Tailored tick size logic for KOSPI, KOSDAQ, US, and Japan markets.
- **Asset Management:** Portfolio tracking with principal, valuation, and ROI calculations.
- **History & Visualization:** Daily asset snapshots and stock-specific history visualized with charts.
- **Data Persistence:** Local storage with Room DB and JSON-based backup/restore functionality.

## Technical Stack

- **Language:** Kotlin (100%)
- **UI Framework:** Jetpack Compose (Material Design 3)
- **Architecture:** MVVM (Model-View-ViewModel)
- **Database:** Room (SQLite) with KSP for annotation processing
- **Networking:** Jsoup (for web parsing) & Yahoo Finance API (via `StockPriceService`)
- **Concurrency:** Kotlin Coroutines & Flow (StateFlow for reactive UI)
- **Charts:** MPAndroidChart
- **Serialization:** Gson (for data backup)

## Project Architecture

### 1. UI Layer (`com.example.vrapp.ui`)
- **`VRApp.kt`:** Main entry point and navigation host (Bottom Navigation: Home, Chart, Settings).
- **`MainScreen.kt`:** Dashboard showing stock list, asset summary, and filters/sorting.
- **`DetailScreen.kt`:** In-depth view of a single stock, VR calculation table, and transaction history.
- **`ChartScreen.kt`:** Visual representation of asset growth and stock performance.
- **`SettingsScreen.kt`:** App configuration and backup/restore tools.

### 2. ViewModel Layer (`com.example.vrapp.viewmodel`)
- **`StockViewModel.kt`:** The central hub for business logic. It manages UI state, coordinates data from the repository, handles price refreshes, and executes transactions.

### 3. Logic Layer (`com.example.vrapp.logic`)
- **`VRCalculator.kt`:** Core mathematical engine.
    - **Formula:** `V_new = V_old + (Pool / G) + poolDeposit`
    - **Bands:** `Low = V * (1 - G/100)`, `High = V * (1 + G/100)`
    - Handles market-specific **Tick Sizes** (e.g., KOSPI price-dependent steps).

### 4. Data Layer (`com.example.vrapp.data`)
- **`AppDatabase.kt`:** Room database definition with auto-migration support.
- **`StockRepository.kt`:** Abstracts data access from the DAO.
- **`StockPriceService.kt`:** Handles real-time price fetching and exchange rate updates.

### 5. Models (`com.example.vrapp.model`)
- `Stock`: Main entity containing VR parameters (V, G, Pool, vrPool).
- `TransactionHistory`: Records of buys, sells, deposits, and VR recalculations.
- `DailyAssetHistory`: Periodic snapshots of the entire portfolio value.
- `StockHistory`: Snapshots of individual stock states for chart rendering.

## Development & Build Commands

| Command | Description |
| :--- | :--- |
| `./gradlew build` | Full project build |
| `./gradlew installDebug` | Install the debug APK to a connected device/emulator |
| `./gradlew test` | Run local unit tests (JUnit) |
| `./gradlew connectedAndroidTest` | Run instrumented tests on a device |
| `./gradlew clean` | Remove build artifacts |

## Key Conventions

1.  **Reactive UI:** Use `StateFlow` in ViewModels and `collectAsStateWithLifecycle` in Compose.
2.  **Immutability:** Data models are mostly immutable; use `.copy()` for updates.
3.  **Core Logic Isolation:** Keep the VR mathematical logic in the pure `VRCalculator` object for testability.
4.  **Currency Handling:** Always consider exchange rates (USD/JPY/KRW) when calculating total portfolio value.
5.  **Room Migrations:** Schema changes require updating the version in `AppDatabase` and ensuring auto-migration or manual migration scripts are present in `app/schemas`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dongkikim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
