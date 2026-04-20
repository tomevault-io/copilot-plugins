## beon-widget-sdk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Beon Widget SDK is a Flutter-based embeddable chat widget for real-time customer communication. It connects to the Beon backend API and supports WebSocket (Laravel Reverb) for real-time messaging. Primary target is Flutter Web.

## Build Commands

```bash
flutter pub get                    # Get dependencies
flutter run -d chrome              # Run on web (primary target)
flutter build web                  # Build for web release
flutter analyze                    # Analyze code
flutter test                       # Run all tests
flutter test test/widget_test.dart # Run single test file
dart pub publish --dry-run         # Validate before publishing
```

## Architecture

### Data Flow

1. `BeonChatWidget` creates a `ProviderScope` with `configProvider` override
2. `visitorProvider` initializes local storage, then `AuthService` for visitor identity
3. `channelConfigProvider` fetches server-side widget configuration (colors, pre-chat fields)
4. `effectiveConfigProvider` merges user config with API config (user values take precedence)
5. `ChatStateNotifier` loads cached messages, connects WebSocket, and fetches message history
6. Real-time messages arrive via `ReverbService` (WebSocket) or `PollingService` (fallback)

### Provider Hierarchy

```
configProvider (user-supplied BeonConfig)
    ↓
effectiveConfigProvider (merged with API ChannelConfig)
    ↓
┌───────────────────┬────────────────────────┐
↓                   ↓                        ↓
themeProvider   visitorProvider      chatStateProvider
                    ↓                        ↓
              authServiceProvider    reverbServiceProvider
                                     pollingServiceProvider
```

### Key Patterns

- **Optimistic Updates**: Messages appear immediately with `MessageStatus.sending`, updated on server response
- **Message Deduplication**: Uses message ID and content matching for WebSocket confirmations
- **Conversation UID**: Single UID per conversation, generated on first message if none exists
- **Web Conditional Imports**: `script_loader.dart` → `script_loader_web.dart` / `script_loader_stub.dart`

### API Integration

Base URL: `https://v3.api.beon.chat/api/widget/v5`

Key endpoints:
- `GET /channels/validate/{apiKey}` - Fetch channel config (colors, pre-chat fields, position)
- `GET /messages?uid={conversationUid}&page={n}` - Paginated message history
- `POST /message/send` - Send message with metadata (fingerprint, device info)

### WebSocket (Laravel Reverb)

- Host: `wss://v3.api.beon.chat:443`
- App Key: `t2vthpcpevqgpzw70dku`
- Channel: `conversation_id_{id}` (public channel)
- Events: `conversations_message` for new/updated messages

### State Classes

- `ChatState`: Immutable state with messages, loading flags, pagination info
- `WidgetState`: UI state (isOpen, currentView, unreadCount)
- `Visitor`: User identity with optional name/phone/email from pre-chat form

## Testing

Tests use `flutter_test` and `mocktail`. Current tests cover:
- `BeonConfig` parsing and defaults
- `Message` JSON parsing and equality
- `Visitor` model and pre-chat completion
- `Validators` for form fields

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/unlimited-software-house) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
