## writing-grader

> Build a writing readability and clarity grader. Pure HTML/CSS/JS, single index.html.

# writing-grader — Hemingway App alternative

Build a writing readability and clarity grader. Pure HTML/CSS/JS, single index.html.

## What Hemingway App does
- Paste text → highlights issues inline
- Grades by readability (Flesch-Kincaid grade level)
- Highlights: adverbs (blue), passive voice (green), hard sentences (yellow = hard, red = very hard), simpler alternatives (purple)
- Right sidebar: stats panel (grade, reading time, word count, sentences, adverbs, passive voice count, hard sentences)
- Shows color legend

## Design
- Clean two-panel layout: editor on left, stats on right
- Editor is a contenteditable div with live highlighting
- Stats panel: grade display (big number), breakdown of issues
- Color coding: adverbs=blue, passive=green, hard sentence=yellow, very hard=red, simpler word=purple
- Dark mode with red accent

## Algorithms
- Flesch-Kincaid readability (standard formula)
- Sentence length detection (>14 words = hard, >25 = very hard)
- Adverb detection (words ending in -ly)
- Passive voice detection (was/were/is/are/been + past participle pattern)
- Common complex word suggestions (utilize→use, commence→start, etc.)

## Stack
Pure HTML/CSS/JS, no dependencies

## UPGRADE: Add Supabase Auth + Data Persistence

## Supabase Config (embed in each tool)
- URL: https://grxvqkaokerjrsklqubr.supabase.co
- Anon Key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImdyeHZxa2Fva2VyanJza2xxdWJyIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzI0Mzg4OTAsImV4cCI6MjA4ODAxNDg5MH0.UYg_98cYSfxV9egaoDyuM1LIntTn_qzEvBm5xUUVw6A
- SDK: Load from CDN: https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js

## Auth Features Required
- Sign up with email + password (auto-confirmed, no email verification)
- Sign in with email + password
- Sign out
- Show user email in top bar when logged in
- Show Sign In / Sign Up buttons when logged out
- Auth modal/form: inline in a modal, not a separate page

## Data Features Per Tool

### code-screenshot (table: snippets)
- Save snippet: title, code, language, theme, background, font_size
- My Snippets page: list user's saved snippets, click to load
- Share: public snippets get a shareable URL (/?id=uuid)
- Browse: gallery of recent public snippets

### color-palettes (table: palettes) 
- Save palette: name, colors (JSON array of hex strings)
- My Palettes: list user's saved palettes, click to load
- Community: browse recent public palettes, click to fork
- Share: /?id=uuid loads a specific palette

### writing-grader (table: documents)
- Save document: title, content, grade_level, word_count
- My Documents: list saved docs, click to load
- Auto-save after 3 seconds of no typing

### type-scale (table: scales)
- Save scale: name, base_size, ratio, font_family
- My Scales: list saved scales
- Share: /?id=uuid loads a specific scale
- Browse: recent public scales

## Implementation Notes
- Add supabase-js CDN script tag
- Initialize: const supabase = window.supabase.createClient(URL, ANON_KEY)
- Auth UI: modal with email/password sign up + sign in tabs
- Top bar: show user email when logged in, "Sign In" button when not
- All save operations require auth (check supabase.auth.getUser())
- Load from URL: check ?id= param on page load → fetch from Supabase
- Keep ALL existing functionality — just ADD auth + save/load on top
- Dark theme, red accent (#ff2d2d) for all UI elements

---
> Source: [jax-agent/writing-grader](https://github.com/jax-agent/writing-grader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
