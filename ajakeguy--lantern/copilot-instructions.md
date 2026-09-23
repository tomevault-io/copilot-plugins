## lantern

> > **Standing instruction to Claude Code: this file is living documentation.**

# LANTERN — Project Constitution

> **Standing instruction to Claude Code: this file is living documentation.**
> You are responsible for keeping it current. At the end of any session that changes
> architecture, adds/modifies a game, changes the phonics scope, or makes a design
> decision, update the relevant section AND the Build Log below — without being asked.
> If a decision in-session contradicts this file, flag the conflict, resolve it with
> the developer, then record the outcome here. This file is the single source of
> truth; no context should need to be passed in from outside it.

*(Name settled: **Lantern**. GitHub repo: `Lantern`.)*

---

## What this is

A web app (React, hosted on Vercel) that builds advanced early literacy skills for
kids roughly ages 3–8. Built by a parent, for their kids first, with care and craft.
Not an edu-tech product clone.

## Philosophy — the part that must never erode

**Reading reveals the world.** Words are not a school subject bolted onto life;
they are how a person illuminates what was already there. A lantern doesn't create
the room — it shows it to you.

Operationalized as one design law:

> **Reading must have consequences. Never "read this, get a point" —
> always "read this, and something becomes possible that wasn't."**

Decode a word → the thing appears. Read a note silently → you know a secret nobody
said out loud. Read a sentence → you can build the scene it describes. Every
mechanic must pass this test before it ships.

### Two strands, one roof

- **SOUND** (learning to light the lantern): phonemic awareness → letter-sounds →
  blending → automaticity. Audio-forward, out-loud, phonics-driven.
- **LIGHT** (learning to see by it): silent reading, comprehension, meaning,
  stamina. Deliberately audio-restrained — silent reading is the point.

