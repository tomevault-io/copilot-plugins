## go-libghostty

> - **Build:** `make build` (runs CMake + `go build`)

# Libghosty Go Bindings

## Commands

- **Build:** `make build` (runs CMake + `go build`)
- **Test:** `make test`
- **Clean:** `make clean` (removes `build/`, re-triggers CMake fetch on next build)

## Build

- CMake fetches ghostty source via `FetchContent`; pinned commit is in `CMakeLists.txt`
- Never modify anything under `build/`
- cgo links static library by default; `-tags dynamic` links shared

## Code Guidelines

- Ghostty headers are at `build/_deps/ghostty-src/zig-out/include/ghostty/`
- Use a heavy commenting style that explains what/why clearly.
- Comment every exported function, type, field, etc.
- Types should be in the same filename as the C header, by default.
  If they're already moved, keep them where they are. For example,
  `color.h` types should go in `color.go`.
- If a type is a thin layer over a C type, comment with what
  C type it corresponds to, e.g. `C: GhosttyResult`
- When a C API uses `GhosttyString` with `len=0`, libghostty treats
  the pointer as garbage, so no nil special-casing is needed.
- Define constants by their C reference, e.g.
 `DAConformanceVT100  = C.GHOSTTY_DA_CONFORMANCE_VT100`. Don't
  hardcode values.

---
> Source: [mitchellh/go-libghostty](https://github.com/mitchellh/go-libghostty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
