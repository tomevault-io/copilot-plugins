## viva-flashcard-skill

> >


# Viva Flashcard Skill

You are orchestrating the generation of a thesis-specific flashcard deck for a UK PhD viva, then optionally drilling the candidate against it. Follow the phases below in order. Do not skip phases or drop card categories without surfacing it to the user first — coverage of the standard content axes is load-bearing (see `references/flashcard-methodology.md` for the source citations).

> **Mode tags.** `[both]` text + future voice; `[text-only]` v1 default; `[voice-only]` deferred to v2.

---

## Phase Map

```
0. Information gathering   thesis OR --reuse-profile path; role; deck size; flags
1. Thesis Profile          extract OR reuse via --reuse-profile
2. Card generation         single subagent → deck.json
3. Pre-show & approve      user reviews, drops, re-rolls, regenerates
4. Export                  deck.json + .apkg + .csv + .md + .pdf
5. Drill mode (optional)   --drill: in-session active recall + SM-2 log
```

If unclear which role the user wants, list available roles from `roles/` (excluding any file starting with `_`) and ask. Default is `phd-viva`.

---

## Phase 0 — Information Gathering

### 0.1 Locate inputs

Identify from the user's prompt:

- **Thesis source** OR **`--reuse-profile <path>`** — one of:
  - PDF path, `.tex` path (or directory of `.tex` files), Markdown summary, or pasted abstract + chapter summaries (in that preference order — richer source = better profile).
  - `--reuse-profile <path>` pointing at an existing `thesis_profile.json`. When present, skip Phase 1.
- **Role** — one of `roles/`. Default: `phd-viva`. If absent, list role names (excluding files starting with `_`) and ask.
- **Deck size** — `--deck-size {small|medium|large}`. Default: `medium`. Targets 50 / 80 / 120 cards respectively.
- **Flags**:
  - `--cite-pages` — **ON by default**. Back-of-card includes page/section refs (`Ch 4 §4.3, p.73`). Disable with `--no-cite-pages`.
  - `--examiner-deck` — **OFF by default**. v1 accepts the flag and produces the standard deck plus a one-line warning that examiner-specific weighting is deferred to v2.
  - `--drill` — runs Phase 5.
  - `--voice` — only valid with `--drill`. Phase 5 runs through a file-bridge to a companion `voice_runner.py` process the user starts in another terminal. See §5.4 below.
  - `--audio` — **OFF by default**. Phase 4 renders an MP3 per card side via `edge-tts` and embeds the clips in `deck.apkg`. See §4.5. Optional companions: `--audio-voice <name>` (default `en-GB-SoniaNeural`) and `--audio-rate <pct>` (default `+0%`).
  - `--append <existing-deck.json>` — append new cards to an existing deck rather than overwrite. Preserves Anki review history downstream.

### 0.2 Validate inputs `[both]`

| Code | Trigger | Action |
|---|---|---|
| F-1 | Thesis missing AND `--reuse-profile` not supplied | Ask user to paste an abstract + chapter list, OR point at a prior `thesis_profile.json`. Do not silently proceed. |
| F-2 | `--reuse-profile <path>` does not resolve to a readable JSON | Ask user to correct the path or supply thesis source for fresh extraction. |
| F-3 | Role file missing | List available role files; ask user to pick. |
| F-4 | `.docx` / scanned PDF without converter | Ask user to paste plain text or supply a Markdown summary. |
| F-5 | `--voice` without `--drill` | Reply "Voice mode applies to drill only. Run with `--drill --voice`." Continue without voice. |
| F-5b | `--voice --drill` set but companion runner not attached after 30 s | Print "No voice runner attached after 30 s. Continuing in text mode. In another terminal: `python3 scripts/voice_runner.py {OUTPUT_DIR}/voice/`." Fall back to text drill. |
| F-6 | `--examiner-deck` supplied without `examiner_brief` in profile | Print warning, continue with standard deck. |
| F-7 | `--append` target file does not exist | Ask user to confirm: create new deck, or correct path. |
| F-13 | `--audio` set but `edge-tts` not importable | Print "Audio export requires `edge-tts`. Install with `pip install edge-tts` and re-run with `--audio`." Skip Phase 4.5; .apkg still exports without audio. Other formats unaffected. |

