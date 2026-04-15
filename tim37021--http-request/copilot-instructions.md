## http-request

> This is a C++ project that implements an asynchronous, callback-based HTTP client using `libcurl`. The core of the project is an abstraction for HTTP transfers (`HttpTransfer` and `HttpTransferContext`) that allows for non-blocking network operations. The implementation uses modern C++ features (C++23) and standard tooling like CMake and Conan.

# Gemini Project Context

## Overview

This is a C++ project that implements an asynchronous, callback-based HTTP client using `libcurl`. The core of the project is an abstraction for HTTP transfers (`HttpTransfer` and `HttpTransferContext`) that allows for non-blocking network operations. The implementation uses modern C++ features (C++23) and standard tooling like CMake and Conan.

## Key Technologies

- **Language:** C++23
- **Build System:** CMake
- **Package Manager:** Conan
- **Core Libraries:**
  - `libcurl` for HTTP operations.
  - `abseil-cpp` for common utilities (logging, flags, synchronization).
  - `gtest` for unit testing.

## Project Structure

- `src/`: Contains the core source code for the HTTP client abstraction and implementation.
  - `network_manager.h`: Defines the core `HttpTransfer` and `HttpTransferContext` interfaces.
  - `curl_transfer.h`/`.cpp`: The `libcurl`-based implementation of the transfer interfaces.
  - `pipe.h`/`.cpp`: A thread-safe pipe for streaming data between threads.
  - `stream.h`/`.cpp`: Defines the `Readable` and `Writable` stream interfaces.
  - `main.cpp`: `download` executable (GET example).
  - `main2.cpp`: `upload` executable (POST example).
- `test/`: Contains unit tests for the project components.
  - `pipe_test.cpp`: Tests for the `Pipe` implementation.
- `build/`: The build output directory.

## Build and Test Commands

**Configure and Build:**
```bash
mkdir -p build
cd build
conan install .. --build=missing
cmake .. -DCMAKE_TOOLCHAIN_FILE=build/Release/generators/conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Release
cmake --build .
```

**Run Tests:**
```bash
cd build
ctest
```

## Architectural Notes

- The design is heavily asynchronous and callback-driven (`OnResponseCallback`, `OnDoneCallback`).
- `HttpTransfer` represents a single HTTP request/response lifecycle.
- `HttpTransferContext` manages multiple `HttpTransfer` objects and polls the underlying `curl_multi_handle`.
- The `Pipe` class is a key component for decoupling data producers (like a file reader or in-memory generator) from the network consumer (`libcurl`'s read callback). This enables true streaming of request and response bodies.
- Error handling was refactored to use `absl::StatusOr<HttpResponse>` in the `on_response` callback, ensuring the callback is always invoked, even on transport errors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tim37021)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tim37021)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
