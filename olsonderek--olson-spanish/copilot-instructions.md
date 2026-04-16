## olson-spanish

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

# PRD: Mobile-First Spanish Class App (Front-End Only)

## 1) Summary & Goals

A super-simple, mobile-first web app for a once-a-week high school elective. The app shows the current week’s vocabulary and example phrases, lets students tap to reveal meanings and conjugations, and allows quick “reviewed this session” check-offs. No login, no backend — just a static deploy with one editable config file you can update weekly.

**Core goals**

* Be fast, dead simple, and delightful on phones.
* One tap reveals meaning; another tap shows conjugations; another returns to Spanish.
* Press-to-play audio for phrases.
* Per-session review checkmarks with an easy reset.
* “Single source of truth” via a weekly config file (JSON).

**Non-goals**

* Accounts, grades, or analytics.
* Teacher dashboard or per-student tracking beyond a local device session.

---

## 2) Users & Constraints

**Primary users:** 13–18 year-old students using personal phones.
**Environment:** School Wi-Fi/cellular, mixed iOS/Android, short sessions (5–15 min).
**Constraint:** App must run entirely client-side (static hosting). No database.

---

## 3) Scope of Features

### 3.1 Must-Haves

1. **Weekly Vocab Browser**

   * Display only the current week by default (from config).
   * Each vocab item is a tappable card that cycles states:

     * **State A:** Spanish (default)
     * **State B:** English translation
     * **State C:** Conjugations/Forms (if present)
     * **State A again**
   * Cards show a subtle “reviewed” check when the student marks them during the session.

2. **Phrases/Sentences with Audio**

   * List of example phrases that use this week’s vocab (highlight matched words).
   * **Play button**: speak phrase in Spanish.
   * Tap to mark a phrase as reviewed (per session).

3. **Session Review Tracking**

   * A floating **“Session progress”** bar with counts (e.g., “Words: 6/20 · Phrases: 3/8”).
   * **Reset Session** button clears current checkmarks.
   * Session state persists in **localStorage** and auto-expires after 24 hours or manual reset.

4. **One Config File Update**

   * All content controlled by a single `config.json` checked into the repo and deployed with the site.

### 3.2 Nice-to-Haves (V1.1+)

* **Week Picker** to revisit prior weeks (hidden by default, toggleable).
* **PWA** install + offline cache (static assets + current week config).
* **Dark mode** toggle.
* **Speed control** for audio playback (0.75x, 1x, 1.25x).
* **Shuffle mode** for vocab drilling.

---

## 4) Accessibility & UX Principles

* **Mobile-first** layout; target comfortable tap areas (44×44 px).
* **WCAG 2.1 AA**: color contrast, focus states, screen reader labels (ARIA), logical heading order.
* **Tap cycle** must be obvious (micro-animation + helper text “tap to cycle” on first use).
* **Audio**: Play only on user gesture; show **Play/Stop** and a minimal progress bar; provide **phonetic hint** or slow speak option if feasible.

---

## 5) Information Architecture

* **Home / Week View**

  * Header: Week title + info (e.g., “Week 6: La Comida”)
  * Tabs or segmented buttons: **Vocabulary** | **Phrases**
  * Content grid/list:

    * Vocabulary cards
    * Phrase list with play buttons
  * Footer / floating bar: Session progress + Reset

* **(Optional) Week Selector** (modal or side sheet)

  * List of weeks (title + date range)

---

## 6) Functional Requirements

### 6.1 Vocab Cards

* **Tap behavior** cycles through 3 states per item:

  * A → B → C → A
* **Conjugations/forms** display:

  * Verbs: person/tense sets (present, preterite, imperfect; min set in V1)
  * Nouns/adjectives: gender/number forms if provided
* **Reviewed toggle**: a check icon in the corner; tap check area to mark/unmark.

### 6.2 Phrase Items

* Show Spanish phrase, auto-highlight words that match this week’s vocab (case-insensitive).
* **Audio**:

  * **Primary**: Web Speech API (SpeechSynthesis) using `es-ES` voice if present on device.
  * **Fallback**: Pre-generated MP3s (optional field in config) served statically.
* **Reviewed toggle**: same pattern as vocab.

### 6.3 Session Tracking

* **Storage**: `localStorage` keys scoped by `week_id`:

  * `session.vocabReviewed.{week_id}` → array of vocab IDs
  * `session.phraseReviewed.{week_id}` → array of phrase IDs
  * `session.startedAt.{week_id}` → timestamp
