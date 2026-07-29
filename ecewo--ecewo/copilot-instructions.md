## ecewo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ecewo is a single-threaded, asynchronous C web framework with an Express.js-style API. It is built on libuv (event loop), llhttp (HTTP/1.1 parsing), and rax (radix-tree router), with a custom arena allocator. The entire public API is one header (`include/ecewo.h`); every public symbol is `ecewo_*`-prefixed and types are opaque for ABI/FFI safety.

## Commands

The `makefile` wraps the CMake/CTest workflow. `make help` lists everything.

- `make test` — configure `build/` with `-DECEWO_BUILD_TESTS=ON`, build, run the full CTest suite.
- `make asan-ubsan` / `make msan` / `make tsan` — sanitizer builds (MSAN/TSAN require clang) into separate `build-*/` dirs.
- `make valgrind` — memcheck run.
- `make fuzz` — build libFuzzer targets (`build-fuzz/fuzz-router`, `fuzz-route-register`); requires clang.
- `make format` / `make format-file FILE=src/x.c` — clang-format.
- `make lint` / `make lint-fix` / `make lint-file FILE=src/x.c` — clang-tidy. **Requires `build/` to already exist** (it reads `build/compile_commands.json` via `-p build`), so run `make test` or `cmake -B build` first.

### Running a single test

Tests are CTest cases named by their bare name; each is also a standalone binary `ecewo-test-<name>`:

```sh
ctest --test-dir build -R body-streaming --output-on-failure   # by name (regex)
./build/ecewo-test-body-streaming                              # run the binary directly
```

### Adding a test

1. Create `tests/test-<name>.c` (use `tester.h` macros: `RUN_TEST`, `ASSERT_EQ`, `RETURN_OK`, etc., and `ecewo-mock.h`).
2. Register it in `CMakeLists.txt` with `ecewo_test(<name>)`.
3. Reconfigure: `cmake -B build -DECEWO_BUILD_TESTS=ON`.

Tests link `ecewo::mock` (the `ecewo-mock` plugin, pulled via `ecewo_add(mock@dev)`), which spins up a **real** server on `TEST_PORT` (8888). `MockParams`/`request()` cover most cases; tests that need precise TCP framing (split reads, chunked, oversized bodies) open raw sockets to `TEST_PORT` directly — see `tests/test-body-streaming-split.c`.

The library builds with `-Werror` (unused function/variable) and tests with `-Wall -Wextra -Werror`, so warnings break the build.

## Architecture

### Process runtime singleton

There is one process-wide runtime (`ecewo__runtime_t`, static in `src/server.c`) that owns the shared libuv loop, signal handlers, the pooled-arena subsystem, and the registry of apps. It is lazily initialized on the first `ecewo_create()` and torn down via `atexit`. Multiple `ecewo_app_t` instances share this one loop, each bound to its own port (`ecewo_bind`/`ecewo_run`, or the combined `ecewo_listen`). Per-app shutdown is independent: one app can stop while others keep serving.

### Request lifecycle — the central, non-obvious flow

`server_on_read` (server.c) → `router()` (router.c) → `dispatch()` (router.c). The parser and request state are **connection-scoped**: `http_context_t persistent_context` is embedded in `ecewo_client_s` and its buffers live in the connection arena, so request state survives across multiple TCP reads.

The key trick: `on_headers_complete_cb` (http.c) returns `HPE_PAUSED`. This pauses llhttp at headers-complete so the router can match the route and decide buffered-vs-streaming **before the body is parsed**, then `llhttp_resume()`s. Consequences:

- **Buffered (default):** the handler is *deferred* — the matched handler/middleware/req/res are saved into `client->pending_*` with `handler_pending = true`, and run once the full body has buffered. `run_pending_handler()` is the single place that attaches the completed body and runs the saved chain (called from both the same-read path and the split-read path).
- **Streaming (opt-in):** the `ecewo_body_stream` middleware sets `http_context_t.on_body_chunk` so `on_body_cb` forwards chunks to the user's `on_data`/`on_end` callbacks instead of buffering. Saved as `client->stream_req` when the body spans multiple reads; completed via `body_stream_complete()`.

