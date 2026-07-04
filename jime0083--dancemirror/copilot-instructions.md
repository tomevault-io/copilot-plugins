## dancemirror

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Code Change Guidelines

When modifying or adding code, always provide a summary after completing the work:

1. **Which file** - List the files that were added or modified
2. **Why** - Explain the purpose/reason for the change
3. **How** - Briefly describe what was implemented or changed

Do NOT show all the changed code inline. Provide a clear, organized summary instead.

## Project Overview

DanceMirror is a React Native (Expo) iOS app for comparing dance videos side-by-side. Users can display a reference video (from TikTok/Instagram URL or uploaded file) on the left and their own dance video on the right to check timing and form.

## Commands

```bash
# Development
npm start              # Start Expo dev server
npm run ios            # Run on iOS (requires prebuild)
npm run web            # Run in browser

# iOS Build
npx expo prebuild --platform ios    # Generate native iOS project
cd ios && pod install               # Install CocoaPods dependencies
npx expo run:ios                    # Build and run on iOS device/simulator

# Type checking
npx tsc --noEmit
```

## Architecture

### State Management
- **VideoContext** (`src/contexts/VideoContext.tsx`): Central state for both videos - URIs, playback position, duration, speed, sync offset
- **SubscriptionContext** (`src/contexts/SubscriptionContext.tsx`): Manages premium subscription state (ad-free mode)
- **LanguageContext** (`src/contexts/LanguageContext.tsx`): Manages app language (Japanese/English)

### Internationalization (i18n)
- Translation files in `src/i18n/translations.ts`
- Supported languages: Japanese (`ja`), English (`en`)
- Use `useLanguage()` hook to get `t()` function for translations
- Language can be changed in Settings screen

### Video Sources
- **Left video (reference)**: Can be either uploaded file (`sourceType: 'upload'`) or URL (`sourceType: 'url'`)
  - Upload: Full playback control via `expo-av`
  - URL: WebView embed for TikTok/Instagram/YouTube (limited control)
- **Right video (user's)**: Always uploaded file with full playback control

### Key Components
- `VideoPlayer`: Native video player using `expo-av` with seek control
- `WebViewPlayer`: Embeds external video URLs (converts to embed format)
- `SeekBar`: Synchronized seeking for both videos (only when both are uploaded)
- `ControlPanel`: Play/pause, speed (0.5x-2x), sync offset adjustment

### URL Sharing Flow
The `useSharedUrl` hook (`src/hooks/useSharedUrl.ts`) handles:
1. Deep links via `dancemirror://` URL scheme
2. Clipboard detection when app becomes active (auto-detects TikTok/Instagram/YouTube URLs)

### Screen Orientation
App is locked to landscape mode via `expo-screen-orientation` and `app.json` config.

## iOS Native Extensions

Share Extension files are in `ios/ShareExtension/` for sharing videos directly from TikTok/Instagram to the app. Requires manual Xcode configuration after prebuild.

## Monetization (Placeholder)

- Banner ad placeholder between videos
- Interstitial ad trigger every 10 minutes of playback
- Subscription toggle in SettingsScreen (test mode - no real IAP implemented yet)

---
> Source: [jime0083/DanceMirror](https://github.com/jime0083/DanceMirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
