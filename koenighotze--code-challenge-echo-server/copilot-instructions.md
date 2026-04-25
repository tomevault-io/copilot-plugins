## code-challenge-echo-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an implementation of RFC 862 - Echo Protocol using Kotlin with minimal dependencies. The server listens on port 7, accepts TCP connections, and echoes back any data received from clients.

## Build and Development Commands

### Build
```bash
./gradlew build
```

### Run Tests
```bash
./gradlew test
```

### Run a Single Test
```bash
./gradlew test --tests "ClassName.testMethodName"
# Example:
./gradlew test --tests "EchoRequestTest.it_should_read_and_echo_single_line"
```

### Run the Application
```bash
./gradlew run
```

### Build Executable JAR
```bash
./gradlew jar
```

The JAR will be created in `build/libs/` and can be run with:
```bash
java -jar build/libs/echo-server-1.0-SNAPSHOT.jar
```

## Architecture

### Core Components

**EchoServer** (`src/main/kotlin/org/koenighotze/echoserver/EchoServer.kt`)
- Main server class that binds to port 7 and accepts incoming TCP connections
- Uses a cached thread pool (`ExecutorService`) to handle multiple concurrent connections
- Tracks all in-flight requests in a synchronized collection
- Provides graceful shutdown that closes the server socket, signals all active requests to shut down, and waits for threads to complete

**EchoRequest** (`src/main/kotlin/org/koenighotze/echoserver/EchoRequest.kt`)
- Handles individual client connections
- Reads data line-by-line from the client and writes it back (echo)
- Implements socket timeout (5 seconds) with keep-alive messages ("Are you alive?")
- Uses an `AtomicBoolean` flag to support graceful shutdown mid-connection

**Main** (`src/main/kotlin/org/koenighotze/echoserver/Main.kt`)
- Entry point that creates the server on port 7
- Sets up a console watcher thread that allows typing 'Q' to gracefully shutdown
- Falls back to SIGINT/Ctrl+C if no interactive console is available

### Concurrency Model

The server uses Java's thread pool executor to handle multiple concurrent connections:
- Main thread listens for new connections via `ServerSocket.accept()`
- Each accepted connection is submitted to the executor as a new task
- Each task creates an `EchoRequest` instance to handle the client connection
- In-flight requests are tracked so the server can signal them during shutdown

### Shutdown Flow

1. Server socket is closed (stops accepting new connections)
2. All tracked `EchoRequest` instances are signaled via `shutdown()`
3. Executor waits up to 10 seconds for clean termination
4. Remaining threads are forcefully shut down via `shutdownNow()`

## Testing

Tests use JUnit 5 with Mockito for mocking. Key test files:
- `EchoRequestTest.kt`: Tests the echo request handler, including timeout behavior, shutdown signals, and stream management
- `EchoServerTest.kt`: Placeholder tests for server-level behavior (mostly incomplete)

The codebase includes several `TODO()` tests that need implementation, particularly around socket timeout handling.

## Package Structure

Note: There is a package naming inconsistency in the codebase:
- Main classes use: `org.koenighotze.org.koenighotze.echoserver`
- Tests use: `org.koenighotze.setupverification.org.koenighotze.echoserver`

This duplication should be addressed when refactoring.

## Implementation Progress

This is a work-in-progress following a step-by-step tutorial:
- Step 1: Basic server startup and connection acceptance (DONE)
- Step 2: Multiple concurrent connections (DONE)
- Step 3: Echo data back to clients (DONE)
- Step 4: Add TCP/UDP command line flag (TODO)
- Step 5: Clean shutdown (PARTIALLY DONE - shutdown works but may need refinement)

## Java/Kotlin Version

The project uses:
- Kotlin 2.2.20
- JVM Toolchain 21 (Java 21)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/koenighotze) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
