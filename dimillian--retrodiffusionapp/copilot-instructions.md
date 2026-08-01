## retrodiffusionapp

> - SwiftUI-only UI; use the Observation framework (`@Observable`, `.environment(...)`) for state sharing.

# Agent Guide for RetroDiffusionApp

## Core Practices
- SwiftUI-only UI; use the Observation framework (`@Observable`, `.environment(...)`) for state sharing.
- Keep view state local when possible; services/clients are injected via `@Environment(Service.self)`.
- Concurrency: use Swift 6.2 async/await. Only keep actors where isolation matters (networking, library store). Prefer stateless utilities for heavy work plus background tasks.
- Avoid cross-actor hops for CPU work; offload pure work with `Task.detached` or `Task(priority:)` from outside the actor and keep actor state out of those closures.
- Snapshot mutable actor state before awaits (e.g., copy `apiKey` before making requests).
- Respect Sendable where appropriate; don’t capture non-Sendable objects inside `Task` closures.

## Key Services
- `Networking` (actor): API calls. Wrapped by `NetworkClient` (`@MainActor @Observable`) for SwiftUI usage.
- `ImageUtils` (struct): resize/base64 helpers; pure, stateless. Call from background tasks when heavy.
- `ImageSaver` (class): async Photos save via continuations; stateless, use from background tasks.
- `LibraryStore` (actor): SQLite-backed metadata + image persistence, paginated fetches.
- `LibraryClient` (`@MainActor @Observable`): pages through `LibraryStore`, caches images in `NSCache`, exposes async save/delete/load.
- `GenerationQueue` (`@MainActor @Observable`): orchestrates generate/pixelate tasks, depends on `NetworkClient` + `LibraryClient`.

## Environment Injection (current setup)
- App entry (`RetroDiffusionAppApp`): creates `NetworkClient`, `LibraryClient`, `GenerationQueue`, injects via `.environment(...)`, and wires dependencies with `setDependencies(networkClient:libraryClient:)`.
- Views use `@Environment(NetworkClient.self)` and `@Environment(LibraryClient.self)` (plus `GenerationQueue` where needed).

## Library UI
- `LibraryView` uses `LibraryClient.images` with paging; triggers `loadNextPage()` near the end of the list.
- Thumbnails/details load images asynchronously via `LibraryClient.loadImage`; a small in-memory cache is maintained in `LibraryView` for sharing.

## Networking Flow
- Preprocess images off-actor (`Task.detached`) before requests.
- Capture `apiKey` into a local before request building to avoid mid-flight changes.

## Setup Notes
- API key: load from `Config.plist` (`API_KEY`) or user default override; `Config.plist` is gitignored.
- iOS target 26.1+, Xcode 26.1+. Pure SwiftUI; no Storyboards.

## When Adding Code
- Keep CPU/IO off the main actor unless UI-bound; use async/await.
- Prefer small, focused components; keep logging reasonable (avoid dumping base64).
- Be cautious adding new actors—only where state isolation is required; otherwise use utilities + background tasks.

---
> Source: [Dimillian/RetroDiffusionApp](https://github.com/Dimillian/RetroDiffusionApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
