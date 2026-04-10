## kiteboardingkit

> This file documents the codebase structure, conventions, and workflows for AI assistants working in this repository.

# CLAUDE.md — KiteboardingKit

This file documents the codebase structure, conventions, and workflows for AI assistants working in this repository.

## Project Overview

**KiteboardingKit** is a zero-dependency Swift library that provides kiteboarding/kitesurfing equipment recommendation logic for Apple platforms. The formulas originate from [Jim Douglass's interactive kiteboarding calculator spreadsheet](http://jimbodouglass.blogspot.com/2011/01/interactive-kiteboarding-calculator.html).

**Language**: Swift 6.1+
**Package Manager**: Swift Package Manager (SPM)
**Type**: Pure library — no app target, no external dependencies
**License**: MIT

---

## Repository Structure

```
KiteboardingKit/
├── .github/workflows/swift.yml        # CI: build + test on macOS with Swift 6.1
├── Sources/KiteboardingKit/
│   ├── KiteboardingKit.swift          # Main protocol + calculator implementation (203 lines)
│   ├── KiteboardingKitError.swift     # Error enum (2 cases)
│   ├── KiteSize.swift                 # KiteSize typealias + KiteSizeRange struct
│   ├── Wind.swift                     # WindSpeed struct (ideal/min/max)
│   ├── BoardSize.swift                # BoardSize + BoardOptions structs
│   └── Measurements+Extensions.swift  # Convenience extensions on Measurement<Unit*>
├── Tests/KiteboardingKitTests/
│   ├── KiteTests.swift                # 6 test methods covering kite calculations
│   └── BoardTests.swift               # 4 test methods covering board calculations
├── Package.swift                      # SPM manifest — swift-tools-version: 6.1
└── README.md
```

---

## Build & Test Commands

```bash
# Build the library
swift build -v

# Run all tests
swift test -v

# Run a specific test
swift test --filter KiteTests/testKiteSize
```

All CI runs on `macos-latest` with Swift 6.1 (see `.github/workflows/swift.yml`).

---

## Source Code Architecture

### Protocol-First Design

The public API is defined as a protocol:

```swift
// Sources/KiteboardingKit/KiteboardingKit.swift
public protocol KiteboardingCalculatorType {
    func trainerKiteSize(weight:wind:) throws -> KiteSize
    func kiteSize(weight:wind:)               -> KiteSizeRange
    func windSpeed(weight:kiteSize:)          -> WindSpeed
    func boardSize(weight:)                   -> BoardOptions
}
```

The concrete type is `KiteboardingCalculator` (a struct). Tests use the protocol type to stay implementation-agnostic.

### Unit Handling Convention

All public API parameters accept `Measurement<Unit*>` from Apple's Foundation framework — this gives callers type-safe unit conversions (kg, lbs, knots, mph, km/h, etc.).

Internally, every function converts to SI units before computing:
- Mass → **kilograms**
- Speed → **knots**

Convenience factory methods live in `Measurements+Extensions.swift`:

```swift
Measurement<UnitMass>.pounds(150)
Measurement<UnitMass>.kilograms(68)
Measurement<UnitSpeed>.knots(20)
Measurement<UnitSpeed>.kilometresPerHour(37)
```

### Core Formulas

| Method | Formula | Notes |
|--------|---------|-------|
| `trainerKiteSize` | `(0.52 × kg) / knots` | Throws if wind > 24 kn |
| `kiteSize` (ideal) | `(2.175 × kg) / knots` | — |
| `kiteSize` (min) | `0.75 × ideal` | — |
| `kiteSize` (max) | `1.50 × ideal` | — |
| `windSpeed` (ideal) | `(2.175 × kg) / m²` | Inverse of kiteSize |
| `boardSize` | `modifier × kg^(1/3)` | Different modifier per wind condition |

Results are rounded to **1 decimal place**.

### Error Handling

```swift
// Sources/KiteboardingKit/KiteboardingKitError.swift
public enum KiteboardingKitError: Error {
    case difficultToKiteboard           // Conditions outside usable range
    case trainerKiteAboveSafeLevel      // Wind > 24 knots for learner kite
}
```

### Return Types

| Type | Fields |
|------|--------|
| `KiteSizeRange` | `ideal`, `minimum`, `maximum` (all `Double`, m²) |
| `WindSpeed` | `ideal`, `minimum`, `maximum` (all `Measurement<UnitSpeed>`) |
| `BoardSize` | `length` (cm), `width` (cm), `area` (computed, cm²) |
| `BoardOptions` | `beginner`, `lightWind`, `normalWind`, `hardWind` (all `BoardSize`) |

---

## Key Conventions

- **Structs over classes** — all public types are value types.
- **No external dependencies** — keep the library dependency-free.
- **Foundation `Measurement` for all units** — never pass raw `Double` for a physical quantity in the public API.
- **Internal SI conversion** — convert to kg/knots before any calculation, convert back for output if needed.
- **MARK comments** — use `// MARK: -` to separate logical sections within files.
- **Protocol-based abstraction** — new calculator variants should conform to `KiteboardingCalculatorType`.
- **Rounding** — results rounded to 1 decimal place using `(value * 10).rounded() / 10`.

---

## Testing Conventions

- Framework: **Swift Testing** (`import Testing`, `@Test`, `#expect`)
- Test files mirror the feature they test (`KiteTests.swift` → kite calculations, `BoardTests.swift` → board calculations).
- Tests use both imperial and SI inputs to verify unit conversion correctness (e.g., `kiteSize` vs. `kiteSizeSI`).
- Use `#expect` for assertions and `#expect(throws:)` for error expectations.
- Test methods use the `@Test` attribute and do not require a `test` prefix.

---

## CI/CD

Defined in `.github/workflows/swift.yml`:
- Triggers on **push** and **pull request** to `main`.
- Runs `swift build -v` then `swift test -v` on `macos-latest` with Swift 6.1.
- There is no separate lint step — the Swift compiler warnings serve as the linter.

---

## What to Avoid

- Do **not** add external Swift package dependencies without a strong justification.
- Do **not** expose raw `Double` values for physical quantities in the public API — wrap them in `Measurement`.
- Do **not** introduce classes when a struct will do.
- Do **not** change the rounding behavior without updating the test expectations.
- Do **not** push directly to `main` — use a pull request.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dornad)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dornad)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
