## web-app-starter

> This document provides context for the C++23 web application starter project.

# Gemini Context: C++ Web App Starter

This document provides context for the C++23 web application starter project.

## Project Overview

This project is a C++23 web application starter kit that uses `CMake` for building. It includes a web server, various support executables, and a testing suite.

The project is structured with separate directories for source code (`src`), include files (`include`), and data (`data`). It also includes directories for HTML (`html`) and plugins (`plugins`).

A bash script named `mk` is provided to simplify common development tasks like building, testing, and running the application.

## Build Instructions

The project is built using `CMake` and a C++23 compatible compiler. The `mk` script automates the build process.

- **Initialize the build:**
  ```bash
  ./mk init
  ```

- **Build the project:**
  ```bash
  ./mk build
  ```

- **Clean the build:**
  ```bash
  ./mk clean
  ```

- **Remove the build directory:**
  ```bash
  ./mk clobber
  ```

## Running the Application

- **Run the server in the foreground:**
  ```bash
  ./mk run
  ```

- **Start the server in the background:**
  ```bash
  ./mk start
  ```

- **Shut down the server:**
  ```bash
  ./mk shutdown
  ```

- **Check server status:**
  ```bash
  ./mk status
  ```

## Testing

The project has two test suites: `unit_tests` and `api_tests`. The `mk` script can be used to run them.

- **Run unit tests:**
  ```bash
  ./mk test
  ```

- **Run API tests (this will start the server if needed):**
  ```bash
  ./mk api
  ```

## Source Code

The main source code for the application is located in the `src` directory. Header files are in the `include` directory. The project is organized into several executables:

- `webapp-server`: The main web application server.
- `support`: A support executable.
- `supervisor`: A supervisor executable.
- `unit_tests`: The unit test runner.
- `api_tests`: The API test runner.

## Dependencies

The project uses several third-party libraries as dependencies:

- `spdlog`: For logging.
- `nlohmann_json`: For JSON manipulation.
- `Catch2`: For testing.
- `libsodium`: For cryptography.
- `termio`: For terminal I/O.
- `money`: For handling monetary values.
- `domainkeys`: For DomainKeys Identified Mail (DKIM).
- `quickkv`: For key-value storage.

These dependencies are managed through `CPM.cmake` and `find_package` in the `CMakeLists.txt` file.

## Other Commands

The `mk` script provides several other useful commands:

- **`./mk all`**: Pulls latest code, cleans, initializes, builds, and runs all tests.
- **`./mk format`**: Formats the C++ code using `clang-format`.
- **`./mk watch`**: Watches for file changes and automatically builds and runs unit tests.
- **`./mk watch_api`**: Watches for file changes and automatically builds and runs unit and API tests.
- **`./mk help`**: Displays a list of all available commands.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darrylwest) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
