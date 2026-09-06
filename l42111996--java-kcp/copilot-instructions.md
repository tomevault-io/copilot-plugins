## java-kcp

> This is a multi-module Maven project for a Netty-based Java KCP implementation. The root `pom.xml` aggregates four modules:

# Repository Guidelines

## Project Structure & Module Organization

This is a multi-module Maven project for a Netty-based Java KCP implementation. The root `pom.xml` aggregates four modules:

- `kcp-fec/`: forward error correction and Reed-Solomon code under `src/main/java/com/backblaze/erasure`.
- `kcp-base/`: core KCP protocol, channel management, CRC, and thread pool code under `src/main/java`.
- `kcp-example/`: runnable client/server examples under `src/main/java/test`.
- `kcp-lockStepSynchronization/`: lock-step synchronization examples and TPS tests.

Tests and benchmarks live in module `src/test/java` trees where present. Generated output belongs in `target/` and should not be committed.

## Build, Test, and Development Commands

- `mvn clean package`: builds all modules and jars.
- `mvn test`: compiles and runs the configured test phase across all modules.
- `mvn -pl kcp-base -am test`: tests `kcp-base` and any required upstream modules.
- `mvn -pl kcp-example -am package`: builds examples plus dependencies.
- `mvn -Prelease verify`: runs the release profile, including source/javadoc jars and signing; use only with proper release credentials.

Run examples from an IDE or Maven exec tooling after building, usually in server/client pairs such as `test.KcpRttExampleServer` and `test.KcpRttExampleClient`.

## Coding Style & Naming Conventions

Target Java 8 (`maven.compiler.release=8`) and UTF-8 source files. Follow the existing style: 4-space indentation, same-line braces, `PascalCase` classes, `camelCase` fields and methods, and uppercase constants. Keep public protocol APIs stable unless a compatibility change is intentional and documented. Prefer existing Netty `ByteBuf` ownership patterns, and release buffers explicitly when required.

## Testing Guidelines

There is no strong JUnit convention; several checks are standalone `main` classes or benchmarks. Add automated tests under the relevant module's `src/test/java`, mirror the tested package, and name correctness tests `*Test` and performance checks `*Benchmark`. For networking changes, validate with paired examples and note ports, packet loss settings, and observed behavior.

## Commit & Pull Request Guidelines

History uses concise messages such as `fix native fec error`, `update fec lib`, and `opt: ...`, sometimes with issue references like `#44`. Keep commits focused, use an imperative summary, and include `fix:`, `opt:`, or `update` when useful.

Pull requests should describe affected modules, behavioral or compatibility impact, validation commands run, and any linked issues. Include benchmark results for performance-sensitive KCP, FEC, or threading changes.

## Security & Configuration Tips

Do not commit Maven Central tokens, GPG material, IDE-local settings, or generated `target/` artifacts. Native FEC libraries live under `kcp-fec/src/main/java/com/backblaze/erasure/fecNative/native`; update them deliberately and document platform impact.

---
> Source: [l42111996/java-Kcp](https://github.com/l42111996/java-Kcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