### 0.3 Create the session output directory `[both]`

```
{cwd}/viva-flashcard-output/{YYYY-MM-DD-HHMM}-{role}/
```

Use `date +%Y-%m-%d-%H%M`. If the directory already exists, append `-2`, `-3`, etc.

Files written into this directory across the session:

- `thesis_profile.json` — Phase 1 output (or symlink to the reused source)
- `deck.json` — Phase 2 generated deck (canonical machine-readable form)
- `deck.csv` — Phase 4 CSV export
- `deck.md` — Phase 4 Markdown export, grouped by content axis
- `deck.apkg` — Phase 4 Anki package (via `scripts/export_anki.py`)
- `deck.pdf` — Phase 4 printable index-card PDF (optional; only if Pandoc/LaTeX available)
- `audio/{card-id}-{front|back}.mp3` — Phase 4.5 TTS clips (only if `--audio`)
- `review_log.json` — Phase 5 drill log (only if `--drill`)
- `session-metadata.json` — written at end

---

## Phase 1 — Thesis Profile

This phase produces (or reuses) a `thesis_profile.json`. The schema and extraction procedure are documented in `references/thesis-extraction.md`.

**Reuse path (preferred)**: if `--reuse-profile <path>` was supplied, copy it into `{OUTPUT_DIR}/thesis_profile.json` and skip extraction. Record `profile_source: "reused"` in session metadata along with the source path.

**Extraction path**: in one main-session pass, parse the thesis material into the Thesis Profile schema. Required fields are listed in `references/thesis-extraction.md`. Record `profile_source: "extracted"`.

The Thesis Profile is read by the card-generator subagent in Phase 2. It is the single source of truth for "what's in the thesis".

---

## Phase 2 — Card Generation by Subagent `[both]`

### 2.1 Spawn the card-generator subagent

Use the Agent tool with `subagent_type: "general-purpose"`. **One subagent only** — study material does not benefit from internal/external examiner framing. Adversarial cards are generated as a tagged subset (`difficulty: stretch`).

Read `agents/card-generator.md` for the prompt template. The subagent reads:

- `{OUTPUT_DIR}/thesis_profile.json`
- `references/card-generation.md` (coverage matrix, per-axis card counts, geography rules, cite-pages rules)
- `references/flashcard-methodology.md` (active recall + spaced repetition rationale; Vitae four-question lens)
- `assets/deck_schema.md` (the card schema it must produce)
- `assets/card_examples.md` (worked examples across content axes)
- the active role file at `roles/{role}.md` (for content-axis priorities)

The subagent produces a deck of N cards (50 / 80 / 120 depending on `--deck-size`) following the coverage matrix in `references/card-generation.md` §"Coverage Strategy". It writes the deck to `{OUTPUT_DIR}/deck.json` per `assets/deck_schema.md`.

### 2.2 Validate the generated deck

After the subagent returns, run the validator checks listed in `references/card-generation.md` §"Validation Procedure":

- Card count within ±10% of the deck-size target.
- Every priority-1 content axis (from the role file) has ≥1 card.
- Every `lim-N` in `thesis_profile.limitations[]` has ≥1 anchored card (mandatory limitation coverage).
- ≥10% of cards are `form_axis: "geography"` if `--cite-pages` is on (default).
- ≥15% of cards have `difficulty: "stretch"`.
- Every card has all required schema fields.

If validation fails, re-spawn the subagent **once** with the diagnostic. If it still fails, surface F-8 and offer "proceed with partial deck".

### 2.3 Pre-show the deck to the user `[both]`

Display a summary, not the full deck — at 50–120 cards a wall of text is unhelpful. Format:

