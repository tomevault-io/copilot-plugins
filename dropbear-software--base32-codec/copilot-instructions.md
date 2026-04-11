## base32-codec

> This project is a Dart package named `base32_codec`. It provides a flexible and efficient way to encode and decode Base32 data, following the conventions of `dart:convert`. The package is written in pure Dart and has no platform-specific dependencies.

# Project Overview

This project is a Dart package named `base32_codec`. It provides a flexible and efficient way to encode and decode Base32 data, following the conventions of `dart:convert`. The package is written in pure Dart and has no platform-specific dependencies.

## Key Features

*   **Multiple Base32 Variants:** Supports three popular Base32 alphabets:
    *   [RFC 4648](https://datatracker.ietf.org/doc/html/rfc4648) (the standard)
    *   [RFC 4648 "Hex"](https://datatracker.ietf.org/doc/html/rfc4648#section-7)
    *   [Crockford's Base32](http://www.crockford.com/base32.html)
*   **Stream-Based Conversion:** The API is compatible with Dart's `Stream` and `Codec` interfaces, making it suitable for processing large data sets without loading everything into memory.
*   **Synchronous Conversion:** For smaller data, the library provides simple `encode` and `decode` methods.

# Building and Running

This is a Dart library, so there is no main executable to run. However, you can run the tests and benchmarks to verify its functionality and performance.

## Running Tests

The project uses the `test` package for testing. To run the tests, use the following command:

```bash
dart test
```

## Running Performance Benchmarks

Performance benchmarks are included to measure the speed of encoding and decoding operations across different Base32 variants and conversion types (synchronous and asynchronous). To run them, execute the following command from the project root:

```bash
dart run benchmark/base32_codec_benchmark.dart
```

# Development Conventions

## Code Style

The project follows the standard Dart and Flutter linting rules, as defined in the `analysis_options.yaml` file. The `lints` package is used to enforce these rules.

## Testing

The project has a comprehensive test suite in the `test` directory. The tests cover all three Base32 variants and include test vectors from RFC 4648 to ensure correctness. The tests also cover the stream-based (chunked) conversion logic.

## Benchmarking

Performance benchmarks are implemented using the `benchmark_harness` package. They cover synchronous and asynchronous encoding/decoding for all three Base32 variants, providing a way to track performance and prevent regressions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dropbear-software)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dropbear-software)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
