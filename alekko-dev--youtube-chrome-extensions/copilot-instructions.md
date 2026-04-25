## youtube-chrome-extensions

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Briefly** is a Chrome extension (Manifest V3) that generates AI-powered summaries of YouTube videos by extracting transcripts and processing them with OpenAI's GPT models. This codebase was reverse-engineered from minified production code and rebuilt with modern TypeScript, React 19, and Vite 7.

## Build Commands

```bash
# Install dependencies
npm install

# Development build with hot reload
npm run dev

# Production build (outputs to dist/)
npm run build

# Preview production build
npm run preview
```

**After building**, load the extension in Chrome:
1. Navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked" and select the `dist/` folder
4. After code changes, click the reload icon on the extension card

## Architecture

### Chrome Extension Structure

This is a **Manifest V3** extension with three isolated execution contexts that communicate via Chrome's message passing API:

1. **Background Service Worker** (`src/background/index.ts`)
   - Orchestrates transcript extraction and OpenAI summarization
   - Manages per-tab badge state and system notifications
   - Tracks in-progress summaries in `chrome.storage.local`

2. **Content Script** (`src/content/index.ts`)
   - Injected into all `youtube.com` pages
   - Has direct DOM access to the YouTube page
   - Handles two types of messages:
     - `GET_TRANSCRIPT_DATA`: Clicks transcript button and extracts transcript from DOM
     - `SEEK_VIDEO`: Controls video playback (seek, play, scroll)
   - Cannot access Chrome extension APIs or popup state

3. **Popup UI** (`src/popup/`)
   - React application shown when clicking extension icon
   - Starts summaries by sending `START_SUMMARY` to the background worker
   - Listens for `SUMMARY_PROGRESS`, `SUMMARY_DONE`, and `SUMMARY_ERROR` messages
   - Has access to Chrome APIs but NOT to page DOM
   - Persists settings and viewed-summary state to `chrome.storage.local`

### Message Passing Pattern

Communication between popup and content script uses Chrome's message passing:

```typescript
// Popup sends message to content script
chrome.tabs.sendMessage(tabId, {
  type: 'SEEK_VIDEO',
  time: timeInSeconds
});

// Content script receives and responds
chrome.runtime.onMessage.addListener((message, _sender, sendResponse) => {
  if (message.type === 'SEEK_VIDEO') {
    // Manipulate video element
    sendResponse({ success: true });
  }
  return true; // Keep channel open for async response
});
```

### Data Flow

1. **User clicks "Summarize This Video"** in popup
2. **Popup** extracts video ID from active tab URL
3. **Popup → Background**: popup sends `START_SUMMARY` with `{ videoId, videoTitle, tabId, openaiApiKey, comfortableLanguages }`.
4. **YouTube Transcript Extraction** (`src/utils/youtube.ts` + `src/content/transcriptDom.ts`):
   - Sends `GET_TRANSCRIPT_DATA` message to content script
   - Content script clicks "Show transcript" button on YouTube
   - Waits for transcript panel to load
   - Extracts transcript segments from DOM (`ytd-transcript-segment-renderer` elements)
   - Parses timestamps (MM:SS or H:MM:SS format) and text
   - Returns `TranscriptEntry[]` with timestamped text
5. **OpenAI Summary Generation** (`src/utils/openai.ts`):
   - Formats transcript with timestamps
   - Calls OpenAI Chat Completions API (GPT-4o-mini)
   - Returns markdown-formatted summary with clickable timestamps
6. **Background → Storage & Badge**:
   - Stores summary under `summary-{videoId}` and clears `summaryInProgress`
   - Sets a green per-tab badge (`✓`) and optional notification when summary is ready
7. **Background → Popup**:
   - Sends `SUMMARY_PROGRESS`, `SUMMARY_DONE`, or `SUMMARY_ERROR` messages to any open popup
8. **Popup**:
   - Reads summaries from `chrome.storage.local` for the active video
   - Clears the `✓` badge for the active tab the first time the user views that summary
9. **User clicks timestamp** in summary
10. **Popup** sends `SEEK_VIDEO` message to content script
11. **Content script** seeks video, plays if paused, scrolls into view

### State Management

No external state library - popup state is in `App.tsx` using React hooks:
- `settings`: API keys loaded from and persisted to `chrome.storage.local`
- `summary`: Summary for the active video (includes `videoId`, title, content, timestamp, `viewedByUser`)
- `loading` / `loadingMessage`: Progress indicators driven by background messages
- `error`: String for error messages derived from structured error codes
- `showSettings`, `isExpanded`, `isYouTubeVideo`: Local UI state
- Background worker maintains `summaryInProgress` and per-video summaries in `chrome.storage.local`

## Key Implementation Details

### YouTube Transcript Extraction

**Critical**: The extension extracts transcripts by interacting with YouTube's native transcript UI:

1. Background worker calls `getYouTubeTranscript(videoId)` in `src/utils/youtube.ts`.
2. That helper sends `GET_TRANSCRIPT_DATA` to the content script.
3. Content script uses `src/content/transcriptDom.ts` to search for transcript entry points using DOM selectors:
   - `button[aria-label*="transcript" i]` or `button[aria-label*="Show transcript" i]`
   - If not found directly, looks in the "More actions" menu
4. Clicks the transcript button (or menu item) programmatically
5. Waits briefly for transcript panel to load (timing assumptions are encapsulated in `openTranscriptPanel`)
6. Extracts all `ytd-transcript-segment-renderer` elements from DOM
7. For each segment:
   - Extracts time element (class contains "time")
   - Extracts text element (class contains "segment-text")
   - Parses timestamp string (MM:SS or H:MM:SS) to seconds
   - Returns structured `{ text, start, duration }` objects