```
**Generated deck** — {N} cards ({core: n1}, {stretch: n2}; {K} content axes; {M} geography cards)

Coverage by content axis:
  topic_choice          {n}    e.g. "Summarise the thesis in 3 minutes"
  contribution          {n}    e.g. "Defend the originality of co-1"
  methodology           {n}    ...
  ...

Coverage by chapter:
  ch-1  {n} cards
  ch-2  {n} cards
  ...

Limitation coverage:
  lim-1 ✓ (2 cards)
  lim-2 ✓ (1 card)
  ...

Sample cards (one per content axis):
  [card-007 / contribution / stretch]
    Front: "Where exactly is the originality of co-1, and how do you defend its location?"
    Back:  3 key points + Ch 4 §4.3 (p.73)
  ...
```

Then ask:

> Approve as-is, drop cards by id, regenerate the whole deck, or re-roll specific cards by id? (Type `approve`, `drop card-007,card-013`, `regenerate`, or `reroll card-007,card-013`.)
>
> You can also ask to see the full deck (`show all`) or filter (`show contribution`, `show ch-3`).

Loop until the user approves. Each edit writes a fresh `deck.json` and increments a `revision` counter. Previous revisions archived as `deck-r{N-1}.json`.

---

## Phase 3 — (folded into Phase 2.3 — pre-show & approve)

The pre-show / approve loop in Phase 2.3 is the entire third phase. Listed separately in the phase map for clarity but executed in the same step.

---

## Phase 4 — Export `[both]`

After approval, generate export files. Each is independent — if one fails, others still proceed.

### 4.1 CSV (`deck.csv`)

Single-pass write from `deck.json`. Columns:

```
id,content_axis,form_axis,difficulty,anchor_id,front,back_key_points,back_evidence,back_trap,back_comparison,tags
```

`back_key_points` is `;`-joined; `back_evidence` is `;`-joined `loc|page|what` triples. Importable into Quizlet, RemNote, Mochi.

### 4.2 Markdown (`deck.md`)

Group by content axis. Within each section, one card per `### Q-XXX` block. Front in bold, back as a bulleted list with evidence inline. Readable on phone, scannable in a text editor.

### 4.3 Anki package (`deck.apkg`)

Run `python3 scripts/export_anki.py {OUTPUT_DIR}/deck.json {OUTPUT_DIR}/deck.apkg` (append `--audio-dir {OUTPUT_DIR}/audio` if §4.5 ran). The script uses the `genanki` library (require user has it: `pip install genanki`). Note type defined in `assets/anki_template.md`. Tags: `viva::{role}`, `axis::{content_axis}`, `chapter::{ch-N}`, `difficulty::{core|stretch}`. When `--audio-dir` is supplied and contains MP3s, the .apkg embeds them and Anki auto-plays per side.

If `genanki` is not installed, print:

> Anki export requires `genanki`. Install with `pip install genanki` and re-run with `--export anki` to retry. The CSV at `deck.csv` can be imported manually in the meantime.

Continue with other formats — do not abort.

### 4.4 PDF (`deck.pdf`) — optional

If Pandoc and a LaTeX engine are available, render `deck.md` to a printable index-card layout (4 cards per A4 page, Q on one side, A on the other). If not available, skip silently and note in `session-metadata.json`. This is a convenience — most users will study in Anki.

### 4.5 Audio (`audio/*.mp3`) — optional, only if `--audio`

Pre-render TTS audio for every card so Anki auto-plays it on the matching side, no companion runner required. This phase is purely additive — the .apkg, CSV, and Markdown exports above still work without it.

Run **before** §4.3 so `export_anki.py` can pick up the MP3s:

```
python3 scripts/render_audio.py {OUTPUT_DIR}/deck.json {OUTPUT_DIR}/audio/ \
    --voice {audio_voice} --rate {audio_rate}
```

- Defaults: `--voice en-GB-SoniaNeural`, `--rate +0%`. Override via `--audio-voice` / `--audio-rate` at skill invocation.
- Spoken form is produced by `scripts/voice_transforms.render_card_front` / `render_card_back`, so audio matches the wording the drill-mode runner would speak.
- Existing MP3s are not re-rendered — safe to interrupt and re-run on regenerate / reroll. After Phase 2.3 edits, the user can delete just the affected MP3s in `{OUTPUT_DIR}/audio/` and re-run §4.5 to refresh those clips only.
- If `edge-tts` is not importable, surface F-13: print install hint, skip §4.5, continue to §4.3 without `--audio-dir`. Other formats unaffected.

