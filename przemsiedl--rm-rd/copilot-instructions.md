## rm-rd

> Type: Windows remote desktop (client/server) in C++

## AI Agent Guide

Project: RM_RD
Type: Windows remote desktop (client/server) in C++
OS: Windows only (WinAPI, WinSock)

### Quick Overview
- Client (GUI): connects to server, requests frames, displays bitmap, forwards input.
- Server (console): captures screen, sends frames, applies remote input.
- Shared: protocol structs, screen capture/diff, compression.
- Tests: compression round-trip test harness.

### Entry Points
- Client app: `Client/Client.cpp` (WinMain window procedure)
- Server app: `Server/server.cpp` (console main)
- Protocol: `Shared/Frame.h`
- Screen capture/diff: `Shared/BmpStream.h`, `Shared/BmpStream.cpp`
- Compression: `Shared/Compression.h`, `Shared/Compression.cpp`

### Build System
- Project files: `Client/Client.ide`, `Server/Server.ide`, `Tests/Tests.ide`
- Script: `clean.bat`
- No README or build docs are present.

### Runtime Defaults
- Server listens on TCP port 8080.
- Client default host is hardcoded in `Client/Client.cpp` (see WM_CREATE).

### Protocol Summary
- Binary protocol with packed structs in `Shared/Frame.h`.
- `FrameCmd` header uses magic "CMD", command ID, payload length.
- `HeaderBmp` header uses magic "BMP", flags, geometry, length.
- Frame flags:
  - FULL frame vs DIFF frame (XOR diff)
  - COMPRESSED flag indicates payload is compressed
- Mouse/keyboard payloads use `DataMouse` / `DataKey`.

### Data Flow
1. Client connects, sends `CMD_GET_FRAME`.
2. Server captures screen via `BmpStream`, computes XOR diff.
3. Server compresses diff (`Compression::encrypt`) and sends `HeaderBmp` + data.
4. Client receives header + payload, decompresses if needed, applies XOR diff.
5. Client builds a `HBITMAP` and displays it.
6. Client sends mouse/keyboard commands as they occur.

### Design Patterns (Observed)
- Client-Server architecture: separate GUI client and screen server.
- Command pattern (lightweight): command IDs with typed payloads.
- Strategy/Callback: `FrameGeneratorCallback` lets server override frame source.
- Double-buffering / temporal diff: current/previous frames and XOR diff.
- Producer-consumer style threading: server listens in one thread, each client in a worker thread.

### Concurrency and Threading
- Server uses a listen thread + per-client threads.
- Clients list guarded by a `CRITICAL_SECTION`.
- No async IO; socket operations are blocking with timeouts.

### Memory Ownership
- Raw pointers and manual `new[]`/`delete[]` are used widely.
- `DataBmp` can own or reference frame data (`SetReference` vs `SetReferenceWithOwn`).
- `ImageData` owns its `pData` and is non-copyable.

### Coding Style
- C++ (older style, C-like), WinAPI / WinSock.
- Mixed English + Polish comments and strings.
- Minimal error handling and logging.

### Common Modification Areas
- Protocol or frame format: `Shared/Frame.h`
- Compression or diff logic: `Shared/Compression.cpp`, `Shared/BmpStream.cpp`
- Client rendering/input: `Client/Client.cpp`, `Client/remoteclient.cpp`
- Server capture/input handling: `Server/RemoteServer.cpp`

### Testing
- `Tests/tests.cpp` validates compression round-trip.
- No automated test runner configured.

### Agent Notes and Constraints
- Keep changes Windows-compatible (WinAPI/WinSock).
- **API compatibility: use only APIs available on Windows 98** so the project runs on older machines. Do not use CRT “secure” functions (e.g. `strcat_s`, `strcpy_s`, `sprintf_s`) or other APIs introduced after Windows 98; use `strcat`, `strcpy`, `sprintf` etc. instead.
- Avoid introducing new dependencies unless required.
- Preserve binary protocol compatibility unless explicitly changing it.
- Prefer small, localized edits; document any protocol changes clearly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/przemsiedl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/przemsiedl)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
