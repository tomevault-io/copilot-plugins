## photoc

> - Write C17. Keep code straightforward; prefer simple functions and data structures over clever abstractions.

# Repository guidance for Codex

## Scope and structure

- Write C17. Keep code straightforward; prefer simple functions and data structures over clever abstractions.
- Put reusable logic in `src/core/` and command-specific logic in `src/commands/`. Keep `src/main.c` limited to the CLI entry point.
- Commands must not duplicate shared filesystem, metadata, hashing, or image logic. Put such logic in `src/core/` when it is needed by more than one command.
- Keep dependencies minimal and justify any new dependency. Prefer portable POSIX-compatible code where practical. macOS and Linux are the initial supported platforms.

## CLI and file safety

- Write requested result data to stdout and errors to stderr. Pass a borrowed `const photoc_output *` to command/output helpers; do not store verbosity in global mutable state.
- Use `photoc_output_info` for informational status and operation summaries, `photoc_output_verbose` for extra stderr diagnostics, and `photoc_output_metadata_warning` for non-critical metadata warnings. Result/JSON writers remain unfiltered. Failure explanations use the existing unconditional error helpers, even if labeled warnings. Keep JSON stdout unchanged across output levels.
- Keep exit codes stable and predictable. The current CLI uses `0` for success, `2` for usage errors, and `3` for commands that are not implemented. Document any new exit code before using it.
- File-modifying commands should support a dry-run mode where appropriate. Never overwrite user files unless explicitly requested.
- Make memory ownership explicit. Document who owns and frees dynamically allocated memory, especially at API boundaries.
- Use the shared progress subsystem for long operations. Only the caller thread renders to stderr; worker threads never print spinner frames.

## Verification

- Add tests for all new functionality. Include a regression test with each bug fix.
- Build with warnings enabled and resolve warnings before finishing. The CMake target treats supported compiler warnings as errors.
- Configure, build, and run the test suite with:

  ```sh
  cmake -S . -B build
  cmake --build build
  ctest --test-dir build --output-on-failure
  ```

- For a memory-safety run, configure a separate tree with
  `-DPHOTOC_SANITIZERS=ON` (AddressSanitizer and UndefinedBehaviorSanitizer)
  and run the same build and `ctest` commands there, or use
  `sh scripts/build-and-test-sanitizers.sh`.
- Static analysis is `sh scripts/static-analysis.sh` (Clang analyzer, and
  clang-tidy when installed). `cmake -DPHOTOC_CLANG_TIDY=ON` runs clang-tidy
  during the build. See `CONTRIBUTING.md`.
- Optional scan benchmarks: `python3 scripts/benchmark.py` (see
  `benchmarks/README.md`). Do not optimize from these numbers unless asked.

---
> Source: [ahmetomerv/photoc](https://github.com/ahmetomerv/photoc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