After §4.5 succeeds, §4.3 runs with `--audio-dir {OUTPUT_DIR}/audio` and the .apkg ships with embedded sound. If §4.5 was skipped or partially failed, §4.3 still runs (with whatever audio is present, or none).

---

## Phase 5 — Drill Mode (optional, only if `--drill`)

In-session active-recall loop. The skill picks cards, shows the front, waits for the candidate's typed answer, reveals the back, asks the candidate to self-grade Anki-style.

### 5.1 Card selection

- **First pass** (no `review_log.json` yet): round-robin through the deck in the order written to `deck.json`. Prioritise by `difficulty: core` first, then `stretch`.
- **Subsequent passes** (review log exists): SM-2-weighted. Cards graded `Again` come back within minutes; `Hard` within hours; `Good` next day; `Easy` 4+ days. The skill computes intervals from the log; it does not run the full SM-2 algorithm — it picks the card with the smallest `next_due - now` value, ties broken by `difficulty: core` first.

### 5.2 Drill loop `[both]`

For each selected card:

1. Print the front:
   > **Card {id}** — {content_axis} / {form_axis}{ / stretch if stretch}
   >
   > {front}
2. Wait for the candidate's typed answer. Acknowledge with one line ("Got it." — do NOT evaluate yet).
3. Reveal the back:
   > **Key points the back of the card lists:**
   > - {bullet 1}
   > - {bullet 2}
   > ...
   >
   > **Evidence:** {Ch X §Y, p.N — what}
   >
   > **Trap:** {trap line if present}
   >
   > **Comparison:** {comparison line if present}
4. Ask: `Self-grade — (a)gain, (h)ard, (g)ood, (e)asy?`
5. Append to `review_log.json`:
   ```json
   { "card_id": "card-007", "graded_at": "2026-04-29T15:32:00Z", "grade": "good", "answer_latency_seconds": 47, "answer_text": "..." }
   ```
6. Pick the next card per 5.1.

Stop when the candidate types `wrap up` OR after a session cap of 30 cards (default — tunable via `--drill-cap N`).

### 5.3 Drill summary

After the cap or `wrap up`, print a one-screen summary:

```
**Drill session complete** — {N} cards reviewed.

Grades:  Again {n1}   Hard {n2}   Good {n3}   Easy {n4}
Weakest content axes (most "Again"/"Hard"):
  - {axis}: {n} cards
  - {axis}: {n} cards
Weakest chapters:
  - {ch-N}: {n} cards

Next session — {N} cards due (smallest interval first).

Review log saved to {OUTPUT_DIR}/review_log.json.
```

### 5.4 Voice mode (when `--voice` is supplied)

Voice mode replaces §5.2's terminal I/O with a file-bridge to a companion process. The skill stays inline; audio I/O lives in `scripts/voice_runner.py`, which the user starts in a second terminal.

**Setup**

1. Skill creates `{OUTPUT_DIR}/voice/{inbox,outbox,audio}/` and prints:
   > Voice mode requested. In another terminal, run:
   >
   >   `python3 {SKILL_DIR}/scripts/voice_runner.py {OUTPUT_DIR}/voice/`
   >
   > Waiting up to 30 s for the runner to attach…
2. Skill polls for `{OUTPUT_DIR}/voice/READY` (written by the runner once it has probed TTS / STT / mic recorder). On READY: continue. On 30 s timeout: surface F-5b and fall back to text drill.

**Per-card protocol (replaces §5.2 steps 1–5 when voice is on)**

For each selected card with id `card-NNN`:

