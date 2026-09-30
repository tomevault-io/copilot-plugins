## object-tracking

> 1. **Clean Architecture Strictness**:

# Enterprise Flutter & Dart Standards (Vibecode Edition)

## Core Architecture & Engineering
1. **Clean Architecture Strictness**:
   - Presentation: BLoC / Cubit. State Management MUST use Single Immutable State pattern with `copyWith` and Enums for status tracking (never subclass state classes).
   - Domain: Pure Dart. Entities, UseCases (Single Responsibility), Repository Contracts. Zero Flutter/UI or Data/External dependencies.
   - Data: DTOs, DataSources, Repository Implementations.
   - Core: Singletons, Failures, Router, Network Clients, Themes, DI (GetIt).
2. **Code Quality & Modern Practices**:
   - Use latest Flutter/Dart features (Null Safety, Pattern Matching, Records, Extension Types).
   - Absolute Immutability: Use `final` for class fields, `const` constructors, and immutable collections where feasible.
   - Strict Typing: `dynamic` or loose `Object?` are strictly prohibited without runtime type checking.
   - Error Handling: Functional handling using `dartz` (`Either<Failure, Success>`) for clean failure propagation.
   - Dependency Injection: Always register new UseCases, Repositories, DataSources, and BLoCs in `lib/core/services/service_locator.dart` using GetIt.

3. **Design System & UI/UX Guidelines (Fallback Visuals)**:
   - **Modern Aesthetic**: When no brand/visual identity is supplied, generate clean, premium, and calm UI/UX design tokens.
   - **Color Palette**: Eye-pleasing, low-saturation backgrounds (e.g., Off-White `#F8F9FA`, Charcoal Dark `#121212`), matched with elegant primary accents (Emerald `#0F766E` or Indigo `#4338CA`). Avoid high-contrast harsh primary colors.
   - **Typography & Layout**: Consistent spacing scales (4-8px base unit), subtle border-radii (12-16px), minimal drop-shadows, and smooth micro-animations.
   - **Responsiveness**: Ensure Adaptive Layouts supporting Mobile, Web, and Desktop windows out-of-the-box.
   - **RTL & Localized**: Support internationalization and RTL layout alignment seamlessly.

4. **Self-Consistency & Persistence**:
   - Always auto-check generated code using `flutter analyze` standards before suggesting completion.
   - Maintain self-documenting code with concise doc-comments for public interfaces.

---
> Source: [AFB-11/Object_Tracking](https://github.com/AFB-11/Object_Tracking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
