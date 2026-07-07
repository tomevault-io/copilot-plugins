## corpus

> This file is the source of truth for Claude Code on every task. Read it first before any change.

# Corpus — Project Guidelines for Claude Code

This file is the source of truth for Claude Code on every task. Read it first before any change.

---

## What Corpus is

A Hebrew/English daily micro-learning app, Duolingo-styled, focused on philosophy (with future support for economics and psychology). Built as a single-page app and hosted on GitHub Pages.

## Stack — keep it simple

- **One file:** `index.html` contains everything — HTML, CSS in `<style>`, JS in `<script>`
- **Vanilla JS:** no framework, no build step, no bundler
- **Supabase:** auth and progress storage
- **Fonts:** DM Sans only (already loaded)
- **Hosting:** GitHub Pages

**Never** introduce frameworks, build steps, bundlers, or new dependencies. If a task seems to require one, ask first.

---

## Bilingual + Direction (the #1 source of bugs — read carefully)

Corpus runs in two languages with opposite text direction:
- English: `<html lang="en" dir="ltr">`, default for new users
- Hebrew: `<html lang="he" dir="rtl">`, persisted user preference

**Rules for any new CSS or JS that involves position, alignment, or spacing:**

✅ **Always use logical properties:**
- `inset-inline-start` / `inset-inline-end` (NOT `left` / `right` for direction-relative positioning)
- `padding-inline-start` / `padding-inline-end` (NOT `padding-left` / `padding-right`)
- `margin-inline-start` / `margin-inline-end`
- `border-inline-start` / `border-inline-end`
- `text-align: start` / `text-align: end` (NOT `left` / `right`)

❌ **Never hardcode `direction: rtl` or `direction: ltr` on individual elements.** Direction inherits from `<html dir>`. The only exception is the subject-select screen, which must stay LTR regardless of language.

❌ **Never assume "left = start" in JS.** When animating slides, scrolls, or positions, account for both directions.

If a task needs verification across both languages, the test must run once in `setAppLang('en')` and once in `setAppLang('he')`.

---

## Design system

### Colors — CSS variables only

All colors flow through CSS custom properties defined on `:root` and overridden by theme classes on `<html>`. **Never hardcode hex values in CSS or inline styles.** If a needed color doesn't exist as a variable, propose adding one rather than hardcoding.

**Subject-themed variables** (change per active subject):
```
--accent          main brand color (purple #7B4FD4 default)
--accent-dark     pressed/strong states
--accent-glow     rgba glow for shadows
--purple          alias of --accent (legacy)
--purple-light    
--lilac           
--ink             deepest text (#2A1266 default)
--text            primary body text
--text-2          secondary text  
--text-muted      faint text
--bg              app background (#F8F5FF default)
--surface-2       tinted card background
--border          subtle dividers
--border-deep     stronger dividers
--shadow          rgba shadow (default)
--shadow-lg       larger shadow
```

