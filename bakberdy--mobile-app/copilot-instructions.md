## mobile-app

> Domain layer rules: no Flutter/data imports, UseCase naming, analytics events, file naming, failure analytics.


# Domain Layer Rules

## Imports
- ❌ NO `flutter/` imports — domain is pure Dart
- ❌ NO imports from any feature's `data/` layer
- ✅ Default: `dart:*`, `dartz`, `injectable`, `equatable`, own `core/` (error, usecases, monitoring/analytics)
- `freezed_annotation` is allowed **only** for entities that use Freezed under the folder rule in `model-entity-separation.mdc` (not for use cases, repositories, or flat files in `entity/`)
- Freezed entities must follow **`build.yaml`**: no Freezed **`fromJson`/`toJson`**, no **`.map`/`.when`** — see `model-entity-separation.mdc`

## File naming
Domain classes have **no `Entity` suffix** on the class name or file name.

**Repository and analytics** — file name uses the feature prefix (the contract is feature-owned):
- ✅ `user_config_repository.dart`, `user_config_events.dart`, `routes_route.dart` (routes feature)
- ❌ `repository.dart`, `events.dart`

**Use cases** — one file per use case under `domain/usecases/`. File name is **snake_case from the class**, without repeating the feature prefix (the feature folder already scopes imports):
- ✅ `get_app_theme_mode_use_case.dart`, `set_theme_use_case.dart`, `get_locale_use_case.dart`
- ❌ `user_config_get_app_theme_mode_use_case.dart` (redundant), `get_theme.dart` (missing `_use_case` suffix)

**Entities** — under `domain/entity/`. Prefer `Equatable` (or enums). Use **Freezed** only when it clearly helps (e.g. sealed unions, many immutable variants). If you use Freezed, the entity **must** live in its own subfolder — see `model-entity-separation.mdc`. Otherwise prefer a clear file name; use a feature prefix when the same concept could exist in multiple features:
- ✅ `app_theme_mode.dart` (`AppThemeMode` enum), `routes_route.dart` (`Route`)
- ✅ `domain/entity/example_todo/example_todo.dart` (`@freezed` + `part` + generated file in that folder)
- ❌ `routes_route_entity.dart`, `AppThemeModeEntity`, `@freezed` directly as `entity/foo.dart` (no subfolder)

Class naming:
- ✅ `Route`, `RouteType`, `AppThemeMode`, `GetRouteRequestParameters`
- ❌ `RouteEntity`, `RouteTypeEntity`, `GetRouteRequestParametersEntity`

## UseCase naming
Class must end with `UseCase` (not `Usecase`). File must end with `_use_case.dart` (not `_usecase.dart`).

```dart
// ❌ BAD
class GetAppThemeModeUsecase ...          // file: get_app_theme_mode_usecase.dart

// ✅ GOOD
class GetAppThemeModeUseCase ...          // file: get_app_theme_mode_use_case.dart
class GetLocaleUseCase ...                // file: get_locale_use_case.dart
```

## Analytics events
Each feature owns its analytics events in `domain/analytics/<feature_name>_events.dart`.
Event classes extend `AnalyticsEvent`. Named factory constructors are mandatory — no inline raw strings.

```dart
// ✅ GOOD — user_config: domain/analytics/user_config_events.dart
final class GetAppThemeModeUseCaseEvent extends AnalyticsEvent {
  const GetAppThemeModeUseCaseEvent({required super.name, super.properties});

  factory GetAppThemeModeUseCaseEvent.success() =>
      GetAppThemeModeUseCaseEvent(name: 'get_app_theme_mode_use_case_success');

  factory GetAppThemeModeUseCaseEvent.failure({required Map<String, dynamic> properties}) =>
      GetAppThemeModeUseCaseEvent(name: 'get_app_theme_mode_use_case_failure', properties: properties);
}

// ❌ BAD — raw string inline, not in domain/analytics/
Analytics.track(AnalyticsEvent(name: 'get_theme_failure'));
```

## Failure tracking in UseCases
Failures MUST be sent to analytics **only inside UseCases**, not in BLoC or data layer. See `core.mdc` for analytics conventions.

```dart
// ✅ GOOD — from user_config: get_app_theme_mode_use_case.dart
@override
FutureEither<AppThemeMode?> call(NoParams params) async {
  final result = await _repo.getAppThemeMode();
  return result.fold(
    (failure) {
      Analytics.track(GetAppThemeModeUseCaseEvent.failure(properties: {
        AnalyticsPropertyKeys.failureMessage: failure.message,
        AnalyticsPropertyKeys.failureType: failure.type.name,
        AnalyticsPropertyKeys.failureSource: failure.source,
      }));
      return result;
    },
    (_) { Analytics.track(GetAppThemeModeUseCaseEvent.success()); return result; },
  );
}
```

## UseCase interface and return types
All UseCases MUST extend `UseCase<T, Params>` (or `StreamUseCase<T, Params>` for streams) from `core/usecases/use_case.dart`.
Return type MUST be `FutureEither<T>` — never write out `Future<Either<Failure, T>>` in full.

When a UseCase takes no input, use `NoParams` as the `Params` type.
The `Params` class (when not `NoParams`) must be defined in the same file, below the UseCase class.

```dart
// ✅ GOOD — no input parameters
@LazySingleton()
class GetAppThemeModeUseCase implements UseCase<AppThemeMode?, NoParams> {
  final UserConfigRepository _repo;
  GetAppThemeModeUseCase(this._repo);

  @override
  FutureEither<AppThemeMode?> call(NoParams params) async {
    // ...
  }
}

// ✅ GOOD — with parameters, Params class in the same file
@LazySingleton()
class SetAppThemeModeUseCase implements UseCase<void, SetAppThemeModeParms> {
  final UserConfigRepository _repo;
  SetAppThemeModeUseCase(this._repo);

  @override
  FutureEither<void> call(SetAppThemeModeParms params) async {
    // ...
  }
}

class SetAppThemeModeParms {
  const SetAppThemeModeParms({required this.themeMode});
  final AppThemeMode themeMode;
}

// ✅ GOOD — stream use case
@LazySingleton()
class WatchRouteUpdatesUseCase implements StreamUseCase<List<Route>, NoParams> {
  final RoutesRepository _repo;
  WatchRouteUpdatesUseCase(this._repo);

  @override
  StreamEither<List<Route>> connect(NoParams params) => _repo.watchRoutes();

  @override
  Future<void> disconnect() => _repo.closeRouteStream();
}

// ❌ BAD — wrong base class, full generic written out, Params defined elsewhere
class GetThemeUseCase {
  Future<Either<Failure, AppThemeMode?>> call() async { ... }
}
```

## Repository interfaces
Repository interfaces live in `domain/repository/`. They are pure Dart — no implementation details.
Return types MUST use `FutureEither<T>` or `StreamEither<T>`.

```dart
// ✅ GOOD
abstract interface class UserConfigRepository {
  FutureEither<AppThemeMode?> getAppThemeMode();
  FutureEither<void> setAppThemeMode(AppThemeMode mode);
}
```

## Failures are business logic only
`Failure` types belong in domain. Raw exceptions (storage, network) are caught in `data/` and mapped there.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bakberdy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