1. Skill writes `voice/inbox/NNN-front.txt` containing the speakable form of the front (use `scripts/voice_transforms.render_card_front`). Runner picks it up, speaks it, returns to idle.
2. Skill writes `voice/inbox/NNN-answer-prompt.txt` with the line "Press Enter, give your answer, press Enter again to stop." Runner speaks it, then captures the candidate's PTT response. When done, runner writes `voice/outbox/NNN-answer-prompt.txt` (transcript) and `voice/outbox/NNN-answer-prompt.meta.json` (latency, audio path, STT confidence).
3. Skill polls `outbox/` for the transcript and meta. Display to candidate: "You said: {transcript}".
4. Skill writes `voice/inbox/NNN-back.txt` with the speakable back (use `voice_transforms.render_card_back`). Runner speaks it.
5. Skill writes `voice/inbox/NNN-grade-prompt.txt` containing "Self-grade — again, hard, good, or easy?" Runner speaks, captures, transcribes, writes outbox files as in step 2.
6. Skill matches the grade transcript via `voice_transforms.match_grade`. On `None` or `stt_confidence < 0.6`, skill writes `voice/inbox/NNN-grade-retry.txt` ("Sorry — please say again, hard, good, or easy.") and re-tries once. On second miss, skill prompts for typed grade in its own terminal.
7. Skill appends to `review_log.json` with the extra fields:
   ```json
   {
     "card_id": "card-007",
     "graded_at": "2026-04-29T15:32:00Z",
     "grade": "good",
     "answer_latency_seconds": 47,
     "answer_text": "transcript",
     "answer_audio": "voice/audio/007-answer-prompt.wav",
     "stt_confidence": 0.91,
     "voice": true
   }
   ```

**Heartbeat / reattach**

The runner writes `voice/heartbeat` every ~5 s. If the skill sees no heartbeat update for 60 s, it warns "Voice runner appears stalled — continuing this card in text mode." and switches the *current* drill session back to text. Already-graded cards are preserved.

**Card-text adaptation**

`scripts/voice_transforms.py` rewrites card text for TTS:

- Anchor ids (`co-1`, `lim-2`, `rw-4`, `ch-3`) → `contribution 1`, `limitation 2`, `prior work 4`, `chapter 3`.
- `§4.3` → `section 4 point 3`. `p.73` → `page 73`.
- `Fig 4.2` / `Table 4.2` → `figure 4.2` / `table 4.2`.
- URLs stripped.
- Geography card fronts: "On page N" / "which page" → section-level wording (page numbers are awkward to recall verbally; the realistic spoken-examiner reflex is section-level).

**Exit / wrap up**

`wrap up` and the drill cap behave as in text mode. The summary is written to the skill terminal as usual; the runner is left running so the user can stop it with Ctrl-C in its terminal.

---

## Phase 6 — Output and Summary `[both]`

### 6.1 Session metadata

Create `{OUTPUT_DIR}/session-metadata.json`:

```json
{
  "schema_version": "1.0",
  "skill_version": "0.1.0",
  "created": "{ISO 8601}",
  "session_type": "viva-flashcard",
  "input_mode": "text",
  "role": "{role basename}",
  "candidate": { "thesis_path": "{path}", "name": "{inferred or provided}" },
  "thesis_profile_file": "thesis_profile.json",
  "profile_source": "extracted | reused",
  "profile_source_path": "{path if reused, else null}",
  "deck_size": "small | medium | large",
  "deck_card_count": {N},
  "deck_revisions": {N},
  "flags": {
    "cite_pages": true,
    "examiner_deck": false,
    "drill": false,
    "audio": false,
    "audio_voice": "en-GB-SoniaNeural",
    "audio_rate": "+0%",
    "append_target": null
  },
  "exports": {
    "deck_json": true,
    "deck_csv": true,
    "deck_md": true,
    "deck_apkg": "completed | skipped (no genanki)",
    "deck_pdf": "completed | skipped (no pandoc)",
    "deck_audio": "completed ({n} clips) | skipped (no edge-tts) | not requested"
  },
  "drill": {
    "cards_reviewed": {N|null},
    "review_log_file": "review_log.json|null"
  },
  "output_directory": "{absolute path}"
}
```

### 6.2 Present to user `[both]`

