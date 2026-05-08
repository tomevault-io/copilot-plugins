## ysmparser

> - Use CRLF (preferred windows.)

# AGENTS.md

## Repository expectations

- Use CRLF (preferred windows.)

## Project overview

- This repository is a C++20/CMake project for parsing and exporting `.ysm` files.
- There are two supported runtime paths:
  - Native CLI tool: batch-processes `.ysm` files from an input directory to an output directory.
  - WebAssembly build: compiles the same parser core for browser or Node.js use via Emscripten.
- Main project code lives in `YSMParser/`.
- Browser UI assets live in `web/`.
- Third-party dependencies are vendored under `external/`.

## Key entry points

- Native CLI entry: `YSMParser/main.cpp`
- Parser abstraction and factory:
  - `YSMParser/parsers/YSMParser.hpp`
  - `YSMParser/parsers/YSMParser.cpp`
- Version-specific parsers:
  - `YSMParser/parsers/YSMParserV1.cpp`
  - `YSMParser/parsers/YSMParserV2.cpp`
  - `YSMParser/parsers/v3/YSMParserV3.cpp`
- V3 header/path mapping helpers:
  - `YSMParser/parsers/v3/YSGPHeaderParser.hpp`
  - `YSMParser/parsers/v3/PlayerTextureCollector.hpp`
- Web UI:
  - `web/index.html`
  - `web/app.js`

## Current parser architecture

- `YSMParserFactory::Create()` reads the input file and selects a parser implementation by header/version.
- V1 parser:
  - Reads base64 file names.
  - Uses AES-CBC + zlib.
  - Exports resources directly from an in-memory map.
- V2 parser:
  - Similar to V1, but derives the real AES key using MD5 + Java-style PRNG before decrypting payloads.
- V3 parser is the main complexity center:
  - Parses YSGP header metadata.
  - Verifies file integrity with CityHash.
  - Decrypts with custom XChaCha20-based logic plus MT19937 XOR.
  - Decompresses with zstd.
  - Deserializes models, animations, textures, sounds, language files, controllers, and metadata JSON.
  - Exports files using header-derived hashed path mappings when available.

## CLI behavior

- CLI app name in code: `YSM File Decryptor`.
- Important arguments:
  - `-i, --input`: input directory, required.
  - `-o, --output`: output directory, required.
  - `-v, --verbose`: detailed per-file logging.
  - `-j, --threads`: parallel file processing, `0` means auto.
- The CLI scans recursively for `.ysm` files only.
- Each input file is exported into its own subdirectory under the output root, usually named after the input stem.
- In non-verbose mode, stdout is intentionally silenced during parsing and a progress renderer is shown instead.
- Verbose mode forces single-threaded execution to keep logs ordered.
- If output-directory name collisions are detected between files, the tool also forces single-threaded execution.

## WebAssembly / web app notes

- Top-level CMake supports `YSM_TARGET_WASM=ON`.
- `YSM_WASM_ENV` may be `node` or `web`.
- Relevant presets in `CMakePresets.json`:
  - `wasm-release`
  - `wasm-web-release`
- `web/index.html` is a standalone browser UI for selecting `.ysm` files, running the wasm parser, and downloading a ZIP.
- `web/app.js`:
  - Loads `YSMParser.js` and the wasm module.
  - Writes selected files into the Emscripten FS under `/input`.
  - Calls the CLI entry via `callMain(["-i", "/input", "-o", "/output"])`.
  - Collects `/output` and packs the result with JSZip.
- If working on the web flow, keep native CLI behavior and wasm behavior aligned because the web build reuses the same parser entry path.

## Build system notes

- Top-level build files:
  - `CMakeLists.txt`
  - `CMakePresets.json`
- Native presets exist for Windows x86/x64 plus Linux/macOS release builds.
- The project vendors and builds dependencies directly through CMake subdirectories.
- `version.txt` is read by CMake and injected as `YSM_PARSER_VERSION`.
- For wasm builds, the executable suffix becomes `.js` and additional Emscripten link options are applied.

## Third-party dependencies in active use

- `external/zstd`
- `external/zlib`
- `external/cityhash`
- `external/xchacha20`
- `external/AES`
- `external/md5`
- `external/cpp-base64`
- `external/fpng`
- `external/json/json.hpp`

## Important implementation observations

- `README.md` is currently minimal and does not explain the real architecture; rely on source first.
- V3 contains most of the domain logic and is the first place to inspect for format/export bugs.
- `YSGPHeaderParser` is important because it converts hashed resource identifiers into output paths and also builds `ysm.json`/player metadata structure.
- Path handling already tries to be UTF-8 safe via `PathUtils` and `PlatformCompat`; preserve that behavior when touching file IO.
- Some V3 code still contains debug-style `printf`, `std::cout`, `__debugbreak`, and exploratory parsing code. Be careful not to break wasm or release builds when cleaning this area up.
- There is no obvious dedicated automated test suite in the main project. Validation currently appears to depend on building and running against real sample files.

## Recommended workflow for future changes

- For parser bugs, inspect in this order:
  - `YSMParser/parsers/YSMParser.cpp`
  - The relevant version parser implementation.
  - `YSMParser/parsers/v3/YSGPHeaderParser.hpp` if exported file names/paths look wrong.
- For CLI/output behavior issues, start with `YSMParser/main.cpp`.
- For browser issues, inspect both the wasm preset configuration and `web/app.js`.
- Prefer small, version-specific changes over broad parser-wide refactors unless the file format behavior is clearly shared.
- If changing export layout, verify both:
  - Native CLI output directory structure.
  - Browser ZIP contents generated from `/output`.

---
> Source: [OpenYSM/YSMParser](https://github.com/OpenYSM/YSMParser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
