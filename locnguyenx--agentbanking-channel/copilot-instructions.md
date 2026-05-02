## agentbanking-channel

> This document provides instructions and guidelines for AI coding agents working in this repository.

# AGENTS.md - Guidelines for Agentic Coding Agents

This document provides instructions and guidelines for AI coding agents working in this repository.

## Project Overview

This project is building Channel App for **Agent Banking** facilitating financial services at third-party retail locations.
* **Regulatory Compliance:** Bank Malaysia standards.
* **Security:** Zero-trust architecture. No PII in logs. Hardware-level encryption for PINs.

## Documentation
- `docs` - at project root
- `docs/ideas` - high level requirements (ARCHITECTURE.md, BRD_SUMMARY.md) formal specs (BRD, BDD, Design) for Channel App
- `docs/superpowers/specs/agent-banking-channel/*-brd.md` - BRD for Channel App
- `docs/superpowers/specs/agent-banking-channel/*-bdd.md` - BDD for Channel App
- `docs/superpowers/specs/agent-banking-channel/2026-03-27-agent-banking-channel-design.md` - System Design Specification for Channel App
- `docs/api/openapi.yaml` - external API spec

## 1. Architecture

### 5-Tier System Architecture for Agent Banking

All agents MUST understand this architecture before making any code changes:

1. **Tier 1: Channel Layer (This project)** — POS Terminals (Android/Flutter)
2. **Tier 2: Spring Cloud Gateway** — JWT validation, rate limiting, routing
3. **Tier 3: Domain Core Services** — Rules, Ledger & Float, Onboarding, Switch Adapter, Biller
4. **Tier 4: Translation Layer** — HSM Connector, Switch Connector, Biller Connector
5. **Tier 5: Downstream Systems** — HSM, PayNet, JPN, Billers

See `docs/superpowers/specs/agent-banking-platform/*-design.md` for full architecture details.

### Channel App's Layers
The Channel App lives exclusively in **Tier 1: Channel Layer**.

*   **Presentation:** Flutter Widgets, Riverpod (State Management), routing via `go_router`. Designed for Dual-Display layouts (Agent vs Customer view).
*   **Domain:** Business logic, Use Cases (e.g., `ExecuteDualHandshakeUseCase`), and abstract Entity classes.
*   **Data:** Repositories implementing Domain interfaces. Includes REST API clients (`dio`), local database (`sqflite`), and device secure storage.
*   **Hardware Abstraction Layer (HAL):** Abstraction wrappers around physical POS peripherals (printers, EMV readers) using Android `MethodChannel`.

## CRITICAL RULES (NON NEGOTIABLE)

* **The app is a consumer.** It never implements business rules that belong to the backend (float balances, fee calculation, velocity checks, AML). Those are backend responsibilities accessed via API calls.
* NEVER HARDCODED
* DO NOT MOCK device's features that are supported by the device

## 2. Core Technical Components

### 2.1 State Management & Dependency Injection
*   **Framework:** `Riverpod` (`hooks_riverpod`).
*   **Providers Strategy:**
    *   `authProvider`: Manages JWT lifecycle and background session expiration checks.
    *   `floatBalanceProvider`: Streams Agent Float polling every 30 seconds.
    *   `transactionProvider`: A robust `StateNotifier` governing the strict Dual-Handshake state machine: `INIT -> QUOTING -> WAITING_CONSENT -> WAITING_CARD -> WAITING_PIN -> PROCESSING -> SUCCESS/LOCKED`.

### 2.2 Hardware Abstraction Layer (HAL)
To decouple the app from specific vendor SDKs (e.g., Sunmi, Pax, Aisino), all hardware interactions occur via abstract Dart contracts.
*   **Contracts:** `ICardReader`, `IPinPad`, `IPrinter`, `IBiometricScanner`.
*   **Implementation:** `VendorAPrinterImpl` implements `IPrinter` by communicating over a `MethodChannel` (`com.banking.channel/printer`) to the native Android host which executes the proprietary `.jar/.aar` SDK.
*   **Fallback Strategy:** Features degradation is automatic. If `IPrinter.isAvailable()` is false, the app safely defaults to SMS-only receipts without crashing.

### 2.3 Store & Forward Engine (Local Database)
*   **Engine:** `sqflite` bundled with `sqlcipher` for AES-256 local database encryption.
*   **Encryption Key:** The database master key is securely generated once per device and permanently vaulted inside the **Android Keystore** via `flutter_secure_storage`.
*   **Queue Entity:** Failed reversals (MTI 0400) and offline logs are serialized as JSON and pushed to the `txn_queue` table with their original `X-Idempotency-Key`.
*   **Sync Worker:** `workmanager` schedules a background isolate to execute every 15 minutes, pushing pending queue items to the Backend API.

---

## 3. Security & Anti-Fraud Mechanisms

### 3.1 Protection Measures
1.  **TLS & Pinning:** Mandatory TLS 1.2+ for all `dio` client traffic. API Certificate SHA-256 hashes are pinned to reject Man-In-The-Middle attacks.
2.  **Display Obfuscation:** The Flutter app is restricted with `WindowManager.LayoutParams.FLAG_SECURE` on native Android, preventing OS-level screenshots or screen recordings.
3.  **Encrypted PIN Processing:** The app **never** renders a virtual keyboard for PINs. It strictly delegates PIN capture to the HSM (Hardware Security Module) built into the POS device, which returns an encrypted DUKPT PIN-Block.
4.  **Zero PII Logging:** A custom Logger intercepts all logs. Regex parsers proactively redact strings matching 16-digit PANs or 12-digit MyKads before writing to console/Crashlytics.

### 3.2 Anti-Smurfing & Compliance Locks
When the Backend Rule Engine returns an `ERR_COMPLIANCE_FREEZE` code:
*   The `authProvider` immediately flips `isComplianceLocked = true`.
*   The `go_router` instantly redirects the navigation stack to a dead-end `ComplianceLockScreen`, popping all previous routes.
*   The state persists across app reboots via encrypted local storage until the Backend proactively sends an Unlock webhook.


## API Contract Enforcement

**OpenAPI 3.0 Specification** is the single source of truth for all REST APIs.

## Testing Guidelines
* Integration tests: use BDD test framework `bdd_widget_test`
* BDD scenarios in `*.feature` files are the acceptance criteria
* All BDD scenarios must be run in `test/bdd/features/` directory

**TDD ENFORCEMENT**
* TDD pattern must be complied in development
* Make sure that all the tests are stablized first in the RED phase.
* NEVER change the test to get PASSED in GREEN/REFACTOR phase instead of fixing production code

## Banking-Specific Guidelines

### Money Handling
* All monetary values use `BigDecimal` — NEVER use `float` or `double`.
* Rounding: `HALF_UP` to 2 decimal places.
* Currency: Always `MYR` — validate on all endpoints.

### Audit Trail

### Security
* PINs: Hardware-level encryption via HSM. DUKPT PIN blocks. Never decrypted outside HSM.
* PAN: Masked in all responses and logs (first 6, last 4 digits).
* MyKad: Encrypted at rest (AES-256). Never in plaintext logs.
* TLS 1.2+ for all external traffic.
* mTLS for internal service-to-service communication.

### Geofencing
* Transactions allowed only within 100m of registered Merchant GPS coordinate.
* If GPS unavailable: reject transaction with `ERR_GPS_UNAVAILABLE`.

### Velocity Checks
* Limit transactions per MyKad per day to prevent smurfing.
* Configurable via VelocityRule entity.

---
> Source: [locnguyenx/agentbanking-channel](https://github.com/locnguyenx/agentbanking-channel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