* **Auto-expire** session after 24 hours (on load: if `now - startedAt > 24h`, clear).

### 6.4 Config Resolution

* Load `/config.json` at app start.
* Use `current_week_id` to populate default view.
* If fetch fails, show a friendly offline message and prompt to refresh.

---

## 7) Non-Functional Requirements

**Performance**

* Time to interactive < 2s on mid-range phones (over decent connection).
* Bundle size target < 150KB gzipped (no heavy frameworks; consider Preact or vanilla).

**Compatibility**

* iOS Safari 15+, Chrome/Edge/Firefox (latest 2 versions).
* No sign-in; no cookies; only localStorage.

**Security & Privacy**

* No PII. No tracking/analytics by default.
* HTTPS required (static host).

**Reliability**

* App still usable if TTS voice missing (text visible; phrases playable via fallback MP3 if provided).

---

## 8) Content Model (Single Config File)

**File:** `/config.json`

```json
{
  "app": {
    "title": "Spanish Weekly",
    "current_week_id": "2025-09-01",
    "audio": {
      "tts": { "enabled": true, "voiceHint": "es-ES" },
      "preferMp3": false
    }
  },
  "weeks": [
    {
      "id": "2025-09-01",
      "title": "Week 1: Saludos",
      "dateRange": "Sep 1–7, 2025",
      "vocab": [
        {
          "id": "hola",
          "spanish": "hola",
          "english": "hello",
          "type": "interjection"
        },
        {
          "id": "ser",
          "spanish": "ser",
          "english": "to be (essential)",
          "type": "verb",
          "conjugations": {
            "present": {
              "yo": "soy", "tú": "eres", "él/ella": "es",
              "nosotros": "somos", "vosotros": "sois", "ellos": "son"
            },
            "preterite": {
              "yo": "fui", "tú": "fuiste", "él/ella": "fue",
              "nosotros": "fuimos", "vosotros": "fuisteis", "ellos": "fueron"
            }
          }
        }
      ],
      "phrases": [
        {
          "id": "p1",
          "spanish": "Hola, ¿cómo estás?",
          "english": "Hi, how are you?",
          "audioUrl": ""  // optional mp3; if empty, use TTS
        },
        {
          "id": "p2",
          "spanish": "Yo soy estudiante.",
          "english": "I am a student.",
          "audioUrl": ""
        }
      ]
    }
  ]
}
```

**Notes**

* `current_week_id` controls the landing week.
* Each vocab entry must have a unique `id`.
* `conjugations` object is optional; if absent, the cycle only goes A → B → A.
* `audioUrl` optional per phrase; when present, the app uses it and skips TTS.

---

## 9) UI/UX Specs

**Design language**

* Clean, modern, high contrast; rounded cards; subtle shadows; large touch targets.
* Cohesive color system (e.g., Primary = deep blue; Success = green checks; Accent = amber).

**Layouts**

* **Header**: App title, week title; optional week picker icon.
* **Segmented control**: Vocabulary | Phrases (sticky under header).
* **Vocabulary**: 2-column grid on mobile; 3–4 columns on tablet/desktop.
* **Phrases**: Single column list; each row with play button, text, and checkmark.
* **Floating bar**: Progress + Reset Session.

**Micro-interactions**

* Card flip/slide (150–200ms) on tap to reinforce state change.
* Confetti tick (very subtle) on first time a section reaches 100% reviewed.

**Empty/error states**

* If config missing: “Can’t load this week. Pull to refresh or try again later.”

---

## 10) Technical Approach

**Stack**

* Preact + Vite for DX and small bundle.

**Audio**

* Use **Web Speech API** (`speechSynthesis`) with `es-ES` voice hint.
* If not available or blocked, and `audioUrl` exists, fallback to `<audio>` element.

**State**

* In-memory + localStorage (scoped per `week_id`).

**Deploy targets**

* Vercel, Netlify, or GitHub Pages (static).

**PWA (optional)**

* Cache `/`, `/config.json`, and static assets; allow offline for current week.

---

## 11) Success Metrics

* **Adoption:** ≥90% of students load the app at least once per class.
* **Engagement:** Median session ≥5 minutes; ≥70% items reviewed per session.
* **Usability:** <3 taps to accomplish core tasks; reported ease-of-use ≥4/5 in quick poll.

---

## 12) QA & Acceptance Criteria

**Acceptance tests**

