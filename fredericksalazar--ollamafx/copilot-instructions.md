## ollamafx

> Before any task, read `ARCHITECTURE.md` and `.ai/RULES.md`.

# AGENTS.md — OllamaFX

Before any task, read `ARCHITECTURE.md` and `.ai/RULES.md`.

## Commands

```bash
./gradlew compileJava   # fast verification (no jar, no shadow)
./gradlew build         # full build + shadowJar + dist zips
./gradlew run           # launch app locally
```

No tests exist (`test NO-SOURCE`); only `compileJava` matters for verification.

## Architecture (non-obvious)

- **MVC-lite**: Controllers (`@FXML`-backed) call **Singleton Managers** via `getInstance()`. No DI, no Spring.
- **Threading**: `App.getExecutorService()` (CachedThreadPool, daemon). All UI updates MUST use `Platform.runLater()`.
- **Streaming**: `OllamaManager.askModelStream()` sends raw HTTP POST to `/api/chat`, reads JSON lines. Uses `OllamaStreamHandler` (`Consumer<String>`) for streaming tokens. Signature includes an optional `Consumer<GenerationMetrics>` for capturing `eval_count`+`eval_duration` from the final `done` JSON.
- **Chat persistence**: `ChatSession` → Jackson JSON → `~/.OllamaFX/chats/{uuid}.json` via `ChatManager.saveChats()`.
- **Message model**: `ChatMessage` has `role` (`"user"`|`"assistant"`), `content`, `images` (base64 list, nullable).

## Styling

Use AtlantaFX CSS variables (`-color-fg-muted`, `-color-bg-subtle`, etc.). The main stylesheet is `src/main/resources/css/ollama_active.css`. Never hardcode hex colors in Java.

## Icons

Uses Ikonli Feather pack (`ikonli-feather-pack:12.3.1`). Create icons with:
```java
FontIcon icon = new FontIcon("fth-icon-name");
// or FontIcon icon = new FontIcon(Feather.ENUM);
```
Icon color is controlled via CSS `-fx-icon-color` on `.ikonli-font-icon`. Available icons: `fth-copy`, `fth-edit`, `fth-refresh-cw`, `fth-plus`, `fth-x`, `fth-book-open`, etc.

## I18n

`ResourceBundle.getBundle("messages")` from `src/main/resources/messages*.properties`. Get strings via `App.getBundle().getString("key")`. Three files: `messages.properties` (base), `messages_es.properties`, `messages_en.properties`.

## Chat UI rendering

- **User bubble**: `ChatController.addUserMessage()` — `Label` with CSS `chat-bubble chat-bubble-user`, inside right-aligned `VBox` → `HBox`. Footer toolbar uses CSS `chat-message-toolbar`.
- **Assistant bubble**: `ChatController.addAssistantMessage()` → `setupAssistantContent()` — `MarkdownOutput` (flexmark-based streaming renderer) + toolbar. During streaming, `updateLastMessage()` finds the last HBox and calls `MarkdownOutput.updateContent()`.
- **Action toolbar**: `HBox` with CSS class `chat-message-toolbar`, alignment `CENTER_LEFT`, spacing 10. Buttons are `fth-copy` + `fth-regenerate-cw` or `fth-edit` on user side.
- **Copy feedback**: changes `FontIcon` inline style to `-fx-icon-color: -color-success-fg` for 1s via `PauseTransition`.

## Message streaming flow

1. `sendMessage()` → `prepareSessionForMessage()` adds user bubble + `ChatMessage("user")`
2. `createAssistantPlaceholder()` adds `RingProgressIndicator`
3. `handleGenerationTask()` runs in background thread:
   - RAG query (if enabled)
   - `OllamaManager.askModelStream()` with `OllamaStreamHandler` callback
   - On first token: `RingProgressIndicator` replaced with `setupAssistantContent()`
   - Streaming: `updateLastMessage()` throttled to ~30ms
   - On complete: `addMetricsToLastMessage()` inserts a `.chat-metrics` label (11px mono, muted)
4. `regenerateResponse()` (line ~1166): cancels current, removes last assistant from session+UI, re-submits last user message

## Package map (high-signal only)

| Package | Role |
|---------|------|
| `controller/` | FXML-backed views (14 files). `ChatController` is the core (~1500 lines) |
| `manager/` | Business logic singletons (13). `OllamaManager` (API), `ChatManager` (CRUD), `RagManager` (LangChain4j) |
| `model/` | Data models. `ChatSession`, `ChatMessage`, `OllamaModel`, `GenerationMetrics` (record) |
| `ui/` | Custom JavaFX widgets. `MarkdownOutput`, `CodeBlockCard`, `ImagePreviewStrip` |
| `service/` | Cross-cutting: `UpdateManagerService`, `MarkdownService` |
| `util/` | `Utils`, `ImageUtils`, `SecurityUtils` |

## Gotchas

- **No module-info.java** — classpath mode, not JPMS modules.
- **`ChatController` is monolithic** (~1500 lines): mixes streaming, RAG, multimodal, and UI construction. Avoid adding more — extract where possible.
- **`OllamaManager.askModelStream`** uses raw HTTP + manual JSON, not ollama4j's built-in chat. The `activeStream` InputStream reference must be cleared in `finally`.
- **`ChatSession.getMessages()` returns mutable `ArrayList`** — removals from it during regenerate are safe.
- **I18n string injection**: in `processResources`, properties files are expanded with Gradle's `project.properties` — be careful with `${}` in property values.
- **JavaFX natives**: four architectures in runtimeOnly dependencies (linux, win, mac, mac-aarch64). ShadowJar bundles the current platform's natives at runtime.

---
> Source: [fredericksalazar/OllamaFX](https://github.com/fredericksalazar/OllamaFX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
