## top-speed

> `top_speed_net/` contains the main solution, [TopSpeed.sln](top_speed/top_speed_net/TopSpeed.sln).

# Repository Guidelines

## Project Structure & Module Organization
`top_speed_net/` contains the main solution, [TopSpeed.sln](top_speed/top_speed_net/TopSpeed.sln).

Core projects:
- `TopSpeed/`: client game. Primary targets are `net472` and `net10.0-windows`/platform-specific `net10.0` builds.
- `TopSpeed.Server/`: dedicated server runtime on `net8.0`.
- `TopSpeed.Shared/`: shared protocol, localization, track/data contracts, and common helpers.
- `TS.Audio/`, `MiniAudioExNET/`, `SteamAudio.NET/`, `CrossSpeak/`: audio, native wrappers, and screen-reader integration.
- `TopSpeed.Tests/`: test project covering shared/client logic. This project is active and part of the solution.

Assets and data:
- `top_speed_net/Sounds/`
- `top_speed_net/Tracks/`
- `top_speed_net/Vehicles/`
- `top_speed_net/languages/client/`
- `top_speed_net/languages/server/`

Supporting material:
- `.github/workflows/dev-build.yml`
- `docs/`

## Current Architecture

### Server
The server is no longer expected to grow through large `RaceServer` partials. `RaceServer` is the host/facade; behavior is split under `TopSpeed.Server/Network/Services/`.

Current server boundaries:
- `Services/Session/`: handshake, packet ingress, disconnect lifecycle, session-level messaging.
- `Services/Room/`: room membership, host actions, readiness, numbering, bots, lobby state.
- `Services/Race/`: prepare/start/stop, participant finish handling, race completion.
- `Services/Notify/`: outbound room/race packet emission.
- `Services/Runtime.cs`: network/runtime scheduling.
- `Services/Chat.cs`, `Live.cs`, `Media.cs`: side channels, not room/race authority.

Rule: room ownership stays in `Room`, race ownership stays in `Race`, and packet emission stays in `Notify`. Do not reintroduce room/race logic directly into the server root.

### Client Multiplayer
Client multiplayer is split into state, dispatch, runtime, and UI reactions.

Current client boundaries:
- `TopSpeed/Game/Multiplayer/Dispatch/`: packet registration, decoding, and routing.
- `TopSpeed/Game/Multiplayer/Race/`: multiplayer race runtime and binding state.
- `TopSpeed/Core/Multiplayer/Coordinator/State/Rooms/`: authoritative room store and room drafts.
- `TopSpeed/Core/Multiplayer/Coordinator/RoomSync/Handlers/`: packet-to-store reducers.
- `TopSpeed/Core/Multiplayer/Coordinator/RoomUi/`: room UI/menu/speech reactions after store changes.
- `TopSpeed/Core/Multiplayer/Domain/Rooms/` and `Domain/Online/`: multiplayer domain models.

Rule: packet handlers update store/runtime state first; menu/speech effects happen second. Avoid mixing room state mutation with UI side effects in the same method.

### Menu, Dialog, and Localization
- Menu items and screen titles should be fed through `LocalizationService.Mark(...)` at the source.
- Shared announcement/rendering helpers should translate user-facing text centrally instead of depending on callers to pre-translate everything.
- Input/control display text should go through shared formatters, not raw enum `ToString()` output.

## Build, Test, and Development Commands
From repository root:
- `dotnet restore top_speed_net/TopSpeed.sln`
- `dotnet build top_speed_net/TopSpeed.sln -c Debug`
- `dotnet build top_speed_net/TopSpeed.sln -c Release`
- `dotnet test top_speed_net/TopSpeed.Tests/TopSpeed.Tests.csproj -c Debug --no-build`

Common publish commands:
- Server: `dotnet publish top_speed_net/TopSpeed.Server/TopSpeed.Server.csproj -c Release -p:ServerPublishProfile=ReleaseWinX64`
- Client Windows/net10: `dotnet publish top_speed_net/TopSpeed/TopSpeed.csproj -c Release -f net10.0-windows -r win-x64`

Translation templates:
- `powershell -ExecutionPolicy Bypass -File top_speed_net/languages/Generate-Templates.ps1`

## Coding Style & Naming Conventions
- Language: C#, 4-space indentation.
- Naming:
  - `PascalCase` for types, methods, properties.
  - `camelCase` for locals and parameters.
  - `_fieldName` for private fields.
- Prefer narrow ownership by folder/module, not large mixed-purpose files.
- Prefer short, responsibility-based type names inside a responsibility folder. Do not add redundant suffixes like `RoomService` inside `Services/Room`.
- Do not use dotted filenames like `RoomStore.Apply.cs`. Use folders instead, for example `State/Rooms/Apply.cs`.
- Partial classes are acceptable when they are already the local pattern and the split is coherent. Do not use partials to hide unrelated behavior.
- Prefer shared formatting/translation helpers for repeated user-facing text paths.

## Testing Guidelines
- Minimum gate: `dotnet build top_speed_net/TopSpeed.sln -c Debug` must pass with 0 errors.
- Preferred gate for logic changes: also run `dotnet test top_speed_net/TopSpeed.Tests/TopSpeed.Tests.csproj -c Debug --no-build`.
- For gameplay/network changes, do runtime smoke testing:
  - menu navigation and spoken output,
  - race loop behavior,
  - multiplayer connect/join/leave,
  - room prepare/start/finish/result flow,
  - disconnect/quit cleanup.
- For localization changes, regenerate templates and verify the affected `msgid` entries in `languages/client/messages.pot` or `languages/server/messages.pot`.

## Commit & Pull Request Guidelines
- Use focused imperative subjects, for example:
  - `Fix dialog title localization`
  - `Refactor multiplayer room store`
  - `Add localized input display text`
- Keep one concern per commit.
- When behavior changes, include the user-visible effect in the commit body or PR description.
- PRs should include:
  - what changed,
  - why it changed,
  - verification commands,
  - runtime checks when relevant.

## Security & Configuration Notes
- Do not commit secrets, tokens, or local machine-specific configuration.
- Do not commit build or publish output directories.
- Keep custom content inside approved asset folders.
- Prefer project-relative paths and checked-in assets over hard-coded local filesystem dependencies.

---
> Source: [diamondStar35/top_speed](https://github.com/diamondStar35/top_speed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