**Advantages of this approach**:
- More reliable than API methods (uses same data YouTube shows users)
- Doesn't depend on undocumented APIs or URL structures
- Works as long as YouTube's transcript UI exists
- No authentication or rate limiting issues

**If YouTube changes their DOM structure**, the selectors may need updating. Key elements to look for:
- Transcript button and menu item: Check aria-label attributes and menu items
- Transcript segments: Look for transcript-related element tags
- Time/text elements: Inspect class names in transcript panel
- All selectors live in `src/content/transcriptDom.ts`; update them there first.

Error codes for transcript failures are defined in `src/utils/errors.ts` (`TranscriptErrorCode`) and are surfaced to the popup via `isTranscriptError` / `parseError`. When selectors break, prefer throwing specific codes like:
- `UI_NOT_FOUND` when the transcript UI can’t be located
- `MENU_ITEM_NOT_FOUND` when the transcript option is missing from the menu
- `SEGMENTS_NOT_FOUND` when the panel opens but no segments are present

### Styling with Tailwind CSS v4

**Important**: This project uses Tailwind CSS **v4**, which has different syntax than v3:

- **CSS Import**: Use `@import "tailwindcss";` NOT `@tailwind base/components/utilities;`
- **No config file needed**: Tailwind v4 auto-detects source files (no `tailwind.config.js`)
- **PostCSS plugin**: Uses `@tailwindcss/postcss` in `postcss.config.js`
- **Typography plugin**: provides a set of `prose` classes to add beautiful typographic defaults

### Vite Build Configuration

**Critical settings** in `vite.config.ts`:

```typescript
export default defineConfig({
  plugins: [
    react(),
    crx({ manifest: manifest as any }) // @crxjs/vite-plugin handles Chrome extension bundling
  ],
  base: './', // REQUIRED: Chrome extensions need relative paths, not absolute /assets/
  build: {
    outDir: 'dist',
  }
});
```

The `base: './'` is essential - without it, CSS/JS paths will be `/assets/...` which fail in Chrome extensions (must be relative like `../../assets/...`).

### Manifest V3 Specifics

- Service worker is TypeScript (`src/background/index.ts`), Vite compiles to JS
- Content script runs on `https://www.youtube.com/*` only
- Permissions: `activeTab`, `scripting`, `storage`, `windows`
- Icons must be in `public/icons/` for @crxjs to copy them correctly

## Common Gotchas

1. **CSS not loading**: Verify `base: './'` in vite.config.ts and check generated HTML has relative paths (`../../assets/` not `/assets/`)

2. **Content script can't access popup state**: They're isolated contexts. Use `chrome.tabs.sendMessage()` to communicate.

3. **API keys not persisting**: Ensure `chrome.storage.local.set()` is called with both keys as an object, not separately.

4. **Transcript extraction fails**:
   - Check if YouTube changed their DOM structure for the transcript button or panel
   - Inspect the transcript button's aria-label attribute
   - Check if `ytd-transcript-segment-renderer` elements still exist in the transcript panel
   - Open DevTools on YouTube page and check console for `[Content]` logs to debug

5. **Extension doesn't reload**: After `npm run build`, you must:
   - Click the reload icon in `chrome://extensions/`
   - **Hard refresh the YouTube page** (Ctrl+Shift+R / Cmd+Shift+R) to load new content script
   - The old content script code may be cached until page refresh

6. **Settings modal black areas**: The overlay div should not have `bg-black bg-opacity-50` classes - these create dark areas on the sides of the fixed-width popup.

## File Organization

- **`src/popup/App.tsx`**: Popup UI orchestrator, settings + summary viewer, talks to background worker
- **`src/popup/components/`**: Presentational components (SettingsModal, SummaryView)
- **`src/utils/youtube.ts`**: YouTube transcript extraction helper (sends `GET_TRANSCRIPT_DATA` to content script)
- **`src/utils/openai.ts`**: OpenAI Chat Completions API integration
- **`src/content/index.ts`**: DOM manipulation (transcript extraction via `transcriptDom`, video seeking, playback control)
- **`src/content/transcriptDom.ts`**: Centralized selectors and timing logic for the transcript panel
- **`src/background/index.ts`**: Background orchestrator (summaries, badges, notifications, storage)
- **`public/icons/`**: Extension icons (copied to dist/ by @crxjs)

## External Dependencies

- **Runtime**: OpenAI API (GPT-4o-mini), YouTube's native transcript UI
- **User-provided**: OpenAI API key (required), YouTube API key (optional, not currently used)
- **Cost**: ~$0.01-0.05 per summary via OpenAI API

## Recent Major Changes

### Transcript Extraction Rewrite (2025)
The transcript extraction was completely rewritten to use DOM extraction instead of YouTube's timedtext API:
- **Old approach**: Scraped video page HTML, parsed JSON for captionTracks, fetched timedtext XML
- **Problem**: YouTube's timedtext API returned empty responses (200 OK with 0 bytes)
- **New approach**: Content script clicks transcript button and extracts from DOM
- **Result**: More reliable, no API dependencies, works as long as transcript UI exists

Timedtext / Innertube-based approaches are now kept only as historical context; the production path is **DOM transcript panel first** via `src/content/transcriptDom.ts` and `GET_TRANSCRIPT_DATA`.

### Background-Orchestrated Summarization (2025)
Summarization was moved from the popup into the background worker to make long-running summaries more robust and to support multiple videos in parallel:
- Popup sends `START_SUMMARY` and listens for progress / result messages.
- Background worker handles transcript extraction, OpenAI calls, storage, badges, and notifications.
- Per-tab badges (`✓`) stay visible until the user opens the popup on that tab and views the summary.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alekko-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
