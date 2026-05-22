## meo-3-open-service

> This repository contains the MEO 3 open edge service for IoT education hardware. Main service code lives under `src/main/java/org/thingai/app/meo`, with HTTP routes in `api/`, MQTT handling in `handler/mqtt/`, device logic in `handler/device/`, telemetry in `handler/telemetry/`, and LAN discovery in `service/`.

# Repository Guidelines

## Project Structure & Module Organization

This repository contains the MEO 3 open edge service for IoT education hardware. Main service code lives under `src/main/java/org/thingai/app/meo`, with HTTP routes in `api/`, MQTT handling in `handler/mqtt/`, device logic in `handler/device/`, telemetry in `handler/telemetry/`, and LAN discovery in `service/`.

Shared DTOs, entities, constants, callbacks, and abstract handlers live in the separate `meo-common/` Gradle project under `meo-common/src/main/java/org/thingai/meo/common`. Bundled platform dependencies are stored in `libs/`. Developer notes are in `docs/`, and infrastructure setup scripts are at the repo root, such as `install-meo-linux.sh`.

## Build, Test, and Development Commands

The Gradle wrapper files may not be executable in every checkout; use `bash ./gradlew` if `./gradlew` is denied.

- `bash ./gradlew compileJava`: compile the open service.
- `bash ./gradlew test`: run service tests when present.
- `cd meo-common && bash ./gradlew build`: build the shared common module jar.
- `bash ./gradlew clean`: remove generated build output.

Use a supported Java version for Gradle, preferably JDK 17 or 21. Very new JDKs may fail before project compilation.

## Coding Style & Naming Conventions

Use Java with 4-space indentation. Keep package names under `org.thingai.app.meo` for service code and `org.thingai.meo.common` for shared contracts. Follow existing class prefixes such as `MDevice`, `MService`, `MMqtt`, and `Meo...`. Public API handlers should remain thin and delegate business logic to handler classes.

Prefer explicit DTO/entity classes over raw JSON maps for stable service contracts. Keep comments short and useful, especially around hardware protocol details.

## Testing Guidelines

JUnit 5 is configured in both Gradle builds, but no tests are currently checked in. Add tests under `src/test/java` or `meo-common/src/test/java`, mirroring the production package. Name tests with `*Test.java`, for example `MDeviceHandlerTest.java`. Focus coverage on protocol parsing, device registration, MQTT topic handling, and DAO-backed behavior.

## Commit & Pull Request Guidelines

Recent history uses Conventional Commit-style messages, for example `feat: add ResponseDeviceInfo class` and `refactor: unify device discovery handler hierarchy`. Keep commits small and scoped.

Pull requests should include a short purpose statement, implementation notes, test results, and linked issues when relevant. For UI-facing changes, include screenshots or recordings. For hardware or MQTT changes, document topic names, payload examples, and any required Node-RED, Mosquitto, Avahi, or InfluxDB setup.

## Security & Configuration Tips

Do not commit local database files, credentials, tokens, or generated device keys. Treat MQTT broker URLs, InfluxDB tokens, Wi-Fi details, and device transmit keys as configuration, not source constants.

---
> Source: [MEO-3/meo-3-open-service](https://github.com/MEO-3/meo-3-open-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