* [ ] Load on iOS Safari and Android Chrome without install prompts.
* [ ] Vocab tap cycles A → B → C → A (C only when conjugations exist).
* [ ] Phrases play on tap with visible play/stop state.
* [ ] Checkmarks persist across page reloads during the same day.
* [ ] Reset Session clears all checkmarks for the current week.
* [ ] Changing `current_week_id` in `config.json` changes the default view after redeploy.
* [ ] Screen reader announces card state (“Spanish”, “English”, “Conjugations shown”).
* [ ] All interactive targets pass 44×44 px size check.

---

## 13) Risks & Mitigations

* **TTS voice inconsistency on some devices** → Provide optional MP3s in config.
* **LocalStorage cleared by user** → Session is expendable by design; no grades rely on it.
* **Network hiccups** → Simple offline page + PWA caching (optional).

---

## 14) Project Plan (Lightweight)

**V0 (1–2 days)**

* Static scaffold, config loader, vocab/phrases render, tap cycle, basic styles.

**V1 (2–4 more days)**

* Audio (TTS + MP3 fallback), session tracking, progress bar, reset, accessibility passes.

**V1.1 (optional)**

* PWA, week picker, dark mode, shuffle, speed control.

---

## 15) Teacher Ops: Updating Content

* Edit `/config.json`:

  * Change `current_week_id` to this week.
  * Add/update a `weeks[]` entry with new `vocab[]` and `phrases[]`.
  * Optionally add `audioUrl` MP3s to `/audio/week-XXXX/`.
* Commit and deploy.

**We can set up a tiny schema check** (pre-commit or CI) to validate the JSON shape so deploys don’t break class.

---

## 16) Sample Week Content (Mini)

```json
{
  "id": "2025-09-08",
  "title": "Week 2: En la Escuela",
  "dateRange": "Sep 8–14, 2025",
  "vocab": [
    { "id": "lapiz", "spanish": "lápiz", "english": "pencil", "type": "noun" },
    { "id": "estudiar", "spanish": "estudiar", "english": "to study", "type": "verb",
      "conjugations": {
        "present": { "yo": "estudio", "tú": "estudias", "él/ella": "estudia", "nosotros": "estudiamos", "vosotros": "estudiáis", "ellos": "estudian" }
      }
    }
  ],
  "phrases": [
    { "id": "p1", "spanish": "Necesito un lápiz.", "english": "I need a pencil.", "audioUrl": "" },
    { "id": "p2", "spanish": "Yo estudio español cada día.", "english": "I study Spanish every day.", "audioUrl": "" }
  ]
}
```

## Development Commands

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Run linter
npm run lint
```

## Architecture

**Stack**: Preact + Vite for minimal bundle size and fast development
**Styling**: CSS custom properties + CSS modules pattern
**State**: Simple hooks + localStorage for session persistence
**Audio**: Web Speech API with MP3 fallback support

**Key Components**:
- `App.jsx`: Main app shell with navigation state
- `WeekView.jsx`: Tab-based vocab/phrases view
- `VocabCard.jsx`: Interactive flip cards (Spanish → English → Conjugations)
- `PhraseItem.jsx`: Audio-enabled phrase practice with vocabulary highlighting
- `SideNav.jsx` & `CalendarNav.jsx`: Week browsing interfaces
- `ProgressBar.jsx`: Session tracking with reset functionality

**Hooks**:
- `useConfig`: Loads and validates config.json
- `useSession`: Manages localStorage session state with auto-expiry
- `useAudio`: Handles TTS and MP3 audio playback

## Key Directories

```
src/
├── components/     # All React components
├── hooks/         # Custom hooks for state/logic
├── utils/         # Utility functions (if needed)
├── app.jsx        # Main app component
├── main.jsx       # App entry point
└── index.css      # Global styles with design system

public/
└── config.json    # Weekly content configuration
```

## Development Notes

**Content Management**: 
- All vocabulary and phrases are controlled via `/public/config.json`
- Update `current_week_id` to change the default week shown
- Each week needs unique `id`, `title`, `dateRange`, `vocab[]`, and `phrases[]`

**Adding New Weeks**:
1. Add new week object to `weeks[]` array in config.json
2. Optionally add MP3 files to `/public/audio/week-{id}/`
3. Set `current_week_id` to the new week's ID
4. Deploy (static hosting handles the rest)

**Mobile-First Design**:
- All touch targets are minimum 44×44px
- Navigation uses slide-in panels for space efficiency  
- Cards use tap cycling for vocab state changes
- Audio buttons are prominent and accessible

**Session Persistence**:
- Review checkmarks persist in localStorage scoped by week ID
- Sessions auto-expire after 24 hours
- No server required - purely client-side state

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OlsonDerek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
