## providers-and-secrets

> Calendar provider, OWA, account, secret-storage, and TLS guardrails.


# Providers and secrets

Observed project facts:

- `OWAWidget/Models/CalendarAccount.swift:15` declares `CalendarAccount` as `Codable`, `Sendable`, and `Hashable`.
- `OWAWidget/Models/CalendarAccount.swift:36` through `OWAWidget/Models/CalendarAccount.swift:39` intentionally exclude passwords from coding keys.
- `OWAWidget/Services/KeychainService.swift:4` through `OWAWidget/Services/KeychainService.swift:60` store, load, and delete passwords via Keychain.
- `OWAWidget/Services/CalendarService.swift:135` through `OWAWidget/Services/CalendarService.swift:156` save/delete passwords through `KeychainService` when accounts change.
- `OWAWidget/Services/CalendarService.swift:255` through `OWAWidget/Services/CalendarService.swift:274` rebuild providers from accounts and Keychain passwords.

Rules:

- Never store passwords, cookies, CANARY tokens, session headers, or real corporate server addresses in source files, tests, release notes, docs, logs, or fixtures.
- Passwords must remain in Keychain. Do not add password fields to `CalendarAccount` coding keys or persisted account JSON.
- Do not weaken TLS validation by default. If supporting a local corporate Exchange scenario requires a TLS exception, make it explicit, user-controlled, and documented in the plan.
- When adding a calendar provider, implement `CalendarProvider`, add the account type to `CalendarAccount`, and wire provider creation through `CalendarService.rebuildProviders()`.
- OWA request and response changes should preserve CANARY handling, redirect/cookie behavior, and offline cache fallback.
- Keep logged diagnostic data useful but sanitized. Prefer counts, status codes, paths, and token names over full URLs, cookies, request bodies, or credentials.

Relevant tests to consider:

- `Tests/OWAWidgetTests/OWARequestPayloadTests.swift`
- `Tests/OWAWidgetTests/OWAResponseDecodingTests.swift`
- `Tests/OWAWidgetTests/OWAJoinURLResolutionTests.swift`
- `Tests/OWAWidgetTests/MeetingURLDetectorTests.swift`
- `Tests/OWAWidgetTests/CalendarEventCodableTests.swift`
- `Tests/OWAWidgetTests/CalendarServiceOfflineTests.swift`

---
> Source: [ilyabazhenov/mac-owa-widget](https://github.com/ilyabazhenov/mac-owa-widget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
