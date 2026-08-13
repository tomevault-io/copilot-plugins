## maintenance-state-store

> Design document: Prevent OTA updates during vehicle operation

# CLAUDE.md — maintenance-state-store Developer Guide

## Project Overview

Design document: Prevent OTA updates during vehicle operation

This library is part of the safety mechanism that prevents OTA updates from being applied
to a vehicle while it is driving. It serves as the **Source of Truth for the Fail-Prevent layer**,
persisting the maintenance state (OFF / ON / UNKNOWN) to the filesystem with atomic writes
and corruption detection.

C++ and Python are **independent implementations** sharing the same file format and API contract.
They are kept in separate directories and use separate build/packaging systems.

---

## Common Commands

### C++ build

```bash
cmake -B build
cmake --build build
```

### C++ tests

```bash
cmake --build build && cd build && ctest --output-on-failure
```

### Python install

```bash
pip install -e .
```

### Python tests

```bash
pip install -e . && pytest tests/test_store.py -v
```

### All tests (C++ + Python)

```bash
cmake --build build && cd build && ctest --output-on-failure && cd .. && pytest tests/test_store.py -v
```

---

## Key Files

### C++

| File                                                          | Role                                                          |
| ------------------------------------------------------------- | ------------------------------------------------------------- |
| `include/maintenance_state_store/maintenance_state_store.hpp` | Public C++ API (`State` enum + `Store` class)                 |
| `src/maintenance_state_store.cpp`                             | Core implementation (CRC32, atomic write, read)               |
| `CMakeLists.txt`                                              | Build definition — produces `.a` and `.so`; ament_cmake support |
| `package.xml`                                                 | ROS2 package manifest — required for colcon workspace builds    |
| `tests/test_store.cpp`                                        | GoogleTest suite (20 cases)                                   |
| `tests/mss_cli.cpp`                                           | CLI tool for cross-language tests (`write`/`read`)            |
| `tests/fixtures/`                                             | Shared JSON fixtures verified by both test suites             |

### Python

| File                                          | Role                                       |
| --------------------------------------------- | ------------------------------------------ |
| `python/maintenance_state_store/__init__.py`  | Package entry point (re-exports)           |
| `python/maintenance_state_store/_store.py`    | Implementation (`State`, `Store`)          |
| `python/maintenance_state_store/py.typed`     | PEP 561 marker for type checkers           |
| `pyproject.toml`                              | Package config (hatchling)                 |
| `tests/test_store.py`                         | pytest suite (27 cases, incl. cross-lang)  |

### Versioning

| File      | Role                                                    |
| --------- | ------------------------------------------------------- |
| `VERSION` | Single source of truth for the project version (`0.1.0`) |

Both `CMakeLists.txt` (`file(READ VERSION ...)`) and `pyproject.toml` (`[tool.hatch.version]`) read from this file.
On release, the CI overwrites it with the tag: `echo "${GITHUB_REF_NAME#v}" > VERSION`.

### CI

| File | Role |
| --- | --- |
| `.github/workflows/test.yml` | `cpp`, `python` (matrix), `sonar`, `compat` — on push/PR |
| `.github/workflows/release.yml` | `python` only — on `v*` tag push, uploads `.whl`/`.tar.gz` |

---

## Architecture Decisions

### C++ and Python are independent implementations

Both implement the same file format and API contract (read / write / force_write).
They are not linked: the Python package uses `binascii.crc32` (stdlib) and `os.rename()`
rather than wrapping the C++ library via pybind11.

**Rationale**: a pure Python package can be installed with `pip install` without a C++ toolchain,
which is essential for CLI tools, monitoring services, and Ansible playbooks.

**Risk mitigation**: cross-language compatibility is verified at three levels:

1. `test_checksum_round_trip` — Python recomputes CRC32 independently and checks it matches
2. Shared fixture files (`tests/fixtures/`) — both suites read the same pre-generated JSON files
3. Live subprocess tests — `mss_cli` (C++ binary) writes a file that Python reads, and vice versa; runs in the `compat` CI job

