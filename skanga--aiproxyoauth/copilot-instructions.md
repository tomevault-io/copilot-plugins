## aiproxyoauth

> mvn clean package -DskipTests   # Build fat JAR → target/AIProxyOauth-1.0.0.jar

## Build Commands

```bash
mvn clean package -DskipTests   # Build fat JAR → target/AIProxyOauth-1.0.0.jar
mvn clean package               # Build with tests
mvn test                        # Run all tests
mvn test -Dtest=ClassName       # Run a single test class
mvn clean compile               # Compile only
```

**Run the proxy:**
```bash
java -jar target/AIProxyOauth-1.0.0.jar --port 8080
java -jar target/AIProxyOauth-1.0.0.jar --generate-key myapp   # Generate an API key
```

## Architecture Overview

AIProxyOauth is a Java 21 proxy that exposes OpenAI-compatible REST endpoints and routes requests to ChatGPT's internal Codex backend, translating between the OpenAI Chat Completions API format and the upstream Responses API format.

**Request flow:**
```
Client (OpenAI SDK) → Javalin HTTP server → Handler → CodexHttpClient (injects OAuth headers)
                                                    ↓
                               ChatCompletionsHandler: translates request to Responses API format,
                               receives SSE stream, translates back to Chat Completions format
```

**Key architectural components:**

| Component | Location | Responsibility |
|-----------|----------|----------------|
| `AIProxyOauth` | root | picocli entry point; wires all components |
| `ServerConfig` | config/ | Immutable record holding all runtime configuration |
| `ProxyServer` | server/ | Javalin setup, routing, API key enforcement |
| `ChatCompletionsHandler` | server/ | Bidirectional translation: Chat Completions ↔ Responses API |
| `ResponsesHandler` | server/ | Passthrough with normalization to upstream |
| `AuthManager` | auth/ | Thread-safe OAuth token loading and auto-refresh |
| `CodexHttpClient` | transport/ | Outbound HTTP with auth header injection |
| `SseParser` / `SseCollector` | sse/ | Line-by-line SSE parsing; collects stream into final response |
| `ModelResolver` | model/ | Multi-level cached model discovery (1h version, 5m model list) |
| `UsageTracker` | usage/ | Per-API-key token usage with `ConcurrentHashMap` + `LongAdder` |

## API Endpoints

- `GET /health` — liveness check
- `GET /v1/models` — model list (cached 5 minutes)
- `POST /v1/chat/completions` — main endpoint; translates to/from Responses API
- `POST /v1/responses` — passthrough to upstream Responses API
- `GET /v1/usage` — per-key usage stats (admin key sees all keys)

## Key Design Details

**API Translation (`ChatCompletionsHandler`):** Converts `messages[]` (system/user/assistant/tool roles), `tools`/`function_call`, `stream`, `reasoning_effort`, and token usage between the two formats. This is the most complex file (~561 lines).

**Streaming:** Each request runs on a virtual thread. SSE lines are parsed and forwarded in real time. For non-streaming requests, `SseCollector` buffers the SSE stream and emits a single JSON response.

**Auth refresh:** `AuthManager` uses a `ReentrantLock` to serialize refreshes. Tokens are refreshed if they expire within 5 minutes or if more than 55 minutes have elapsed since last refresh. OAuth config is read from `auth.json`.

**Model discovery fallback chain:** local Codex CLI binary → NPM registry → hardcoded version string. Results are cached with double-checked locking.

**API key enforcement:** Optional. Keys follow the format `sk-proxy-<32 hex chars>`. An admin key is designated at startup for viewing all usage stats. By default the server runs in open mode.

## Technology Stack

- **Java 21** — required (uses virtual threads)
- **Javalin 7 / Jetty 12** — HTTP server
- **Jackson** — JSON processing (uses `JsonNode` for dynamic manipulation)
- **picocli** — CLI argument parsing
- **java.net.http** — outbound HTTP client (no extra dependency)

---
> Source: [skanga/AIProxyOauth](https://github.com/skanga/AIProxyOauth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
