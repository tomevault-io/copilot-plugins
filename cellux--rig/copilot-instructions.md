## rig

> - Do not run `make` and any `./build/rig ...` smoke tests in parallel.

# Notes

- Do not run `make` and any `./build/rig ...` smoke tests in parallel.
- You must build first, wait for the build to finish, and only then run any `./build/rig ...` smoke tests.
- You must use `make test` for final verification.
- `./build/rig ...` smoke tests are supplemental only. They must not be used as a substitute for `make test`.
- Any change to C code, build files, or embedded builtin module sources requires rebuilding `rig` before running `./build/rig ...`.
- Any Lua or Fennel file under `src/modules/` counts as an embedded builtin module source unless the task explicitly proves otherwise.
- Any change under `scripts/` requires rebuilding `rig` before running tests or `./build/rig ...`.
- The only Lua/Fennel changes that do not require rebuilding `rig` are standalone scripts under `tests/` or `examples/`.
- When in doubt, run `make test`.
- For repo-wide pattern searches or mechanical cleanups, search the full relevant tree, not just `src/`.
  - Include `tests/`, `examples/`, and `scripts/` unless the task is explicitly scoped more narrowly.
- This project is used only by its sole author, so backward compatibility is not a concern.

## API Naming

- When exposing a C API through a Lua module, mirror the original C naming after removing the module or library prefix.
- Preserve the original casing style:
  - `CamelCase` stays `CamelCase`
  - `snake_case` stays `snake_case`
- If a C module prefixes all symbols with its module name, expose that prefix as the Lua module name and remove it from the member name.
  - Example:
    - C: `SDL_GetTicks`
    - Lua: `sdl3.GetTicks`
- If a Rig C module loads symbols dynamically via `dlsym`, each bound symbol should use a `rig_X__NAME` identifier in `X.c`.
  - Use a double underscore for dynamically bound symbols from the underlying C library.
- If a Rig C module implements its own Lua-facing abstraction in `X.c`, its identifier should use `rig_X_NAME`.
  - Use a single underscore for Rig-owned wrapper or abstraction names.
- Constant names may mirror the original C names exactly after removing the module or library prefix.
  - There is no need for double underscores in constant names.
  - We do not reserve constant names for any separate Rig-owned naming scheme.
- Constants may remain in all-caps when mirroring foreign APIs or representing enum-like values.

## Runtime And API Design

- `rig.run(...)` is the only public runtime entrypoint.
  - Runtime-aware modules should register modes and hooks with `rig`.
  - Backend modules should not expose their own public main-loop driver.
- Per-run startup and runtime configuration should go through `rig.run(...)` options.
  - Avoid monkey-patching module globals to control runtime behavior.
  - Use module-scoped option tables such as `sdl3 = { ... }`, `sdl3_gpu = { ... }`, and `uv = { ... }`.
  - Use app methods for run-local phase and event behavior, and `rig.register_runtime_hook(...)` for persistent module-level hooks.
- Public async APIs should be coroutine-based.
  - Callback-style async APIs are internal implementation details.
  - User-facing async operations should suspend through `sched`.
- `sched` is the generic scheduler layer.
  - Backend-specific yield protocol details should be hidden behind module APIs.
  - Users should not have to call `sched.await("backend.op", ...)` directly when a proper wrapper exists.
- If a utility is backend-agnostic, put it in `rig`.
  - Backend-specific modules may wrap it with backend-specific convenience methods.
- Runtime behavior must not depend on module load order.
  - Use explicit runtime mode ownership via `mode = "..."`.

---
> Source: [cellux/rig](https://github.com/cellux/rig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
