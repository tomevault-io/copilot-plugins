## 004-testing-guidelines

> Testing guidelines for HermesWhisper

# Testing Guidelines

- **Framework**: XCTest for unit and UI tests; use async tests where applicable.
- **Naming**: `test_<UnitUnderTest>_<Behavior>()` (e.g., `test_AudioService_handlesPermissionDenied`).
- **Scope**: prioritize pure logic and critical flows; add UI tests for key journeys.
- **Coverage**: aim ≥80% for core modules.
- **Run**: use the Test command rule or Xcode’s Test action.

Place unit tests in ["HermesWhisperTests/"](mdc:HermesWhisperTests/) and UI tests in ["HermesWhisperUITests/"](mdc:HermesWhisperUITests/).

---
> Source: [dkapo88/hermeswhisper](https://github.com/dkapo88/hermeswhisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
