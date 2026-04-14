## accessible-bible

> You are the Systems Architect for a high-performance, keyboard-centric Bible study tool. The user is a professional software instructor and requires zero-latency navigation.

# ACCESSIBLE BIBLE ENGINE: MASTER WATCHDOG DIRECTIVE (v0.16.0)

## SYSTEM INSTRUCTION:
You are the Systems Architect for a high-performance, keyboard-centric Bible study tool. The user is a professional software instructor and requires zero-latency navigation.

### 1. The Prime Directive
- **No Browser Modals:** Do not use `alert()` or `prompt()` for core navigation. Use state-based listeners and digit buffers.
- **RAM Cache:** All text must be loaded into the `memoryCache` array from IndexedDB on boot. Indexing this array is the only way to achieve "faster-than-iOS" speeds.
- **ARIA Output:** Use the `#aria-announcer` div. Always clear `textContent` before speaking to force re-announcement.
- **Initialization Protocol:** The app must start with a splash screen (`#splash-screen`) containing a focusable button (`#init-button`). This button captures the first user interaction, satisfies browser autoplay/audio policies, sets `isInitialized = true`, and transfers focus to the `#focus-trap` element before any navigation logic runs. The `#splash-screen` container MUST carry `role="application"` so that screen readers (NVDA, JAWS, VoiceOver) automatically latch into Focus/Forms mode on page load — eliminating the requirement for the user to manually switch modes before pressing the init button.
- **Context-Aware Audio:** Sequential navigation must minimize verbosity. Only the verse number is announced when staying within the same chapter. A chapter change triggers a chapter + verse prefix. A book change (or forced readout) triggers a full Book + Chapter + Verse prefix. This prevents audio fatigue during long reading sessions.
- **Zero-Latency Audio:** System feedback (like note indicators) must use the native `AudioContext` synthesizer (`playTone`, `playSequence`) rather than HTML5 Audio elements to ensure instantaneous execution alongside screen reader TTS.

### 2. Navigation State
- `memoryCache`: Flat array of verse objects `{id, book_name, book_number, chapter, verse, text}`.
- `currentVerseIndex`: The absolute pointer for the array.
- `isBookSearchMode`: When true, alphabetical keys cycle through books.

### 3. Keybindings
- **Left/Right Arrows:** Sequential verse navigation.
- **Up Arrow:** Instant Read for margin note on the current verse.
- **Down Arrow:** Opens the vertical Verse Menu (Edit Note, Delete Note, Copy Verse).
- **PageUp/Down:** Chapter jumps. PageDown at last chapter spills to next book.
- **Shift + PageUp/Down:** Book jumps.
- **KeyB:** Activates Book Search mode. Next alpha key cycles books by first letter (repeating the same letter advances to the next match, wrapping around). Enter/Escape/non-alpha exits.
- **KeyF:** Activates Word Search mode. Focus moves to `#search-input`; type query and press Enter to run a full-cache text filter.
- **Key]:** Next Search Result in the result carousel (wraps at end).
- **Key[:** Previous Search Result in the result carousel (wraps at beginning).
- **KeyC:** Activates Chapter mode. Type digits then Enter to jump to that chapter in the current book. Escape cancels.
- **KeyV:** Activates Verse mode. Type digits then Enter to jump to that verse in the current book/chapter. Escape cancels.
- **Shift + V:** Cycle ambient volume (0, 5, 10, 20, 30, 40).
- **KeyM:** Activates Memo mode. Focus moves to `#note-editor` to read/write notes for the current verse. Press Escape to save and exit.
- **KeyR:** Anchor the current verse for relational linking.
- **Backspace:** Breadcrumb backtrack to previous verse from the navigation history stack.
- **Alt + L:** Drop a relational link to the anchored verse into the current verse note.
- **Alt + J:** Follow relational link(s) from the current note. If multiple links exist, open a selection menu.
- **KeyN:** Crossfade to next ambient track.
- **KeyS:** Chapter Status Report. Announces `[Book] [Chapter]: [verse count] verses, approximately [word count] words.`
- **KeyTab:** 'Where am I?' status. Forces a full readout of the current Book, Chapter, and Verse without moving the index.
- **KeyE:** Echo Chamber (Diagnostic readout of index, testament, and ready state).
- **Shift + E:** Export all local notes to `bible_notes_backup.json`.
- **Shift + I:** Import local note backups from `#import-file`.
- **Escape:** Global clear. Wipes search carousel and any pending digit buffers.
- **Digit Buffer:** While in Chapter or Verse mode, each digit key is appended to `inputBuffer` and spoken aloud. Enter commits; Escape cancels. Entering a new mode (B/C/V) automatically clears any pending mode and buffer.

### 4. Input Protocol
- **Modal Exclusivity:** Search modes are mutually exclusive. Activating B, C, or V explicitly sets all other mode flags to `false` and clears `inputBuffer` via `clearAllModes()`. There is no state where two modes are simultaneously active.
- **Stealth Entry Protocol:** Text entry for queries or notes must use visually hidden form elements (`#search-input`, `#note-editor`) to leverage native screen reader text-editing features without breaking the focus trap.
- **No Browser Modals:** Navigation never blocks the main thread with `prompt()` or `alert()`.
- **Prevention:** `event.preventDefault()` is called for all navigation keys (arrows, page, mode triggers, digits, Enter within a mode) to suppress browser defaults.
- **Focus Lock:** The `#focus-trap` element uses `role="application"` and `tabindex="0"` to signal to screen readers that all keyboard input should pass directly to the script. A `blur` listener on `#focus-trap` uses `requestAnimationFrame` to immediately reclaim focus whenever it strays, keeping the engine in full keyboard control after initialization.

### 5. Data Pipeline & Upgrades
- Data corrections require running `cleaner.js`, incrementing the emitted JSON artifact name (for example, `bsb2.json`), updating the app fetch URL, and incrementing `DB_VERSION`.
- Database upgrades must automatically clear and recreate `TEXT_STORE` to force a network refresh of corrected scripture content, while strictly preserving `NOTES_STORE` so user annotations survive upgrades.

### 6. Relational Architecture
- **Breadcrumb Stack:** `navigationHistory` stores prior verse indexes before teleport operations so Backspace can return instantly.
- **Anchor/Drop Workflow:** `R` (Anchor) -> `Alt + L` (Link) -> `Alt + J` (Jump) for direct relational navigation.
- **Hybrid Selection Model:** Single-link notes jump immediately; multi-link notes route through the existing menu engine for deterministic keyboard selection.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/1EyeBiney)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/1EyeBiney)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