### Public API is `State` + `Store` only

Internal helpers (`state_to_string` in C++, `_canonical` / `_checksum` in Python)
are not part of the public API and must not be called from outside their modules.

### Writing `UNKNOWN` is rejected

`write(UNKNOWN)` returns `false` / `False` and does nothing.
`UNKNOWN` is a read-only error state that signals the system to block both driving and OTA.

### `force_write` is identical to `write` in implementation

The distinction is documented intent only: `force_write` is reserved for
CLI / maintenance tooling and must not be called from application logic.

### Checksum target

CRC32 is computed over `"{version}|{state}|{timestamp}"` (e.g. `"1|ON|1760000000"`).
Only the semantically meaningful fields are checksummed, not the full JSON.

C++ uses a lookup-table CRC32 with polynomial `0xEDB88320` (standard Ethernet / ISO 3309).
Python uses `binascii.crc32`, which implements the same algorithm.

### Atomic Write sequence

1. Write to a `.tmp` file in the same directory (same filesystem — guarantees `rename` atomicity)
2. `fsync()` the `.tmp` file to flush to disk
3. `rename()` to atomically replace the target file
4. `fsync()` the parent directory to persist the directory entry

On write failure, the `.tmp` file is removed on a best-effort basis.
The existing state file is never corrupted.

### Default path

`/var/lib/maintenance_state_store/state.json`

Autoware, Client, and the monitoring service all reference this path.
Override via the `Store` constructor argument.

---

## C++ Dependencies (fetched automatically via FetchContent)

| Library       | Version | Purpose         |
| ------------- | ------- | --------------- |
| nlohmann/json | v3.11.3 | JSON read/write |
| GoogleTest    | v1.15.2 | C++ unit tests  |

## Python Dependencies

None beyond the standard library (`json`, `os`, `binascii`, `pathlib`, `time`, `enum`).

---

## Mapping to Design Document

| Design doc section               | C++ implementation                                   | Python implementation                         |
| -------------------------------- | ---------------------------------------------------- | --------------------------------------------- |
| Section 6 / File Writing Library | `src/maintenance_state_store.cpp`                    | `python/maintenance_state_store/_store.py`    |
| Provided API — Read              | `Store::read()`                                      | `Store.read()`                                |
| Provided API — Write             | `Store::write()`                                     | `Store.write()`                               |
| Provided API — Repair            | `Store::force_write()`                               | `Store.force_write()`                         |
| Storage format (JSON)            | `write()` — nlohmann::json construction              | `write()` — `json.dumps()`                    |
| Atomic Write guarantee           | tmp → fsync → rename → fsync(dir)                    | tmp → fsync → os.rename → fsync(dir)          |
| Corruption detection             | `read()` — version / checksum / field validation     | `read()` — same validation logic              |
| Fail-safe on corruption          | `read()` — all error paths return `State::UNKNOWN`   | `read()` — all error paths return `UNKNOWN`   |

---

## CI Artifacts

| Artifact name                          | Contents                                  |
| -------------------------------------- | ----------------------------------------- |
| `maintenance_state_store-cpp-<sha>`    | `.a` + `.so*` + `.hpp` + CMake config     |
| `maintenance_state_store-python-<sha>` | `.whl` + `.tar.gz` (built on Python 3.12) |

The `compat` job has no artifact; it only runs the cross-language tests.

---

## Notes

- Tests never write to `/var/lib/...` (requires root); they use temporary directories.
- `build/` is excluded from version control via `.gitignore`.
- `state_to_string` is in the anonymous namespace of `maintenance_state_store.cpp`
  and cannot be called from outside the translation unit.
- `_canonical` and `_checksum` in Python are module-private (leading underscore).
- Cross-language tests in `test_store.py` skip automatically when `build/tests/mss_cli`
  does not exist, so a Python-only install still works without a C++ toolchain.

---
> Source: [tier4/maintenance-state-store](https://github.com/tier4/maintenance-state-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