**Constants across all themes** (don't override per subject):
```
--surface         #FFFFFF (always white)
--amber           #F59E0B (highlights/streaks)
--gold, --gold-light  
--pink            #FF6FA8 (status)
--red, --red-light    (errors)
```

**Theme classes:** `html.theme-philosophy`, `html.theme-economics`, `html.theme-psychology`. Switching themes calls `applyTheme(subject)` which swaps the class on `<html>`.

### Animation conventions

- **Screen transitions:** `fadeInUp` — opacity 0→1 + translateY(12px→0), 0.5s, `cubic-bezier(0.4, 0, 0.2, 1)`
- **Card hover:** `translateY(-6px)`, 0.15s
- **Button press:** `scale(0.97)` + accent ripple
- **Celebratory moments:** `cubic-bezier(0.34, 1.56, 0.64, 1)` (spring)
- **Theme color transition:** 350ms ease on `background-color`, `border-color`, `color`, `box-shadow`
- **Drawer open/close:** 450ms `cubic-bezier(0.4, 0, 0.2, 1)`

❌ **Never use `transition: all`** — kills performance and animates unintended layout properties. Always specify which properties transition.

### Existing components (don't reinvent these)

- `.topbar` — top bar (fixed, z-index 100)
- `.stats-subheader` — stats below topbar (fixed, z-index 99)
- `.bottom-nav` — bottom navigation (fixed, z-index 200)
- `.cc-a` — primary CTA button style
- `.week-row` — DEPRECATED, replaced by `.level-card` + `.lesson-node`
- `.level-card` (`.expanded` / collapsed) — week container on home
- `.lesson-node` + `.lesson-circle` (`.completed` / `.next-up` / `.locked-circle`) — lesson nodes
- `.lesson-path` with `::before` stripe — vertical connector between lessons
- `.subj-screen` — subject select screen
- `.subj-icon-wrap` — subject icon container
- `.drawer-toggle-btn` — chevron in topbar opening subject-select drawer

---

## Screens (state.currentScreen + showScreen())

```
screen-auth        login/signup
screen-home        main dashboard with levels + lessons
screen-pantheon    thinkers gallery
screen-profile     user profile + settings
screen-subject     subject selection (drawer-style overlay)
screen-reader      lesson reader (sections, quiz, reflection)
```

Switch via `showScreen(screenName)`. The function hides all other screens by toggling display. **When adding a new screen, register it in `showScreen`'s screen list** so it gets hidden when others are activated — otherwise screens stack and bugs occur.

---

## State and core data

### Core data accessors
- `getW()` — array of weeks for the current subject
  - `week.id`, `week.title`, `week.titleEn`
  - `week.days[]` — these are LESSONS inside the week
    - `day.id`, `day.title`, `day.titleEn`, `day.thinker`
    - `day.sections[]` — lesson content blocks (text/quiz/reflection)
- `state.completedDays[`${weekId}-${dayId}`]` = `true` if completed

### State helpers (use these, don't manipulate `state` directly for these fields)
- `setSubject(value)` — saves subject, updates state, re-themes app
- `setAppLang(lang)` — switches language and direction, re-renders
- `applyTheme(subject)` — swaps theme class on `<html>`
- `loadState()` / `saveStateLocal()` — persistence (also writes Supabase)

### Reader navigation
- `openReader(weekId, dayId)` — opens a lesson; sets `state.currentWeek` and `state.currentDay`, calls `renderReader()` and `showScreen('reader')`
- Don't bypass `openReader` — use it as the entry point for any "open a lesson" flow

### Locked subjects
- `LOCKED_SUBJECTS = ['economics', 'psychology']` — currently only philosophy is unlocked
- To unlock a subject later, remove it from this array — single-line change

---

## Hard constraints (apply to every task)

### Never
- ❌ Push to git (`git add`, `git commit`, `git push`) — the user reviews and commits manually
- ❌ Touch PNG icons in `/assets/icons/subjects/`
- ❌ Touch lesson content data (titles, sections, quiz questions, thinker names)
- ❌ Introduce new frameworks, build steps, or runtime dependencies
- ❌ Use `transition: all`
- ❌ Hardcode hex colors in CSS — use variables
- ❌ Hardcode `direction: rtl` or `left`/`right` where logical properties work
- ❌ Rename or remove existing functions without checking all callsites
- ❌ Replace existing translations in the language system — Corpus has a complete EN/HE translation already; reuse it via `state.lang` checks (`isEn = state.lang === 'en'`)

### Always
- ✅ Use CSS variables for colors
- ✅ Use logical CSS properties for direction-aware layouts
- ✅ Test changes in both English and Hebrew when relevant
- ✅ Keep new CSS scoped (prefix new classes appropriately, e.g., `.subj-`, `.level-`, `.lesson-`)
- ✅ Group new JS under a comment header indicating its feature, e.g., `// === Subject Select ===`
- ✅ Match existing animation timings — don't invent new ones

---

## Verification protocol

Every change must end with verification. The format:

1. **Run a console snippet** that proves the change took effect (e.g., elements exist, classes are correct, computed styles match expectations)
2. **Paste the snippet's output** in the response
3. **Note any visual checks performed** ("verified visually that X looks correct in both LTR and RTL")
4. **List any regressions checked** — confirm related flows still work (e.g., "subject switching still works after this change")

If a change involves both LTR and RTL, run verification in both modes:
```js
window.setAppLang('en');
// snippet
window.setAppLang('he');
// same snippet
```

---

## Asking before assuming

If a task is ambiguous, **ask before implementing** rather than guessing. Areas that especially warrant clarification:
- New screens — where do they appear in the existing flow?
- New state fields — should they persist across sessions?
- Theme changes — apply globally or scoped to one screen?
- Locked content behavior — is "Coming soon" the right UX, or something else?

It's cheaper to ask than to redo.

---

*Last updated: 2026-05-10*

---
> Source: [omeroth/corpus](https://github.com/omeroth/corpus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
