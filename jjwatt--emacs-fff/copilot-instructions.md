## emacs-fff

> Welcome! If you are an AI agent working on `emacs-fff`, this document provides essential context to help you understand the architecture, build system, and specific implementation details of this project.

# Context for AI Agents (emacs-fff)

Welcome! If you are an AI agent working on `emacs-fff`, this document provides essential context to help you understand the architecture, build system, and specific implementation details of this project.

## Project Overview

`emacs-fff` is an Emacs frontend for [fff.nvim](https://github.com/dmtrKovalenko/fff.nvim), a fast, typo-resistant fuzzy file finder. Unlike traditional Emacs packages that communicate with external tools via async processes and JSON/IPC, `emacs-fff` calls directly into a compiled shared library (`libfff_c.so`) using `tromey/emacs-ffi`.

## Architecture & FFI

The architecture relies heavily on dynamic C FFI:
- **`libfff_c.so` (Rust):** The core logic, compiled from the `fff-c` crate inside the `fff.nvim` repo. It exposes a synchronous C ABI.
- **`emacs-ffi` (C/Emacs Lisp):** A bridge module (`ffi-module.so` and `ffi.el`) that allows Emacs to call C functions using `libffi`.
- **`fff.el` (Emacs Lisp):** The core Emacs package that defines FFI bindings to `libfff_c.so`, handles memory management, and implements the backend protocol.
- **UI Backends (`fff-helm.el`, `fff-ivy.el`, `fff-consult.el`):** Packages that plug into `fff.el`'s data layer to provide the actual user interface.

### Important Data Handling Concepts

- **No JSON:** Results from `libfff_c` are returned as `repr(C)` structs, not JSON. Emacs Lisp accesses these fields directly by reading memory offsets using `ffi--mem-ref`.
- **Struct Offsets:** The byte offsets for fields in `FffResult`, `FffFileItem`, `FffSearchResult`, and `FffGrepMatch` are hardcoded in the `;;; Struct field readers` section of `fff.el`. **If the Rust struct definitions in `fff.nvim` change, these offsets MUST be updated.**
- **Memory Management:** `fff_create_instance` returns an opaque pointer (`FffInstance`). Results are returned wrapped in `FffResult` pointers. The Emacs code must explicitly free these (e.g., via `fff_free_result`, `fff_free_search_result`, etc.). `fff.el` provides a macro `fff--with-result` to handle checking for errors and freeing the result envelope.

## Build System (`Makefile`)

The build process is completely automated via the `Makefile`:
1. It checks for prerequisites (Rust, C compiler, libltdl, libffi, Emacs with module support).
2. It compiles the vendored `emacs-ffi` module (`ffi-module.c`).
3. It clones the `fff.nvim` repository into `.build/fff.nvim`.
4. It builds `libfff_c.so` by running `cargo build -p fff-c` inside the cloned repo.
5. On macOS, it patches the dynamic library links using `install_name_tool` to ensure relocatability.

When asked to "build" the project, simply run:
```bash
make
```

## Known Gotchas & Constraints

- **Dynamic Binding & Helm:** `fff-helm.el` must use top-level `defun`s for candidate functions (e.g., `fff--helm-candidates`), rather than anonymous lambdas. Helm evaluates candidate functions in a dynamic binding context, and using anonymous lambdas can lead to `void-function` errors.
- **Libffi `uint64` Bug:** Passing or returning large `uint64` values can cause Emacs to crash on certain systems due to `libffi` mishandling. As a workaround, `fff.el` intentionally avoids calling `fff_wait_for_scan` directly and instead implements a polling loop checking `fff_is_scanning` (`fff--wait-for-scan-poll`).
- **Use-Package & Straight.el:** Users installing this locally with `straight.el` need to add `:straight nil` to their `use-package` block, or straight will attempt to download the package from MELPA/GitHub.

## Modifying Code

- When fixing bugs in the FFI layer, always double-check pointer types and memory lifecycle.
- When creating or updating a UI backend, adhere to the backend plist protocol defined in `fff.el` (providing `:pick-file` and `:pick-grep` closures).

---
> Source: [jjwatt/emacs-fff](https://github.com/jjwatt/emacs-fff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
