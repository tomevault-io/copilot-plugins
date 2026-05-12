## androclaw

> **Built by Pranav Patil**

# AndroClaw

**Built by Pranav Patil**

AndroClaw is a powerful AI-powered Android assistant that runs directly on your device. It connects to multiple LLM providers and executes real actions on your phone — from sending messages and making calls to searching the web, reading files, taking notes, and controlling other apps.

## Architecture

- **Language:** Kotlin
- **UI:** Jetpack Compose + Material 3
- **DI:** Hilt (Dagger)
- **Database:** Room (SQLite) with versioned migrations
- **Networking:** OkHttp + Retrofit + Jsoup
- **API:** Multi-provider LLM support with streaming
- **Min SDK:** 29 (Android 10)
- **Target SDK:** 35

## Project Structure

```
app/src/main/java/com/androclaw/
├── api/
│   ├── provider/           # Multi-LLM provider abstraction
│   │   ├── LlmProvider.kt       # Provider interface + streaming types
│   │   ├── ClaudeProvider.kt     # Anthropic Claude (SSE streaming)
│   │   ├── OpenAIProvider.kt     # OpenAI + compatible APIs (Groq, OpenRouter)
│   │   ├── GeminiProvider.kt     # Google Gemini
│   │   └── ProviderRegistry.kt   # Provider management
│   ├── ClaudeRepository.kt      # Agentic loop, tool orchestration, streaming
│   ├── ClaudeApiService.kt      # Retrofit interface (legacy, kept for compat)
│   ├── ClaudeResponseParser.kt  # Custom JSON parser
│   ├── ToolDefinitions.kt       # All tool schemas (30+ tools)
│   └── models/ClaudeModels.kt   # Request/response DTOs
├── db/
│   ├── AndroClawDatabase.kt     # Room DB with migrations v2→v5
│   ├── ConversationEntity.kt    # Chat conversations
│   ├── MessageEntity.kt         # Chat messages
│   ├── MemoryEntity.kt          # Persistent AI memory
│   ├── NoteEntity.kt            # User notes
│   └── SkillEntity.kt           # Custom slash command skills
├── di/
│   └── AppModule.kt             # Hilt dependency injection
├── service/
│   ├── AndroClawAccessibilityService.kt  # UI automation + screenshots
│   ├── FloatingButtonService.kt          # Overlay floating button
│   ├── NotificationReaderService.kt      # Read notification content
│   ├── OverlayChatManager.kt             # Overlay chat state
│   └── BootReceiver.kt                   # Auto-start on boot
├── tools/                    # 30+ tool handlers
│   ├── ToolExecutor.kt              # Tool router/dispatcher
│   ├── WebSearchToolHandler.kt      # Web search (DuckDuckGo + Google)
│   ├── WebFetchToolHandler.kt       # URL content reader
│   ├── MemoryToolHandler.kt         # Persistent memory across sessions
│   ├── NotesToolHandler.kt          # Local note-taking
│   ├── SkillToolHandler.kt          # Custom slash commands
│   ├── FileToolHandler.kt           # File explorer (find/read/tree/grep/glob)
│   ├── LocationToolHandler.kt       # GPS + reverse geocoding
│   ├── SmsReaderToolHandler.kt      # Read SMS history
│   ├── CallLogToolHandler.kt        # Call history
│   ├── ScreenshotToolHandler.kt     # Screenshot + vision analysis
│   ├── TtsToolHandler.kt            # Text-to-speech
│   ├── AppLaunchToolHandler.kt      # App launcher with fuzzy matching
│   ├── AutoScrollToolHandler.kt     # Reels/Shorts auto-scroll
│   └── ... (15+ more handlers)
├── ui/
│   ├── ChatScreen.kt + ChatViewModel.kt
│   ├── SettingsScreen.kt + SettingsViewModel.kt
│   ├── OnboardingScreen.kt
│   └── components/ (InputBar, MessageBubble, ChatHistoryDrawer, etc.)
└── utils/
    ├── Constants.kt              # System prompt, prefs keys, tool names
    ├── PermissionHelper.kt       # Runtime permission management
    └── VoiceInputManager.kt      # Speech-to-text
```

