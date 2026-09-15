## ambar-ai-video-editor-plugin-for-premiere-pro

> > Read this file first every session. Then read the specific feature guide.

# CLAUDE.md — Ambar Plugin · Phase 2
> Read this file first every session. Then read the specific feature guide.
> Version: 0.2.0 — Story Intelligence + Multi-format + Amharic + AI Chat

---

## What Ambar Is Now

Professional Premiere Pro UXP + CEP hybrid plugin for AI-assisted editing.
Supports vlogs, podcasts, tutorials, and long-form repurposing.
Brand: **Ambar** · Plugin ID: `com.robelaipremiereassistant.plugin.v2`

---

## READ THESE IN ORDER BEFORE CODING

1. `PPRO_API_CHEATSHEET.md`
2. `CEP_BRIDGE_CHEATSHEET.md`
3. `TRANSCRIPT_GUIDE.md` — three-layer pipeline (do not rewrite)
4. `LOCAL_AI_SETUP.md`
5. `NEW_FEATURES_GUIDE.md` ← NEW — all Phase 2 specs
6. `AMHARIC_GUIDE.md` ← NEW — Amharic language workflow
7. `SHIPPING_CHECKLIST.md`

---

## ⚠️ DEAD FILES — DELETE BEFORE ANYTHING ELSE

| File | Why |
|---|---|
| `js/ai/gemini-service.js` | COLLISION: defines `const AIService` — crashes plugin |
| `STRUCTURE.md` | Pre-refactor, wrong info |
| `FIX_AI_SERVICE_INITIALIZATION.md` | Debug note, done |
| `TEST_RUN_DIAGNOSTICS.md` | Debug note, done |
| `THREE_LAYER_PIPELINE_FIXED.md` | Debug note, done |
| `REFACTOR_PLAN.md` | Superseded |

Remove `<script src="js/ai/gemini-service.js"></script>` from index.html.

---

## What Works (do not break)

- Three-layer silence pipeline (AudioAnalyzer → WhisperService → AIService)
- Ripple delete via CEP bridge (22 functions in host.jsx)
- B-roll placement on V2 with 40% budget cap
- SRT captions + caption track
- Vision bin organiser
- Project memory per sequence
- Sidebar UI with Space Grotesk + JetBrains Mono

---

## New Files (Phase 2)

```
js/core/story-analyzer.js    ← repeat/mistake/take detection + rearrange
js/core/hook-creator.js      ← YouTube/TikTok/Reels hook generation
js/core/repurpose-engine.js  ← long-form → short-form clips
js/core/podcast-mode.js      ← speaker diarization, Q&A workflow
js/ui/ai-chat.js             ← conversational AI interface
```

---

## Updated index.html Script Order

```html
<script src="js/utils/constants.js"></script>
<script src="js/utils/logger.js"></script>
<script src="js/utils/error-handler.js"></script>
<script src="js/utils/validators.js"></script>
<script src="js/utils/capabilities.js"></script>
<script src="js/core/project-memory.js"></script>
<script src="js/core/cep-bridge.js"></script>
<script src="js/core/audio-analyzer.js"></script>
<script src="js/core/frame-extractor.js"></script>
<script src="js/core/premiere-api.js"></script>
<script src="js/core/project-reader.js"></script>
<script src="js/ai/prompt-templates.js"></script>
<script src="js/ai/response-parser.js"></script>
<script src="js/ai/whisper-service.js"></script>
<script src="js/ai/vision-service.js"></script>
<script src="js/ai/ai-service.js"></script>
<script src="js/core/transcript-store.js"></script>
<script src="js/core/edit-executor.js"></script>
<script src="js/core/timeline-editor.js"></script>
<script src="js/core/broll-placer.js"></script>
<script src="js/core/caption-engine.js"></script>
<script src="js/core/project-organizer.js"></script>
<script src="js/core/story-analyzer.js"></script>
<script src="js/core/hook-creator.js"></script>
<script src="js/core/repurpose-engine.js"></script>
<script src="js/core/podcast-mode.js"></script>
<script src="js/ui/ui-state.js"></script>
<script src="js/ui/ai-chat.js"></script>
<script src="js/ui/ui-controller.js"></script>
<script src="js/index.js"></script>
```

gemini-service.js is NOT in this list. Do not add it.

---

## Constants to Add

```js
CONTENT_TYPE: 'vlog',       // 'vlog'|'podcast'|'tutorial'|'interview'
LANGUAGE: 'auto',           // 'auto'|'en'|'am'|'fr' etc.
AMHARIC_PROVIDER: 'gemini',
GEMINI_API_KEY: '',
OUTPUT_FORMATS: {
  SHORTS:  { width:1080, height:1920, maxDurationSec:60  },
  REELS:   { width:1080, height:1920, maxDurationSec:90  },
  SQUARE:  { width:1080, height:1080, maxDurationSec:60  },
  YOUTUBE: { width:1920, height:1080, maxDurationSec:600 },
},
HOOK_DURATION_SEC: 30,
AI_CHAT_HISTORY_MAX: 10,
```

---

## Rules (always)

- `await` all Premiere proxy properties
- BigInt ticks for all timeline positions
- `executeTransaction()` with `return true` for all writes
- ES3 only in host.jsx
- REVERSE ORDER for delete segments
- Never upload full video to cloud
- Graceful degradation without CEP bridge

---

## ⚠️ CRITICAL — Edit Executor Rule

**Never call `CEPBridge.sendCommand('razorAndDelete', ...)` directly from any feature.**
Always use `EditExecutor.executeCut(startWordIdx, endWordIdx, label, paddingSeconds)`.

This is what keeps `TranscriptStore` accurate across all operations in any order.
When a cut bypasses `EditExecutor`, the store goes stale and every subsequent cut
points to the wrong content.

```
WRONG:  await CEPBridge.sendCommand('razorAndDelete', { segments: [...] });
RIGHT:  await EditExecutor.executeCut(startWordIdx, endWordIdx, 'Story: false start', 0.04);
```

**Insertion tracking:** After any operation that inserts content into the sequence
(e.g. `createHookAtStart`), call `EditExecutor.recordInsertion(atMs, durationMs)`
so word timestamps shift correctly for all subsequent operations.

**Exception:** `timeline-editor.js commitEdits()` manages the silence pipeline with
its own SequenceEditor + CEP bridge flow and updates TranscriptStore via its own
reverse-order loop. Silence cuts are structural — everything above the silence layer
uses EditExecutor.

---
> Source: [Robelob/Ambar-AI-Video-Editor-Plugin-For-Premiere-Pro](https://github.com/Robelob/Ambar-AI-Video-Editor-Plugin-For-Premiere-Pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
