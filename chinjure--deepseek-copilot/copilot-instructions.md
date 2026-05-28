## deepseek-copilot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
./gradlew build          # Build the plugin
./gradlew runIde         # Launch a sandboxed IntelliJ instance with the plugin loaded
./gradlew buildPlugin    # Produce the distributable .zip
```

Gradle 8.12, Java 17, IntelliJ Platform Gradle Plugin 2.3.0.

## Architecture

This is an IntelliJ IDEA plugin that provides AI-powered inline code completion via the DeepSeek Chat Completions API. Ghost text appears at the cursor position; Tab accepts it.

**Core flow:**

1. `DeepSeekCopilotStartupActivity` (postStartupActivity) hooks `EditorFactory` to register every editor with the completion service and listens for new/released editors.
2. `DeepSeekCompletionService` manages per-editor state via `EditorState` (stored in a `ConcurrentHashMap<Editor, EditorState>`). On each caret move or document change, it debounces a completion request (`ScheduledExecutorService`, default 300ms). Stale requests are discarded via an incrementing `requestId` counter.
3. The service extracts prefix/suffix text from the document, calls `DeepSeekApiClient.getCompletion()`, then displays results as ghost text using IntelliJ's `InlayModel` (first line as inline inlay at cursor, subsequent lines as block inlays below the current line).
4. `GhostTextRenderer` is an `EditorCustomElementRenderer` that paints gray text using `EditorColors.READONLY_BACKGROUND_COLOR`.
5. Tab acceptance: the service intercepts `IdeActions.ACTION_EDITOR_TAB` globally in its constructor via `EditorActionManager.setActionHandler()`. When a completion is active, Tab inserts the ghost text; otherwise it delegates to the original handler. The original handler is restored on `dispose()`.

**API layer:** `DeepSeekApiClient` uses `java.net.http.HttpClient` to call `POST /v1/chat/completions` with the configured base URL. The system prompt instructs the model to return only code (no explanations, no markdown). `parseResponse()` strips markdown code fences if present. Config: `deepseek-chat` model, temp 0.1, max_tokens 256, stop `\n\n\n`.

**Settings:** `DeepSeekSettingsState` extends `PersistentStateComponent`, persisted to `DeepSeekCopilot.xml`. Editable via Settings → Tools → DeepSeek Copilot (`DeepSeekSettingsConfigurable`). Fields: apiKey, apiBaseUrl, model, debounceDelayMs, maxPrefixLines, maxSuffixLines, enabled.

**Key IntelliJ Platform extension points used:**
- `applicationService` (for the singleton services)
- `applicationConfigurable` (for the settings UI)
- `postStartupActivity` (to wire editor listeners on startup)

## Plugin registration

Defined in `src/main/resources/META-INF/plugin.xml`. Depends only on `com.intellij.modules.platform` (works with any IntelliJ-based IDE, not language-specific).

---
> Source: [Chinjure/deepseek-copilot](https://github.com/Chinjure/deepseek-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