## Features

### Multi-Provider LLM Support
- **Anthropic Claude** (Opus 4.6, Sonnet 4.6, Haiku 4.5)
- **OpenAI** (GPT-4o, GPT-4o Mini, GPT-4 Turbo, GPT-3.5)
- **Google Gemini** (2.5 Pro, 2.5 Flash, 2.0 Flash)
- **Groq** (Llama 3.3 70B, Llama 3.1 8B, Mixtral, Gemma 2)
- **OpenRouter** (access to 100+ models via single API)
- Any OpenAI-compatible endpoint

### Streaming Responses
- Server-Sent Events (SSE) streaming for real-time token-by-token output
- Works with Claude, OpenAI, and Gemini providers
- Fallback to non-streaming for providers/models that don't support it

### 30+ Device Tools
| Category | Tools |
|----------|-------|
| **Communication** | send_sms, read_sms, make_phone_call, call_log, send_whatsapp, send_email, get_contacts |
| **Web Intelligence** | web_search, web_fetch, browse_web |
| **Files** | file_manager (find/open/share/list/info/recent/read/tree/grep/glob) |
| **Memory & Notes** | memory (save/recall/search), notes (create/read/update/delete/list/search) |
| **Apps** | open_app, list_apps, control_app_ui |
| **Media** | media_control, take_screenshot (with vision analysis), text_to_speech |
| **Device** | device_info, toggle_setting, brightness_control, get_location |
| **Productivity** | create_calendar_event, set_reminder, set_alarm, clipboard, notifications (with read) |
| **Social** | auto_scroll_feed (Reels, Shorts, TikTok), share_content |
| **Custom** | skills (create/list/run custom slash commands) |

### Skills System
Users can create custom slash commands:
- `/morning` → "Check weather, read my notifications, list today's calendar events"
- `/commute` → "Get my location, search for traffic conditions to [work address]"
- Skills are stored in the local database and persist across sessions

### Agentic Loop
- Up to 10 tool-use iterations per request
- Automatic tool chaining for complex multi-step tasks
- Vision support: screenshots sent as base64 images to the LLM
- Memory auto-injection: saved memories appended to system prompt

### Security
- API keys stored in EncryptedSharedPreferences (AES-256-GCM)
- Runtime permission management for all sensitive operations
- Accessibility service required for UI automation features
- NotificationListenerService for notification reading (user must grant access)

## Key Commands

```bash
# Build
./gradlew assembleDebug

# Install
./gradlew installDebug

# Run tests
./gradlew test
```

## Adding a New Tool

1. Create `tools/MyToolHandler.kt` — `@Singleton` class with `suspend fun execute(input: Map<String, Any>): String`
2. Add tool definition in `api/ToolDefinitions.kt`
3. Register in `tools/ToolExecutor.kt` (constructor + when branch)
4. Add tool name to `utils/Constants.kt` → `ALL_TOOL_NAMES` + system prompt guidance
5. Add `describeToolCall` entry in `api/ClaudeRepository.kt`
6. If new permissions needed: update `AndroidManifest.xml` + `utils/PermissionHelper.kt`

## Adding a New LLM Provider

1. Implement `api/provider/LlmProvider.kt` interface
2. Register in `api/provider/ProviderRegistry.kt` init block
3. For OpenAI-compatible APIs, just instantiate `OpenAIProvider` with custom `baseUrl` and `supportedModels`

## Database Migrations

Room migrations are defined in `AndroClawDatabase.kt`:
- v2→v3: memories table
- v3→v4: notes table
- v4→v5: skills table

Always add proper migrations. `fallbackToDestructiveMigration()` is kept as last resort only.

---
> Source: [AndroClawD/AndroClaw](https://github.com/AndroClawD/AndroClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
