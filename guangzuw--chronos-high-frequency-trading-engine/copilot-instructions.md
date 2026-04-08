## chronos-high-frequency-trading-engine

> This file provides project-specific instructions and context for Gemini to ensure all interactions and code generations align with the Chronos HFT Engine's architectural and performance goals.

# Chronos HFT Engine: Project Context

This file provides project-specific instructions and context for Gemini to ensure all interactions and code generations align with the Chronos HFT Engine's architectural and performance goals.

## Project Overview
Chronos is an ultra-low-latency Limit Order Book (LOB) engine built with **C++20**. It is designed for high-frequency trading (HFT) environments where performance and deterministic execution are critical.

### Key Technologies
- **Language:** C++20 (utilizing `std::span`, `std::pmr`, and modern concurrency).
- **Messaging:** ZeroMQ (ZMQ) over TCP/IPC for low-latency microservice communication.
- **AI Integration:** ONNX Runtime for real-time, pre-trade risk validation.
- **Data Structures:** Lock-free and cache-aligned structures.
- **Build System:** CMake (with CppCheck static analysis and GTest).
- **Deployment:** Docker (multi-stage builds, CPU pinning).

### Architecture
- **Microservices:** Decoupled services (Gateway, Risk Engine, Matching Engine, Persistence).
- **Symbol Sharding:** Horizontal scaling by dedicating engine instances to specific asset classes.
- **Zero-Allocation:** Use of `std::pmr` and custom memory pools on the hot path.

## Building and Running

### Local Build
```bash
# Configure and build
cmake -B build -S . -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Run unit tests
cd build && ctest --output-on-failure
```

### Docker Build
```bash
docker build -t chronos-hft .
docker run --rm chronos-hft
```

## Development Conventions

### Coding Standards
- **Modern C++:** Strictly use C++20 features. Prefer `std::span` for zero-copy views.
- **Performance First:** Avoid allocations on the matching hot path. Use `std::pmr::monotonic_buffer_resource` or pre-allocated pools.
- **Cache Efficiency:** Align critical structs (e.g., `Order`, `Trade`) to 64-byte cache lines.
- **Type Safety:** Use `enum class` for all constants (OrderSide, OrderStatus).
- **Error Handling:** Avoid exceptions on the hot path; use return codes or `std::optional`/`std::expected`.

### Testing & Quality
- **Unit Testing:** All new logic must have corresponding tests in the `tests/` directory using **Google Test**.
- **Static Analysis:** Ensure code passes **CppCheck** scans (integrated into CMake).
- **Documentation:** Maintain `Architecture.md` and `Detailed_Roadmap.md` as the source of truth for design decisions.

## Roadmap Reference
Refer to `Detailed_Roadmap.md` for the specific implementation tasks. Current focus is on **Phase 1: Foundation (Core Data Structures & Memory)**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GuangzuW)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GuangzuW)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
