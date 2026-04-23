## trips-flutter-app

> Use when you want **design → code**, driven by the real **Figma design system**:


# Flutter rules

## Folder structure
lib/
├── main.dart               # Application entry point
├── app/
│   ├── core/               # Shared core functionality
│   │   ├── constants/      # App-wide constants
│   │   ├── utils/          # Utility functions & extensions
│   │   ├── widgets/        # Shared, reusable widgets
│   │   └── common/         # Shared types (e.g., Failure, PaginatedResponse)
│   └── features/           # Application features (modular)
│       └── feature_name/
│           ├── domain/     # Business logic (entities, repositories)
│           │   ├── entity.dart
│           │   └── feature_repository.dart    # The only repository (concrete)
│           │
│           ├── data/       # Data layer (sources + models)
│           │   ├── remote/
│           │   │   ├── feature_remote_source.dart
│           │   │   └── feature_dto.dart
│           │   │
│           │   └── local/
│           │       ├── feature_local_source.dart
│           │       └── feature_isar.dart
│           │
│           └── presentation/ # UI layer (bloc/pages/widgets)

**Key principles:**
- **Feature-first**: Each feature is isolated and self-contained.  
- **Clean Architecture**: 
  - `domain` = pure business logic  
  - `data` = implementations (remote/local + repository impl)  
  - `presentation` = UI with state management  
- **Shared code**: Placed in `core/`  
- **Common UI**: Shared widgets live in `core/widgets/`  

---

## Packages
- **State management:** Prefer **BLoC**  
  - Separation of concerns  
  - Testable, predictable state  
  - Strong ecosystem support  

- **Local persistence:** Prefer **Isar**  
  - High-performance, schema-based storage  
  - Excellent offline-first support  

- **Networking:** Prefer **Dio**  
  - Rich interceptors (logging, auth, retries)  
  - Strong ecosystem of plugins  

- **Dependency injection:** Prefer **get_it** (+ `injectable`)  
  - Decoupled service management  
  - Test-friendly  
  - Scales with feature-first architecture  

- **Testing:** Prefer **mocktail** and **bloc_test**  
  - Simplified mocking  
  - Strong bloc-driven testing patterns

- use Equatable instead of manual equality overrides

---

## Models
Each feature model should have **three versions**:

- **Domain model (business logic)** → `User`  
  - Pure, framework-agnostic  
  - Contains only business-related fields  

- **API model (JSON-serializable)** → `UserDto`  
  - Handles mapping from/to API responses  
  - Keeps external quirks out of the domain layer  

- **Persistence model (Isar)** → `UserIsar`  
  - Annotated with `@collection`  
  - Optimized for local storage  

- Each model lives **with its source** in the same folder:  
  - `remote/` → API models (Dto)  
  - `local/` → persistence models (Isar)  
  - `domain/` → domain entities  

---

## Sources & Repositories
- **Remote sources** → `UserRemoteSource`  
- **Local sources** → `UserLocalSource`  
- **Repository interface** → `UserRepository` (in domain)  
- **Repository implementation** → `UserRepositoryImpl` (in data)  

---

## UI
- Prefer **creating widgets** over helper methods. Default to creating a widget if:  
  - It has internal state  
  - It needs context-specific logic  
  - It’s a complex or reusable visual element  

- Use **helper methods only** for:  
  - Tiny, stateless code fragments  
  - Inline formatting (e.g., text spans, padding shortcuts)  

---

# MCP Servers for Flutter

## Dart MCP Server
Use the Dart & Flutter MCP server when you want your AI assistant to operate with your project’s **real context + official tooling**, not just suggest code.  

- **Fix runtime/layout issues fast**: e.g. inspect widget tree, propose/apply minimal fixes.  
- **Code-aware troubleshooting**: resolve symbols, imports, re-check analysis.  
- **Package discovery/integration**: search pub.dev, add package, scaffold usage code.  
- **Test-driven iteration**: run tests, analyze failures, apply targeted changes.  
- **App introspection**: query runtime state, hot reload, fetch current errors.  
- **Consistent hygiene**: run formatter/analysis with official tooling.  
- **Scoped refactors**: rename symbols, extract widgets, move files safely.  
- **Onboarding**: surface repo conventions, scaffold starter code.  

**When not to use it**  
- Large, risky rewrites → break them into smaller, reviewed steps.  
- Clients without Tools/Resources → fall back to normal chat suggestions.  

---

## Figma Flutter MCP Server
Use when you want **design → code**, driven by the real **Figma design system**:

- Extract design tokens → theming  
- Scaffold widgets & screen layout  
- Ensure UI matches design source of truth  
- Maintain design-code sync as Figma updates

**Ideal workflow**
> Design in Figma → Extract with MCP → AI generates Flutter UI → Dev refines logic + architecture

---

# Code Quality Rules

## Dart Style & Analysis

### Line Length
Use **maximum line length = 100** characters  
→ Update `.editorconfig` / `analysis_options.yaml` accordingly

```yaml
# analysis_options.yaml example
code-style:
  line-length: 100
```

---

### Color API (Flutter ≥ 3.22)
Avoid using:

Deprecated  
```dart
color.withOpacity(0.5);
```

Use:  
```dart
color.withValues(alpha: 0.5);
```

---

### Background was deprecated in favor of surface for improved Material 3 consistency.
Avoid using:

Deprecated  
```dart
color.withOpacity(0.5);
```

Use:  
```dart
theme.colorScheme.surface
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ISL270) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
