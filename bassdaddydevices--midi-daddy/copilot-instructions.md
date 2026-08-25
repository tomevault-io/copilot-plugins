## midi-daddy

> An Ableton Live Extension that inserts chord progressions and finds matching melodies from a

# MIDI Daddy

An Ableton Live Extension that inserts chord progressions and finds matching melodies from a
bundled library of 19,529 curated MIDI files, scale-aware via Live's Scale Mode.

**Status:** v1 implemented and merged to `main`. Two context-menu actions ship:
- **Insert Chord Progression** — `ClipSlot` (Session View) and `MidiTrack.ArrangementSelection`
  (Arrangement View). `src/actions/insertProgression.ts`.
- **Find Matching Melody** — `MidiClip` scope, with a dynamic "Paste Matching Melody" action
  registered on `ClipSlot`/`MidiTrack.ArrangementSelection` after a melody is chosen (a
  "clipboard" pattern built on the SDK's register/unregister capability, since there's no other
  way to do a two-step "pick, then place" interaction). `src/actions/findMelody.ts`.

End-to-end manual verification inside a running Ableton Live instance has not been done from
this environment (no Live available here) — `npm run package` produces a `.ablx` and `npm test`
covers all pure logic, but the actual context-menu/picker/clip-creation behavior is unverified
until someone installs the `.ablx` in Live and tries it.

## Architecture

Three layers (see the design spec and plan for full rationale):
1. **Library index** (`scripts/build-index.ts` → `assets/midi-index.json`) — derives each
   scale's interval set empirically by parsing its key-1 `Base Scale` MIDI file (not hand-typed
   music theory), then walks `assets/midi-library/MIDI SCALES/` for progressions and melodies.
2. **Scale resolution** (`src/scale.ts`) — `resolveScale(song, index)` maps Live's
   `Song.scaleIntervals`/`rootNote` to a library key/scale, or returns `null` (routes to the
   manual picker) if Scale Mode is off or unmapped.
3. **Actions** (`src/actions/`) — the two context-menu commands, built on `src/library/` (parsing
   + querying) and `src/ui/picker.ts` (the only "ask the user to choose" mechanism, since the SDK
   has no native dropdown/quick-pick).

## Key facts

- Built on the Ableton Extensions SDK v1.0.0-beta.0. Local SDK reference, docs, and examples
  live at `/Volumes/DEV/Ableton Projects/extensions-sdk-1.0.0-beta.0` — read it before changing
  any SDK-facing code. See the `ableton-extensions` skill for a summary.
- Design spec: `docs/superpowers/specs/2026-06-26-midi-daddy-design.md`.
- Implementation plan: `docs/superpowers/plans/2026-06-26-midi-daddy-implementation.md` (also has
  a per-task log of bugs found and fixed during implementation — worth reading before assuming
  any piece of the SDK API surface behaves a particular way).
- `Song.rootNote`/`scaleName`/`scaleIntervals`/`scaleMode` are read-only — this extension reads
  Live's current scale, it never sets it.
- v1 reads chords/melodies only from `assets/midi-library/MIDI SCALES/` (all 15 scales × 12
  keys). `assets/midi-library/MIDI CHORDS/` is bundled but unused until the v2 custom builder.
- **`assets/midi-library/` and `assets/midi-index.json` are gitignored, not committed** — they
  were stripped from git history entirely (19,529 files was deemed not worth carrying in the
  repo, even though the library is rights-cleared for distribution). Both still exist on disk;
  a fresh clone needs the library copied into `assets/midi-library/` manually, then
  `npm run build-index` to regenerate the index, before `npm start`/`npm run package` will work.
- **`getObjectFromHandle(handle, Type)` throws if the handle's actual type doesn't match `Type`.**
  An `ArrangementSelection`'s `selected_lanes` can span non-`MidiTrack` lanes even when the
  triggering scope is `MidiTrack.ArrangementSelection` (the scope describes what was *clicked*,
  not the homogeneity of the whole selection). Always resolve via `DataModelObject` first, then
  narrow with `instanceof` — matching the SDK's own bundled `examples/arrangementselection`.
  Resolving directly via a specific subclass crashed this exact way during implementation
  (Task 9's review caught it; see the plan).
- **`extensions-cli package` does not bundle `assets/` by default.** `npm run package` must pass
  `-i assets` or the produced `.ablx` is a few KB and missing the library/index entirely —
  silently broken at runtime with no build-time error. Already fixed in `package.json`; don't
  drop the flag if you ever rewrite that script.
- The SDK/CLI `.tgz` dependency paths in `package.json` are absolute, not relative — this project
  is sometimes worked from inside a git worktree, and a relative path's correct depth would
  depend on worktree nesting.
- `npm test`'s script passes `tsx --test` an explicit list of `*.test.ts` files (via `find`), not
  bare directory names — `tsx --test src scripts` throws `ERR_MODULE_NOT_FOUND` (the directory
  argument gets treated as a module specifier to resolve, not a directory to discover tests in).

## Commands

- `npm install` — install dependencies (pulls the SDK/CLI from local `.tgz` files).
- `npm run build-index` — regenerate `assets/midi-index.json` from `assets/midi-library/`.
- `npm run build:dev` / `npm start` — dev build / build + launch in Live (Developer Mode required).
- `npm run build` — production build.
- `npm run package` — production build + produce a `.ablx` for distribution.
- `npm test` — run the Node test-runner suite (pure logic only; no Live required).

---
> Source: [bassDaddyDevices/midi-daddy](https://github.com/bassDaddyDevices/midi-daddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
