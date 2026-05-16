## converso

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
npm start          # start Metro bundler
npm run ios        # build + launch on iOS Simulator (requires macOS + Xcode 15+)
npm run android    # build + launch on Android emulator / device
npm test           # run Jest suite
npm run lint       # ESLint
```

One-time iOS setup after `npm install`:
```sh
bundle install && cd ios && bundle exec pod install && cd ..
```

Run a single test file:
```sh
npx jest __tests__/translate.test.ts
```

## Architecture

**Everything lives in two files.** `App.tsx` is the entire application; `src/services/translate.ts` is the only service layer. There are no screens, no navigation, no state management library.

### Hands-free loop (`App.tsx`)

The core flow: Voice STT → `translateText` → TTS speak → Voice STT resumes.

All of this is wired inside a **single-run `useEffect` with `[]` deps**. Because the effect never re-runs, it can't close over stale state directly — every piece of state it needs is mirrored to a ref (e.g. `handsFreeRef`, `isSpeakingRef`). The pattern repeats throughout the file: a `useState` for rendering + a `useRef` updated by a sync effect to give the listener closure fresh values without tearing down Voice.

Key guard: `isSpeakingRef` prevents `startListening` from firing while TTS is active, avoiding self-echo loops. A 15 s safety timeout force-clears `isSpeaking` if the `tts-finish` event never fires.

### Translation service (`src/services/translate.ts`)

Calls the [MyMemory API](https://mymemory.translated.net/doc/spec.php) — no API key, ~5 k words/day free. Source language is **inferred** from the target via `inferSource` (currently only `en ↔ es`). Adding a third language requires updating that function. On any failure the function returns the original text so the hands-free loop keeps running.

To raise the daily limit to ~50 k words, set `CONTACT_EMAIL` at the top of `translate.ts`.

### Mocks & tests

`__mocks__/` provides manual mocks for all three native modules: `@react-native-voice/voice`, `react-native-tts`, and `react-native-safe-area-context`. `jest.config.js` maps all three to those mocks via `moduleNameMapper`.

> **Note:** `__tests__/translate.test.ts` was written against a LibreTranslate API shape and is out of sync with the current MyMemory implementation. Tests will fail or give false confidence until updated.

## Known gaps (from README)

- STT locale is hardcoded to `'en-US'` (`App.tsx` line 84) regardless of the target language toggle.
- No source-language detection; the app assumes the speaker's language is always the inverse of the target.
- Hands-free mode can loop on itself in loud environments; headphones recommended.

---
> Source: [jacquesme/Converso](https://github.com/jacquesme/Converso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
