## explored

> - Short-term: make a MVP for a flutter app, which fllows the users location(not only in foreground mode, but most importantly also in background mode). The app should be able to display the gps locations whre the user has been on the map.

## 1. Project Goals
- Short-term: make a MVP for a flutter app, which fllows the users location(not only in foreground mode, but most importantly also in background mode). The app should be able to display the gps locations whre the user has been on the map.
- Long-term: The app should have the abilitz to displaz what % of a city/region/mountain/... has explored. the app shoudl be able to save those in the users profile and be able to: retrieve the info from other devices, so that changing a phone is not an issue. Also the app should be able to get connected with other services sucha s strava, garmin connect and so on. Also the user should be able to post progress on social media (facebook, instagram, stories and so on).

## 2. Scope / Boundaries
- Scope: cover the short term project goals while keeping in mind the long term goals. We should not get started with the long term goals yet, rather just keep the code modular enough so that we have the ability to expland. 
- Architecture decisions follow the **MVVC rules** defined below.
- No ad-hoc architecture patterns; all new features must comply with View → ViewModel → Repository → Service layering.

## 3. Development Workflow
1. Create an issue with the feature spec and acceptance criteria.
2. Include expected **MVVC structure** for that feature:
   - Required View, ViewModel, Repository, Service(s).
3. Create a feature branch: `feature/<short-desc>`.
4. Implement with small, test-covered commits.
5. Open a PR linking the issue and checklist.
6. Reviewer verifies correctness **and adherence to MVVC architecture**.
7. Merge to `main` using merge commits or rebase (team decision).

## 4. Branching & Releases
- Branch strategy: `main` (stable), `feature/*`, `hotfix/*`.
- Release process: create release tag `vX.Y.Z` after passing release checklist.

## 5. CI / Checks
- Required checks for PRs:
  - `flutter analyze`
  - `flutter test`
  - formatting
  - static type/lints
- CI should reject code that violates **MVVC dependency rules** where possible (via lint/rules).

## 6. Testing Expectations
- Unit tests for pure logic.
- Widget tests for UI components.
- Integration tests for critical user flows (optionally run in scheduled CI).
- **All ViewModels, Repositories, and Services require unit tests.**

## 8. Code Review Checklist
- Does it match the issue and acceptance criteria?
- Are tests added/updated and passing?
- Does the code comply with **MVVC architectural rules**?
- Is the code documented/clean and lints satisfied?
- Are changes small and reversible?

## 9. Communication & Cadence
- Channel for async discussion: e.g., GitHub Issues + PR comments.
- Synchronous: weekly 30-min standup or ad-hoc when required.

## 10. Security & Privacy Notes
- Avoid storing secrets in repo: use secrets manager for CI.
- Note any data privacy concerns and owners.
- Services must not embed API secrets directly in code.

---

## 11. Architecture Rules: MVVC (Mandatory for All Features)

### 11.1 Layer Definitions & Allowed Dependencies
We use 4 layers:

- **View** (widgets / screens)
- **ViewModel** (UI logic + UI state + commands)
- **Repository** (data & business logic, single source of truth)
- **Service** (thin IO wrappers: HTTP/DB/platform/etc.)

Allowed dependencies:

- View → depends only on its **ViewModel**.
- ViewModel → depends on **Repositories** and optional **Use-cases**.
- Repository → depends on **Services**.
- Service → does **not** depend on any higher layer.
- No circular dependencies.
- All dependencies are passed via **constructors** (dependency injection), not global mutable singletons.

### 11.2 Per-Feature Structure
For **every feature/screen**, define at minimum:

- `SomeFeatureView`
- `SomeFeatureViewModel`
- `SomeFeatureRepository` (one or more)
- `SomeFeatureService` (one or more) for external IO

Rules:

- One **View ↔ ViewModel** pair per screen or reusable logical UI component.
- Do **not** let Views talk to Repositories or Services directly.
- Do **not** create “god” ViewModels shared across many unrelated Views.

### 11.3 Views (Widgets)
Views are **dumb** and **declarative**.

Views **may**:

- Render UI based on ViewModel state.
- Call ViewModel commands/methods in response to UI events.
- Perform simple presentation logic only:
  - show/hide with simple conditions
  - layout changes (screen size/orientation)
  - basic navigation calls

Views **must not**:

- Contain business logic.
- Access Repositories, Services, or external APIs.
- Manage non-trivial data transformations (beyond simple formatting).

Implementation rules:

- Each View constructor should accept:
  - `Key? key`
  - `SomeFeatureViewModel viewModel`
- Views rebuild by listening to ViewModel:
  - e.g., `ChangeNotifier` + `ListenableBuilder` / `AnimatedBuilder` / `ValueListenableBuilder` or equivalent.
- ViewModel is the **single source of UI state** for the View.

- Keep widget classes small and composable; split complex UIs into multiple widgets.
- Default convention: one widget per Dart file (especially for reusable widgets).
  - Exception: tiny, private helper widgets tightly coupled to a single parent may live in the same file as private classes (e.g. `_HeaderWidget`).
- Prefer extracting subtrees into widgets (not large `build()` helper methods) to keep rebuilds efficient and code readable.


### 11.4 ViewModels & Commands
ViewModels own **UI logic** and **UI state**.

ViewModel responsibilities:

- Fetch and transform data from Repositories for the UI.
- Hold all UI-relevant state:
  - loading flags
  - error messages
  - selected items
  - filters/search queries
  - pagination and other UI params
- Expose **commands** as the only entry points for user-triggered behavior.

Technical rules:

- Implement ViewModels as `ChangeNotifier` (or compatible Listenable).
- On any public-state change, call `notifyListeners()`.
- ViewModel methods that hit IO must be `async` and handle errors, mapping them into UI state.