`router()` returns `REQUEST_KEEP_ALIVE` / `REQUEST_CLOSE` / `REQUEST_PENDING` (PENDING = handler hasn't replied yet, e.g. awaiting more body or async work). `client->request_in_progress` gates whether a new read resets the context (`client_context_reset`).

Gotcha that has bitten before: the buffered-path 413 guard in `dispatch()` must test `has_stream_middleware`, **not** `ctx->on_body_chunk` — `on_body_chunk` is only set when the stream middleware runs inside `chain_start`, which is *after* the guard, so keying off it wrongly rejects large/chunked streaming bodies. Buffered cap is `BUFFERED_BODY_MAX_SIZE` (1 MB, http.h); streaming default cap is `BODY_MAX_SIZE` (10 MB, body.c), tunable per-request via `ecewo_body_limit`.

### Memory: arenas, not malloc/free

Almost nothing user-facing is individually freed. A pooled LIFO of arenas (`src/arena/arena-pool.c`) hands out a `connection_arena` per connection via `ecewo_arena_borrow` / `ecewo_arena_return`; all per-request allocations (parsed headers, body, req/res structs, handler scratch via `ecewo_alloc`/`ecewo_sprintf`/etc.) are freed as one block when the connection ends or resets. Long-lived/plugin state goes in the app-lifetime arena (`ecewo_app_arena`). Rule: never manually free anything ecewo returns.

### Routing & middleware

`src/route-table.c` is a rax radix tree (O(1)-avg lookup) supporting `:param` and `*` wildcard routes. Two registration styles, both ending up in the same table: convenience macros (`ECEWO_GET`/`ECEWO_POST`/...) and the builder API (`ecewo_route_new` / `ecewo_route_middleware` / `ecewo_route_handler`, in `src/route-register.c`) used by plugins and FFI. Middleware (`src/middleware.c`) is global, path-prefix, or route-specific; `chain_start` runs the chain, and `ecewo_context_set`/`ecewo_context_get` pass per-request values from middleware to handler.

### Async & connection takeover

`ecewo_spawn` (`src/spawn.c`) offloads blocking work to the libuv thread pool and resumes the continuation on the loop thread; a process-wide async-work counter keeps the loop alive while background tasks are in flight. `ecewo_connection_takeover` (server.c) hands the raw TCP socket to a plugin (e.g. WebSocket) after the HTTP upgrade — afterwards ecewo no longer manages the HTTP lifecycle, and `handle->data` is repointed to the plugin's user data (recovered in close callbacks via the fact that `uv_tcp_t handle` is the first field of `ecewo_client_s`). Clients are ref-counted (`ecewo_client_ref`/`unref`) so they can be held across async boundaries.

## Plugin system (CMake)

Plugins are separate repos fetched at configure time. `cmake/registry.cmake` declares them with `ecewo_define_plugin(<name> REPO <url> DEFAULT_VERSION <tag>)`; consumers pull them with `ecewo_add(name@version)` (version optional → default tag). Each plugin must define the target `ecewo::<name>`. The functions live in `cmake/plugins.cmake`.

## Conventions

- Public symbols: `ecewo_*`. Internal symbols shared across translation units: `ecewo__*` (double underscore). File-local: plain statics.
- The public header is C++-safe (`extern "C"`) and written for FFI bindings — keep public types opaque. See `docs/16.ffi-bindings.md`.
- `ecewo_header_set` silently rejects reserved framing/hop-by-hop headers (`Content-Length`, `Transfer-Encoding`, `Connection`, `Host`, `Date`).
- Detailed usage docs live in `docs/` (numbered `01`–`17`), with `17.api-reference.md` as the API reference.

---
> Source: [ecewo/ecewo](https://github.com/ecewo/ecewo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
