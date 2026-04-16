## vip

> This is the backend for the VIP project, a high-performance system designed using the **Drogon** C++ web framework. The project emphasizes modern C++20 features, including **Modules** and **Coroutines**, and utilizes the **Glaze** library for high-speed JSON serialization. The architecture follows a multi-layered approach: **Controller -> Service -> Repository -> Domain -> DTO**, ensuring a clean separation of concerns and maintainability.

# Project Overview: VIP Backend

## Description
This is the backend for the VIP project, a high-performance system designed using the **Drogon** C++ web framework. The project emphasizes modern C++20 features, including **Modules** and **Coroutines**, and utilizes the **Glaze** library for high-speed JSON serialization. The architecture follows a multi-layered approach: **Controller -> Service -> Repository -> Domain -> DTO**, ensuring a clean separation of concerns and maintainability.

## Technology Stack
- **Language:** C++26 (standard 26 specified in CMake)
- **Web Framework:** [Drogon](https://drogon.org/)
- **Serialization:** [Glaze](https://github.com/stephenberry/glaze)
- **Database:** PostgreSQL (managed through Drogon ORM)
- **Package Manager:** [vcpkg](https://vcpkg.io/)
- **Build System:** CMake
- **Other Libraries:** libsodium (for hashing)

## Architecture
- **Controllers:** Handle HTTP requests and routing. Located in `controllers/`.
- **Services:** Orchestrate business logic and manage data flow between repositories and controllers. Located in `services/`.
- **Repositories:** Abstract database operations using Drogon's CoroMapper. Located in `repositories/`.
- **Domains:** Business entities wrapping generated database models. Located in `domains/`.
- **DTOs:** Data Transfer Objects for JSON serialization/deserialization. Located in `dto/`.
- **Models:** Auto-generated database models (Drogon ORM). Located in `models/` (**DO NOT EDIT**).
- **Filters:** Middleware for authentication and request processing. Located in `filters/`.

## Key Development Conventions
- **C++20 Modules:** All core logic is encapsulated in header
- **Coroutines:** Asynchronous operations use `drogon::Task<>` and `co_await`.
- **High Performance:** Glaze is preferred over JsonCpp for all serialization.
- **Brace Initialization:** Use `{}` consistently; omit empty braces for default initialization.
- **Model Immutability:** Never modify files in the `models/` directory.

## Building and Running
### Prerequisites
- CMake (>= 3.5)
- vcpkg (located at `.vcpkg-clion/vcpkg`)
- PostgreSQL

### Commands
- **Configure:** `cmake -B cmake-build-debug -S . -DCMAKE_TOOLCHAIN_FILE=.vcpkg-clion/vcpkg/scripts/buildsystems/vcpkg.cmake`
- **Build:** `cmake --build cmake-build-debug`
- **Run:** `./cmake-build-debug/vip`
- **Test:** `ctest --test-dir cmake-build-debug`

## TODO
- Complete implementation of Domain/Service/Repository layers for all remaining models.
- Implement comprehensive `AuthFilter` with dynamic exemptions from `config.yaml`.
- Finalize Product caching and search functionality.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bac-ptit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
