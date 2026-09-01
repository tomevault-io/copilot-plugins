## intentflow

> IntentFlow architecture rules for iOS features


# IntentFlow Rules

IntentFlow features are workflow-first.

## Required Feature Files

Prefer this shape for non-trivial features:

- `<Feature>Contract.swift`
- `<Feature>Flow.swift`
- `<Feature>Effects.swift`
- `<Feature>Projection.swift`
- `<Feature>Tests.swift`
- `<Feature>.intentflow.yaml` for AI mode

## Reducer Rules

- Reducers must be pure.
- Reducers may return `Next.state`, `.effect`, `.cancel`, `.output`, and `.route`.
- Reducers must not create `Task`, call async APIs, access UI, read storage, or perform networking.
- Add an explicit failure state for async workflows.
- Add an explicit in-progress state for async workflows.

## Effect Rules

- Effects belong in a `FlowEffectHandler`.
- Long-running effects must have an `EffectID`.
- Replaceable effects should use `.cancelInFlight`.
- `AsyncStream` effects must cancel internal tasks in `onTermination`.

## UI Rules

- SwiftUI views and UIKit view controllers are adapters.
- UI may render projected state and send intents.
- UI must not decide product workflow transitions.
- Navigation is emitted as a `Route` and interpreted by the app shell.

## AI Mode Rules

- Update the manifest before changing contract types.
- Every new state must appear in the manifest.
- Every new effect must have a test or acceptance trace.
- Invariants must not be weakened silently.
- If a generated change violates an invariant, stop and ask for a design decision.

---
> Source: [emrecanozturk/intentflow-ios](https://github.com/emrecanozturk/intentflow-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
