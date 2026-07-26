## java-telegram-bot-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a JVM library for the [Telegram Bot API](https://core.telegram.org/bots/api), published to Maven Central as `com.github.pengrad:java-telegram-bot-api`. It provides type-safe wrappers for all Telegram Bot API methods and models. The codebase is mixed **Java and Kotlin** — the original core is Java, while newer request/model/response classes and the idiomatic DSL layer are written in Kotlin (Kotlin Gradle plugin 2.3.21).

## Build and Test Commands

```bash
# Build
./gradlew build

# Run all tests (requires env vars - see Testing section)
./gradlew clean check

# Run PR tests (excludes long-running integration tests:
# Base64Test, CheckTelegramAuthTest, PaymentsTest, TelegramBotTest, UpdatesListenerTest)
./gradlew clean check -PprTest

# Exclude an arbitrary test pattern
./gradlew test -PexcludeTests='**/SomeTest.class'

# Run a specific test class
./gradlew test --tests com.pengrad.telegrambot.TelegramBotTest

# Run a specific test method
./gradlew test --tests com.pengrad.telegrambot.TelegramBotTest.methodName

# Code coverage report
./gradlew jacocoTestReport
```

## Testing

Integration tests require a real Telegram bot. Set these environment variables:
- `TEST_TOKEN` — bot token
- `CHAT_ID` — test chat ID
- `GROUP_ID` — test group ID
- `PRIVATE_KEY` — for Passport decryption tests

Additional optional env vars: `TEST_PASSPORT_DATA`, `TEST_CALLBACK_QUERY`, `TEST_INLINE_QUERY`, etc.

## Architecture

This is a multi-module Gradle project:
- `library/` — the published library
- `sample/` — usage examples

### Core Data Flow

1. Instantiate a request object: `new SendMessage(chatId, "text")`
2. Execute synchronously: `bot.execute(request)` → returns typed response
3. Or execute asynchronously: `bot.execute(request, callback)`

Internally, `TelegramBotClient` serializes the request to an HTTP POST using Gson, calls `api.telegram.org/bot<token>/<MethodName>`, and deserializes the JSON response into a typed response class.

### Key Classes

- **`TelegramBot`** (`library/.../TelegramBot.java`) — Main entry point. Built via `new TelegramBot(token)` or `new TelegramBot.Builder(token).build()` for custom OkHttp/Gson config.
- **`TelegramBotClient`** (`library/.../impl/TelegramBotClient.java`) — OkHttp3 HTTP client; handles all serialization and deserialization.
- **`BaseRequest<T extends BaseRequest, R extends BaseResponse>`** (`library/.../request/BaseRequest.java`) — Generic base for all request classes. Uses generics to link each request type to its response type.
- **`BaseResponse`** (`library/.../response/BaseResponse.java`) — Base for all response classes. Contains `ok`, `errorCode`, `description`.

### Package Structure

- `request.*` — ~150 request classes, one per Telegram API method (e.g., `SendMessage`, `GetUpdates`, `AnswerCallbackQuery`). Sub-packages group newer features, e.g. `request.business.*`, `request.suggestedposts.*`.
- `model.*` — ~190 model classes for Telegram types (e.g., `Message`, `User`, `Chat`, `Update`), with many feature sub-packages (`model.business`, `model.gift`, `model.stars`, `model.story`, `model.checklist`, `model.reaction`, `model.paidmedia`, `model.botcommandscope`, etc.).
- `response.*` — Typed response wrappers (e.g., `SendResponse`, `GetUpdatesResponse`)
- `passport.*` — Telegram Passport decryption logic
- `login.*` — `CheckTelegramAuth` (Telegram Login widget verification)
- `constants.*` — shared enums/constants (e.g., `LivePeriod`)
- `utility.*` — `BotUtils` (webhook/update parsing helpers), Gson type adapters (`utility.gson`), and the Kotlin DSL layer (`utility.kotlin`, including `utility.kotlin.extension` request extensions)
- `impl.*` — Internal HTTP client (`TelegramBotClient`, `FileApi`) and update handlers (`UpdatesHandler`, `SleepUpdatesHandler`)

### Kotlin DSL Layer

In addition to the Java fluent builders, the library ships idiomatic Kotlin extension functions in `utility.kotlin.extension.request.*`. These hang off the `TelegramAware` interface (implemented by `TelegramBot`) and take a trailing `modifier: Request.() -> Unit` lambda, e.g. `bot.sendMessage(chatId, "text") { parseMode(ParseMode.HTML) }`. `RequestPreprocessor` is a hook applied to every outgoing request (default `EMPTY_REQUEST_PREPROCESSOR`).

### Update Processing

Two modes are supported:
- **Polling**: Call `bot.setUpdatesListener(updates -> { ... })` to start background polling via `getUpdates`.
- **Webhook**: Parse incoming webhook payloads with `bot.parseUpdate(json)`.

The `UpdatesHandler` / `SleepUpdatesHandler` classes manage the polling loop internally.

## Conventions

- Every Telegram API method maps 1:1 to a class in `request.*` and a class in `response.*`.
- Request classes use a fluent builder pattern: `new SendMessage(chatId, text).parseMode(ParseMode.HTML).replyMarkup(keyboard)`.
- All model and request classes follow Telegram API naming closely (snake_case field names deserialized by Gson into camelCase fields).
- New classes are increasingly written in Kotlin (often data classes / `KBaseRequest`-based requests); follow the language already used by sibling classes in the same package when adding code.
- The library targets Java 8 runtime compatibility (`sourceCompatibility`/`targetCompatibility = 1.8`), though JDK 11+ is required to build.

---
> Source: [pengrad/java-telegram-bot-api](https://github.com/pengrad/java-telegram-bot-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