#### Commands
A **Command** is the canonical interface for user actions.

- Every meaningful user action (button tap, pull-to-refresh, form submit, etc.) must call a **ViewModel command**, not a repository.
- A command can be:
  - a dedicated command object, or
  - a clearly named `Future<void>` method exposed on the ViewModel.
- A command should manage its own state:
  - `isRunning` / `isLoading`
  - `error` (nullable)
  - optional `lastResult` or status

Views **never** call Repositories or Services directly; they call **ViewModel commands only**.

### 11.5 Repositories
Repositories are **single sources of truth** for specific domain data.

Repository responsibilities:

- Provide APIs like `getUser`, `updateProfile`, `watchItemsStream`, etc.
- Coordinate:
  - network calls via Services
  - local caching (DB, memory, disk)
- Implement:
  - caching and data freshness strategy
  - retry, backoff, and error normalization
  - transformations of raw/service DTOs into domain models

Repository rules:

- Prefer one repository per domain concept:
  - `AuthRepository`, `UserRepository`, `ProductsRepository`, etc.
- Repositories do **not** know about Views or ViewModels.
- Repositories should not depend on each other unless data ownership requires it.
- Provide:
  - abstract interfaces for testing and environment swapping
  - concrete implementations for production, staging, and mock/fake environments.

### 11.6 Services
Services are **stateless IO wrappers**.

Service responsibilities:

- Perform low-level IO:
  - HTTP/REST, gRPC, WebSockets
  - database queries (SQLite/Isar/Hive/etc.)
  - platform channels, file system, device APIs
- Return raw data/DTOs or simple models.
- Do **not** implement:
  - caching
  - complex business logic
  - UI-related behavior

Service rules:

- One Service per external API/data source where reasonable:
  - `UserApiService`, `AuthApiService`, `LocalDbService`, `DeviceLocationService`.
- Stateless: no persistent mutable fields that represent app state.

ViewModels and Views **must not** depend on Services directly; they should always go through Repositories.

### 11.7 Data Flow Requirements
All data and events follow this strict direction: 
User → View → ViewModel (Command) → Repository → Service → Repository → ViewModel (state update) → View (rebuild)

Constraints:
- No skipping layers.
- Treat all models as effectively immutable.
- ViewModels own:
  - Current list snapshots
  - Pagination cursors
  - Filters and sort parameters




###  11.9 Dependency Injection & Navigation

Dependency Injection

- Use Provider / Riverpod or similar DI solution.
- Inject ViewModels, Repositories, and Services via constructors.
- Allow overriding dependencies in tests (mocks/fakes).

Navigation

- Use a declarative router (e.g. go_router).

View-level navigation:
- Use when navigation depends only on UI events.

ViewModel-driven navigation:
- Use when navigation depends on business logic or state (e.g., onboarding completion).

Separation of concerns:
- Keep navigation logic separate from data access.
- No repository calls inside navigation code.


### 11.10 Naming, Languages and Folder Structure

Naming:
- Views: HomeView, LoginScreen, ProfilePage
- ViewModels: HomeViewModel, LoginViewModel
- Repositories: UserRepository, AuthRepository
- Services: UserApiService, AuthApiService
- Use-cases: LoadUserProfileUseCase, SubmitOrderUseCase

Suggested structure:

lib/
  features/
    <feature>/
      view/
      view_model/
      data/
        repositories/
        services/
  domain/
    usecases/
  ui/
    core/
  assets/
 assets/
    translations/
      en.json
      de-DE.json
  lib/
    translations/
      locale_keys.g.dart   // generated
      codegen_loader.g.dart // generated (if used)

Localization (Mandatory: easy_localization)
- We use `easy_localization` for all i18n. No alternative localization systems.
- Translation files live in `assets/translations/` as JSON:
  - `{languageCode}.json` (e.g., `en.json`) - (e.g., `en-US.json`).
- No hardcoded user-facing strings in Dart. Use keys + `.tr()` (prefer generated `LocaleKeys.*`).
  - Generate keys: `flutter pub run easy_localization:generate -f keys -o locale_keys.g.dart`
  - Optional hygiene check: `flutter pub run easy_localization:audit`
- Interpolation/plurals/gender must use easy_localization features (e.g. `tr(args:, namedArgs:, gender:)`).
- MVVC rule: do not translate in Repositories/Services/ViewModels. Pass translation keys + params to the View layer and translate there.
- iOS: supported locales must be added to `ios/Runner/Info.plist` per Flutter guidance.

Creating a text message:
- always when creating a text message you have to ensure the following:
  - the message has a translation in all languages according to the above described convention
  - when creating a new text emssage we should also include test cases which check if this text message in all language fits well in the widgets that we use (no overflows, no errors etc.)


Small Files / Avoid Large Classes
- Prefer small, focused classes (single responsibility). If a class grows, split responsibilities (extract widgets, extract use-cases/helpers).
- Aim for one widget per file; put reusable widgets in a dedicated `widgets/` folder per feature.

### 11.11 Testing (Architecture-Specific)

ViewModels:
- Test commands and state transitions (loading / success / error).

Repositories:
- Test business logic, caching, error handling (mock Services).

Services:
- Test request/response handling with fake clients.

Views:
- Widget tests for key screens and states.
- Navigation tests for critical flows.


### 12. Decision Log

[2025-12-14] Adopted MVVC architecture as mandatory for all features.
Rationale: more predictable, testable, and maintainable codebase.


### 13. Milestones & Roadmap Template



Last updated: 2025-12-14

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaloqnnajdenov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kaloqnnajdenov)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
