## terrible-hashes

> `terrible-hashes` is a Rust library that implements a collection of deliberately ineffective and "bad" hash functions and checksum algorithms. It is intended for educational purposes, satire, or perhaps as a warning of what *not* to do when implementing hashing logic.

# Project: terrible-hashes

## Overview
`terrible-hashes` is a Rust library that implements a collection of deliberately ineffective and "bad" hash functions and checksum algorithms. It is intended for educational purposes, satire, or perhaps as a warning of what *not* to do when implementing hashing logic.

**Warning:** As stated in the project documentation, do not use these algorithms for any production or security-critical purpose.

## Key Technologies
*   **Language:** Rust (Edition 2024)
*   **Build System:** Cargo
*   **Dependencies:**
    *   `image` & `imageproc` (Optional, used for the `image-bitshift` feature)

## Architecture & Modules
The project is structured as a library crate with two main modules exported in `src/lib.rs`:

*   **`src/hashes/`**: Contains implementations of bad hash functions.
    *   `tripple_string_hash.rs`: Implements `triple_hash`.
    *   `equality_hashers.rs`: Likely contains equality-based hash logic.
*   **`src/checksums/`**: Contains implementations of bad checksums.
    *   `image_bitshift.rs`: A checksum implementation that bizarrely converts data into an image, applies a gaussian blur, and sums pixel weights.
    *   `cosmological_constants.rs`: Implementation using cosmological constants.

## Building and Running

### Prerequisites
*   Rust and Cargo installed.

### Build
To build the project with default features:
```bash
cargo build
```

### Testing
Run the project's tests (if any exist yet):
```bash
cargo test
```

### Feature Flags
The project uses Cargo features to control dependencies and included modules.
*   `default`: Enables `all`.
*   `all`: Enables all sub-features.
*   `image-bitshift`: Enables the `image` and `imageproc` dependencies for the specific checksum module.
*   `equality-hashers`: Enables equality-based hashers.
*   `tripple-string-hash`: Enables the triple string hash module.
*   `cosmological_constants`: Enables the cosmological constants module.

## Development Conventions
*   **Code Style:** Standard Rust formatting (rustfmt) is expected.
*   **Documentation:** Modules and functions are documented with warning labels advising against their usage.
*   **Anti-Patterns:** The codebase intentionally employs cryptographic and algorithmic anti-patterns (e.g., simplistic bit shifting, magic numbers, irrelevant processing like image blurring for data checksums).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hschimke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hschimke)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
