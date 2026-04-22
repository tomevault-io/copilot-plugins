## m3t-atendee-app

> BLoC and Cubit conventions — sealed events, Equatable state, part files, copyWith sentinel, error handling.


# BLoC / Cubit Conventions

When adding or editing BLoCs or Cubits in this project, follow these patterns.

## File Layout

- **Bloc:** One main file plus `part` files for events and state.
  - `lib/features/<feature>/bloc/<feature>_bloc.dart` — contains `part '..._event.dart'; part '..._state.dart';` and the Bloc class.
  - `lib/features/<feature>/bloc/<feature>_event.dart` — `part of '<feature>_bloc.dart';` and event classes.
  - `lib/features/<feature>/bloc/<feature>_state.dart` — `part of '<feature>_bloc.dart';` and state class (and enums if needed).
- **Cubit:** Same folder; typically `<feature>_cubit.dart` and `<feature>_state.dart` (no events).

Barrel: `bloc/bloc.dart` exports the three (or two for Cubit) files.

## Events (Bloc only)

- Use **sealed** + **final** classes: `sealed class <Feature>Event extends Equatable` with `final class <EventName> extends <Feature>Event`.
- All events are `const` constructors.
- Override `List<Object?> get props` with all fields (use `[]` for parameterless events).
- No Freezed; use manual Equatable.

## State

- `final class <Feature>State extends Equatable`.
- Use **enums** for status/step when it helps (e.g. `LoginStep`, `LoginStatus`).
- **copyWith:** For nullable fields use a **sentinel** so callers can distinguish “omit” from “set to null”:

```dart
static const _sentinel = Object();

State copyWith({
  SomeEnum? step,
  String? optionalField = _sentinel,
}) {
  return State(
    step: step ?? this.step,
    optionalField: optionalField == _sentinel ? this.optionalField : optionalField as String?,
  );
}
```

- Include all state fields in `props`.

## Bloc vs Cubit

- **Bloc:** Multiple event types, event-driven flows (e.g. login steps, form actions). Use `Bloc<<Feature>Event, <Feature>State>`.
- **Cubit:** Single flow or internal methods only (e.g. load/update user). Use `Cubit<<Feature>State>` and expose methods like `loadCurrentUser()`, `updateAvatar()`.

## Dependencies and Injection

- Blocs/Cubits depend only on **repository interfaces** from `package:domain/domain.dart` (e.g. `AuthRepository`), not on concrete implementations.
- Inject the repository via constructor; the app provides it with `context.read<AuthRepository>()` in `BlocProvider`/`BlocProvider` `create`.

## Error Handling

- In event handlers: `try { ... await repository.someMethod(); emit(newState); } on Object catch (error, stackTrace) { addError(error, stackTrace); emit(state.copyWith(status: Failure, errorMessage: ...)); }`
- Map domain failures to user-facing strings (e.g. `AuthFailure.toDisplayMessage()`, `UnknownError().toDisplayMessage()`) and put them in state (e.g. `errorMessage`) for the UI.
- For stream-based auth: subscribe to `authRepository.status` in the Bloc constructor and `add(AuthStatusChanged(status))`; cancel the subscription in `close()`.

## Lifecycle

- Override `close()` to cancel subscriptions and call repository cleanup (e.g. `authRepository.dispose()`) when the Bloc owns that responsibility.

## References

- `lib/app/bloc/auth_bloc.dart`, `auth_event.dart`, `auth_state.dart`
- `lib/features/login/bloc/login_bloc.dart`, `login_event.dart`, `login_state.dart`
- `lib/features/user/bloc/user_cubit.dart`, `user_state.dart`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/redplanettribe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