The app shifts weight from Sound to Light as the child progresses. Branding and
parent-facing copy should always convey both halves ("learning to light it /
learning to see by it"). The name must never read as phonics-only.

### Hard rules (violations are bugs)

1. **No mascot, no guide character, no digital figurehead.** The world teaches
   through its mechanics. The child is the protagonist. The only correspondent in
   the app is a real human (see Family Post). Do not add a helper character, a
   talking animal, or a narrator persona, even as "just a small touch."
2. **No engagement-farming.** No points, streaks, confetti storms, daily-login
   rewards, countdown timers framed as pressure, or variable-reward loops. The
   reward system is the Field Journal (below) and the feeling of capability.
3. **Errors are never punished.** A wrong blend makes the wrong (funny) thing, not
   a buzzer. Wrong comprehension leads somewhere silly, not to a red X.
4. **Screen is a springboard, not a destination.** Off-screen writing and offline
   games are first-class features, not extras. The ✎ handwritten-word mechanic and
   the Grown-ups offline games must survive every redesign.
5. **Nonsense words are sacred.** Decodable non-words (zib, mip) summon collectible
   creatures. This is disguised assessment — a child reading nonsense is provably
   decoding, not memorizing. Never remove or trivialize it.
6. **No third-party analytics, ads, or accounts beyond what hosting requires.**
   This is for children.

## The reward system: the Field Journal

Every word a child reads becomes a specimen card. Cards grow in with re-reading:
**sketch → inked → full color** (4 clean reads). Counters track *sounds met*,
*specimens*, *fully inked* — capability measures, never scores. Cards carry a ✎
toggle: "I wrote this by hand." Shelves: Things / Odd Menagerie (nonsense
creatures) / Words / Wild Words. The journal is the child's owned, growing proof
of their own knowledge. Procedural creatures are deterministic per word (same word
= same creature, forever).

**The Grove** is the journal's inhabited twin: every summoned thing and creature
lives permanently in one home scene (first tab; the boot-default tab once any
specimen exists). Creatures render at their journal stage — re-reading a word
visibly grows its creature in the world. Populated entirely from `specimens{}`;
no separate state. Tap a resident → it says its name.

## The ladder — games by band

Bands unlock by demonstrated skill, never by age. Overlap is expected.

### Band 0 — Ears only (~3–4) · Strand: Sound
No child-facing screen games. The app coaches the PARENT with offline games tuned
to progression: Sound Chain, robot-talk blending ("c...a...t — what did I say?"),
rhyme hunts, Menu Hunt, Silly Sentences. Phonemic awareness with zero letters is
the strongest predictor of reading success; it belongs off-screen.

### Band 1 — Cracking the code (~4–5) · Sound-heavy
- **Letter Safari** — the lantern mechanic: scenes are dark; a draggable light
  reveals letters camouflaged into naturalist SVG artwork; tap a lit letter →
  sound, and it morphs into its mnemonic. Once all letters are found, the
  **echo round** reverses the question (hear a sound, find its letter) and
  masters the scene (`scenesMastered`). Darkness is quiet, never a buzzer;
  keyboard focus counts as light. [BUILT v1]
- **Blending Bench** — build CVC words from sound tiles; lever stretches, snaps,
  summons the thing (real) or a creature (nonsense). Physical blend: slots pull
  apart during the stretch, rush together on the snap. Optional **summon-request
  notes** (📜 Requests): a note asks for a specific buildable thing, readable
  only — no audio button, by design (bridge toward Quiet Notes). Stateless;
  pool = 3-letter c·v·c THINGS made of met sounds. [BUILT v1]
- **Make It So (3-word)** — read "the cat sat," arrange the scene to match.
  First comprehension moment; comprehension checked by ACTION, never quiz.
- **The Grove** — persistent home scene for everything summoned at the Bench;
  populated from `specimens{}`, creatures shown at journal stage. Landing tab
  for returning devices (within /play). [BUILT]
- **Wild Words** — high-frequency rule-breakers (the, said, was, of) taught as
  delightful outlaws, known by sight. Own journal shelf. NEVER let the Bench
  blend these letter-by-letter — it would teach them wrong.
- **Word Rapids** — automaticity engine. Real words drift by among nonsense;
  grab the real ones before they sink. Must feel like reflexes sharpening,
  never like a timed test. This fills the decoding→fluency gap; do not cut it.

### Band 2 — Fluency & first meaning (~5–6) · Strands merge
- **Word Forge** — hear it, build it (encoding; reverse of Bench).
- **Word Surgery I** — syllable splitting/recombining (din-o-saur).
- **Quiet Notes** — sealed notes read SILENTLY (no audio button, by design),
  acted on: "the smallest frog hides under the blue rock." Reading = secrets.
- **Fork Tales** — decodable micro-stories where reading drives branching
  choices. Wrong turns are funny, not fatal. Connected-text stamina.
- **Family Post opens** — see below.

### Band 3 — Power reading (~6–8) · Light-heavy
- **Word Surgery II** — morphology: prefixes, suffixes, roots (un+happy,
  jump+ing). This is where "advanced" lives.
- **Fork Tales: longer arcs** — multi-session stories.
- **Quiet Notes: inference tier** — notes that require reading between lines.
- **Family Post as endgame** — ceiling-less because the correspondent is human.

### Family Post (spans bands 2–3, the app's soul)
A mailbox where a REAL person (parent, grandparent — remote OK) leaves notes at
the child's level. Replies must be handwritten and shown to the camera. Real
audience, real relationship, no digital figurehead. Parent-side tooling suggests
phrasing at the child's current decodable level.

### Parked (do not build without discussion)
- **Label the World** (photograph + label real objects) — overlaps Family Post
  and ✎ quests; camera/storage complexity not yet justified.
- **Pen Pal with an AI correspondent** — rejected: no digital relationships.

## Phonics scope & sequence

Letter introduction follows SATPIN-style grouping (early letters chosen to make
many words fast): s a t p i n → m d g o c k → e u r h b f l → (extend: j v w x
y z qu → digraphs sh ch th ng → long vowels/silent-e → vowel teams → r-controlled).
Journal data drives what unlocks: sounds met gate Bench tiles; Bench success gates
Make It So; automaticity (Word Rapids) gates Band 2. When extending word lists,
every word must be decodable with only the sounds the child has met, except Wild
Words, which are explicitly flagged as rule-breakers.

## Technical

- **Stack:** Vite + React, plain CSS (no Tailwind dependency), deployed on Vercel
  from GitHub main.
- **Persistence:** localStorage behind a two-function adapter (loadState/saveState)
  so a future backend is a swap, not a rewrite. Schema versioned
  (`lantern:v1`); migrate forward, never wipe a child's journal.
- **State spine shared by all games:** `soundsMet[]`, `specimens{}`,
  `scenesDone[]`, `scenesMastered[]` (+ future: `wildWords[]`,
  `automaticity{}`). Games read/write the spine; no game owns private
  progression that another game needs. New fields are added to
  `DEFAULT_STATE` in App.jsx and loads MERGE over it — old journals gain
  fields, never lose them.
- **Audio:** browser speechSynthesis is the v0 placeholder and is known-bad for
  isolated phonemes (adds schwa: "tuh" for /t/). Priority upgrade: recorded
  phoneme audio files. All audio flows through `src/lib/audio.js` — the single
  seam the recorded swap will touch. Rules learned the hard way: never hand the
  engine a single letter (it says the letter NAME) or a repeated-letter string
  like "sss" (it spells it out); SOUNDS entries therefore split `label` (what
  print shows) from `tts` (engine-safe syllable, tunable by ear in
  `src/data/words.js`). Two fixed rates only (`PHONEME_RATE`, `WORD_RATE`);
  utterances are queued one-per-sound so repeat counts are exact; `cancel()`
  is followed by an ~80ms defer (Chrome clips otherwise). Parents can pick the
  voice in the Grown-ups tab; stored under device-level key
  `lantern:settings:v1` — settings belong to the device, the journal to the
  child; never mix them.
- **Routing:** no router dependency — `src/lib/router.js` (History API).
  `/` and `/grown-ups` = parents page for EVERY visitor (the front door; no
  journal-based redirect — decided 2026-07-27); `/play` = the games
  (App.jsx). `vercel.json` rewrites all paths to `index.html`.
  `src/Root.jsx` owns route selection.
- **Parents page:** `PARENTS_PAGE.md` (repo root) is the copy source of truth —
  edit it first, then mirror verbatim into `src/data/parents.js`. Game cards
  render from the single `GAMES` config there (built vs coming-soon), grouped
  by band, in the fixed four-part format. Future games must ship with their
  card written (spec first). No testimonials, no signup capture.
- **Layout:** Vite app at repo root. `src/Root.jsx` (routing) → `src/App.jsx`
  (game shell + state spine) / `src/pages/ParentsPage.jsx`;
  `src/components/` (Safari, Bench, Journal, Grownups, Creature, GameCard),
  `src/data/` (words.js, parents.js), `src/lib/` (storage.js, audio.js,
  router.js, hash.js), `src/styles.css`. The original single-file seed
  (`Lantern.jsx`) lives in git history at the root commit.
- **Quality floor:** works on a phone held by a 4-year-old (big targets, no
  hover-dependence), visible keyboard focus, `prefers-reduced-motion` respected,
  no reading required to navigate for pre-readers (icons + audio affordances in
  Sound-strand navigation).

## Design language

Deep pine green (#12403A) + warm paper (#F2EBDA) with pink/blue/sun accents;
field-journal / naturalist-kit sensibility, not candy-arcade. Andika (a face
designed for early readers — single-story a, clear letterforms) for anything a
child reads; Lexend for UI. Calm > stimulating, always. Lowercase-first for
letter learning.

## Open questions

- [ ] Name is settled as Lantern (repo created 2026-07-27). Still to do: check
      domain/trademark availability before any public-facing branding hardens.
- [ ] Recorded phoneme audio: source (record ourselves vs licensed set)?
- [ ] Family Post remote correspondents: what's the simplest safe delivery
      (shared link? email-in?) with no accounts for the child?
- [ ] Multi-child profiles on one device?
- [ ] Cross-device persistence: the journal is device-local (localStorage).
      Developer floated Nostr as an opt-in identity/sync layer (2026-07-27,
      via NDK — encrypted app-data events on relays). Not decided. If ever
      pursued: parent-gated, opt-in, encrypted, child never touches keys;
      weigh against simpler options (export/import file, small backend) and
      hard rule 6. Local-first stays the default regardless.

## Build log

*(Claude Code: append an entry — date, what changed, why — every session that
alters code, scope, or decisions. Newest first.)*

- **2026-07-27** — Bench upgrades (enhancement 3 of 3): optional summon-request
  notes — a paper note asks for a buildable thing; deliberately no audio button
  (reading the note IS the game; quiet bridge to Quiet Notes); stateless, pool
  derived from THINGS × met sounds, deterministic order; fulfilling one names it
  in the reveal, wrong summons stay funny and unpunished. Physical blend pass
  (CSS only): lever gives under press, tiles land with a settle, slots pull
  apart on stretch and squash-settle on snap, reveal rises — all inside the
  global prefers-reduced-motion kill-switch.
- **2026-07-27** — The Grove added (enhancement 2 of 3): persistent home scene
  where every summoned thing/creature lives, populated purely from the
  `specimens{}` spine (no new state); creatures render at journal stage so
  re-reading visibly grows them; deterministic hash placement with shared
  collision-nudging (`src/lib/spots.js`, extracted from Safari). New first tab
  and boot-default tab once any specimen exists; Bench reveal copy names the
  Grove; parent card added to PARENTS_PAGE.md (spec-first) + parents.js.
- **2026-07-27** — Letter Safari reworked into the lantern mechanic (enhancement
  1 of 3): dark naturalist SVG scenes (SceneArt.jsx), draggable light (pointer
  drag only — a bare tap deliberately cannot self-light a letter), letters
  camouflaged in scene palette, tap-to-morph into mnemonic, echo round
  (sound → letter) persisting to new spine field `scenesMastered[]`; loads now
  merge DEFAULT_STATE forward. Hash spot placement gains deterministic
  collision-nudging (overlapping letters were untappable). Verified headless:
  unlit taps silent, drag-reveal, exact utterances, echo wrong-tap says only
  that letter's sound, mastery persists, old journals migrate.
- **2026-07-27** — Routing decision reversed by developer: `/` now shows the
  parents page to EVERY visitor, journal or not (the original spec redirected
  returning devices to the games; in practice that meant a parent who had
  played could never find the page at the root URL). PARENTS_PAGE.md
  implementation note updated to match. Games remain at `/play`.
- **2026-07-27** — Audio bugfix pass + parent home page. Fixed the Safari/Bench
  speech bugs (letters spoken as names — "kay kay cup" for c; "sss"-style
  strings spelled out so sounds played 2×–6×; four different rates; Chrome
  cancel-clipping): SOUNDS now splits `label` (print) from `tts` (engine-safe
  syllable), audio.js queues one utterance per sound at two fixed slow rates.
  Added parent voice picker (Grown-ups tab, `lantern:settings:v1`), "Clear the
  bench" button, and the parent-facing home page per PARENTS_PAGE.md (now the
  copy source of truth): `/` for first-timers → `/play` once a journal exists,
  `/grown-ups` always; tiny in-house router + vercel.json rewrite. Verified
  headless with a speechSynthesis-recording stub: exact utterance sequences,
  rates, voice persistence, routing, and all prior checks green.
- **2026-07-27** — First task done: git repo initialized on `main` (remote
  `ajakeguy/Lantern`), seed committed as root commit, then migrated to a
  Vite + React app. `Lantern.jsx` split into `src/App.jsx` +
  components/data/lib modules; storage adapter now localStorage behind the
  unchanged two-function `loadState`/`saveState` interface (`lantern:v1`);
  `speak()` isolated in `src/lib/audio.js` as the single audio seam. Behavior
  identical by design — only addition is a 3-line `html/body/#root` CSS reset
  the artifact host previously provided. Verified headless (Chrome): four tabs
  render, Safari taps register sounds, Bench blends and reveals, Journal
  persists across refresh, `vite build` clean. Deploys to Vercel from main.
- **2026-07-27** — Name settled as **Lantern**; GitHub repo created under that
  name. Seed file renamed `SoundKeeper.jsx` → `Lantern.jsx`; storage key set to
  `lantern:v1`; on-screen title updated.
- **2026-07-27** — Project constitution created in claude.ai design sessions.
  Decisions to date: no mascot (Letter Owl considered and rejected — discomfort
  with children bonding to a digital figurehead); "Sound Keeper" name rejected as
  phonics-only; two-strand Sound/Light structure adopted; ladder pressure-tested
  against scope-and-sequence, adding Word Rapids (automaticity gap), Wild Words
  (irregular high-frequency words), staging Word Surgery into I/II, moving
  Make It So earlier, Fork Tales later; Label the World parked. v0 single-file
  prototype exists (shell, Safari, Bench, Journal, Grown-ups).

---
> Source: [ajakeguy/Lantern](https://github.com/ajakeguy/Lantern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