> **Flashcard deck complete.** {N} cards ({n_core} core, {n_stretch} stretch) covering {K} content axes and {M} chapters.
>
> Exports written to `{OUTPUT_DIR}/`:
> - `deck.apkg` — import into Anki
> - `deck.csv` — import into Quizlet / RemNote / Mochi
> - `deck.md` — readable on phone or in editor
> - `deck.json` — canonical, editable
>
> {if drill ran: "Drilled {N} cards this session — see `review_log.json`."}
> {else: "Run with `--drill` to start an in-session active-recall loop."}
>
> To add cards later, re-run with `--append {OUTPUT_DIR}/deck.json` so Anki review history is preserved.

---

## Error Handling Reference

| Code | Trigger | Action |
|---|---|---|
| F-1 | Thesis missing AND no `--reuse-profile` | Ask user; do not silently proceed |
| F-2 | `--reuse-profile` path unreadable | List candidates; ask user to pick |
| F-3 | Role file missing | List available; ask user to pick |
| F-4 | `.docx` / scanned PDF without converter | Ask for plain text or Markdown |
| F-5 | `--voice` without `--drill` | Reject with hint; continue without voice |
| F-5b | `--voice --drill` but companion runner not attached after 30 s | Print install hint; fall back to text drill |
| F-5c | Voice runner heartbeat lost mid-drill | Warn; switch current drill session to text; preserve graded cards |
| F-6 | `--examiner-deck` without `examiner_brief` | Warn; continue with standard deck |
| F-7 | `--append` target missing | Ask user to confirm new deck or correct path |
| F-8 | Card generator violates coverage | Re-spawn once with diagnostic; if still failing, offer "proceed with partial deck" |
| F-9 | Anki export fails (no `genanki`) | Print install hint; skip; other exports continue |
| F-10 | PDF export fails (no Pandoc) | Skip silently; note in metadata |
| F-11 | User types `wrap up` mid-drill | Skip to drill summary |
| F-12 | Drill cap exceeded | Skip to drill summary |
| F-13 | `--audio` set but `edge-tts` missing | Print install hint; skip §4.5; .apkg exports without embedded audio; other formats unaffected |

## Graceful Degradation

| Available | Behaviour |
|---|---|
| Full deck + all exports | Standard run |
| Full deck, only CSV+MD exports | Skill complete; user installs `genanki` later and re-runs export |
| Full deck + .apkg without audio | `--audio` not requested OR `edge-tts` missing; skill complete; user can `pip install edge-tts` and re-run with `--audio` to add narration |
| Partial deck (subagent failed coverage) | Surface to user with shortfall list; offer proceed-with-partial |
| No deck (extraction failed) | Error report; user supplies better thesis material and re-runs |

---

## Why this shape (vs a generic flashcard tool)

**Why a single subagent.** A two-examiner panel exists to surface examiner *disagreement* in a defence simulation. Study material has no equivalent signal: the candidate is drilling against a fixed deck. Adversarial framing is preserved as a tagged subset (`difficulty: stretch`) rather than a separate generator.

**Why reuse `thesis_profile.json` when supplied.** Re-extracting the thesis costs tokens and may produce a slightly different profile, breaking anchor-id continuity across re-runs. Reusing guarantees that a card anchored to `co-1` always refers to the same contribution.

**Why mandatory limitation coverage.** The single most common failure mode in real vivas is the candidate being unable to defend a limitation they themselves declared. The deck guarantees at least one card per declared limitation.

**Why the geography subset is on by default.** UK examiners routinely ask "turn to page 73". A candidate who knows the content but not the geography of their own thesis fumbles. The geography subset trains the reflex. See `references/card-generation.md` §"Geography cards".

**Why no marking rubric.** Cards are not graded against QAA / FHEQ axes — drill the deck for weeks to build recall, then use a separate mock viva (with a marker) to surface what the deck missed. Self-grading here is Anki-style 4-button only.

---
> Source: [mildwall/viva-flashcard-skill](https://github.com/mildwall/viva-flashcard-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
