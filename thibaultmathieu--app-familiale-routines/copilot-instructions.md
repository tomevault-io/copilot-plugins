## app-familiale-routines

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git workflow

After every meaningful change (new feature, bug fix, refactor), commit and push immediately:

```bash
git add <specific files>
git commit -m "feat|fix|refactor|docs: short description"
git push
```

Use conventional commit prefixes: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`. Never batch unrelated changes into a single commit. Always push after committing so GitHub stays up to date.

## Commands

```bash
npm run dev          # Start dev server (http://localhost:5173/app-familiale-routines/)
npm run build        # TypeScript check + Vite production build
npm test             # Vitest characterization suite (useAppState: migrations, rewards, lifecycle)
npm run sync-assets  # Copy images/music from source folders → public/, regenerate manifests
```

The Vitest suite in `src/hooks/useAppState.test.ts` is the contract for state logic — run it after any change to `useAppState.ts`, `rewardImages.ts` or the migration chain. `npm run build` (tsc) is the second gate.

## Architecture

**Stack**: React 18, TypeScript, Vite 6, Tailwind CSS 3, PWA (vite-plugin-pwa / Workbox), Vitest

**State management**: Single hook `src/hooks/useAppState.ts` — all app state lives here, persisted to `localStorage` via `useLocalStorage`. No external state library.

**Design system**: warm tokens in `tailwind.config.js` (`warm`, `ink`, `line`, `success`, `honey`, `danger`, `night` + `shadow-card/raised/overlay` + `z-overlay/modal/toast`), brand fonts self-hosted via `@fontsource-variable/fredoka` (display) and `@fontsource-variable/nunito` (body), UI primitives in `src/components/ui/` (Card, Button, Pill, Badge, Overlay, ScreenHeader, IconButton, TextInput, FieldLabel). Child identity colors live in `src/theme.ts` (`COLOR_PALETTE`, `tint()` for translucent surfaces, `childTextColor()` for AA-readable colored text). Never concatenate hex + alpha (`color + '15'`) — use `tint()`.

**Screens** (rendered by `src/App.tsx` based on `currentScreen`):
- `home` → `HomeScreen` — routine launcher + timer + parent-gated creation (custom routine form + "Mission express") + universe-unlock banner + "all routines done → new day" banner. Parent access and reward-bearing creation share one **long-press (2 s) gate** (`startLongPress`), with the optional 4-digit `ParentGate.tsx` on top when a PIN is set
- `routine` → `ActiveRoutineScreen` — split-screen (2 children side by side), sounds, music, timers, universe-unlock choice after celebration
- `parent` → `ParentPanel` — reset/stop controls, timer launcher, sanctions, universe access, optional 4-digit parent PIN (`PinSetupOverlay.tsx`/`PinPad.tsx`)
- `gallery` → `GalleryScreen` — per-child reward image collection, **one section per owned universe** on a single scrollable page, shows progress to next universe unlock
- `universe-select` → `UniverseSelectScreen` — per-child universes (parent side): switch among owned, grant locked ones early

**Onboarding** (`OnboardingScreen.tsx`, when `onboardingCompleted` is false): welcome → children (photo strongly suggested, color picked with the child) → starting universe per child → routines. A static splash in `index.html` (`#splash`) shows before React mounts; `App.tsx` fades it out. Logo source: `public/icons/icon.svg` (`AppLogo.tsx` inline copy; `node scripts/generate-icons.mjs` regenerates the PNG icons).

**Universe system (schema V7)**: each child has `universeId` (active pool key in `rewardManifest.ts`) and `unlockedUniverseIds` (owned universes). Universe metadata lives in `src/data/universes.ts`. Pool resolution: `getRewardImagesForChildEntry()` in `src/data/rewardImages.ts` (falls back to legacy index round-robin). Per-universe progress = intersection `unlockedImages ∩ pool`, so switching universes loses nothing. Rewards draw from the union of all owned universes; a cycle reset clears every owned universe's ids once their combined pool is exhausted (see Reward system). To add a universe: drop an image folder in `images_rewards/<Name>/` (or generate one via `node scripts/generate-universe-images.mjs <id>` — reads `GEMINI_API_KEY` from the local vault `C:/secrets/keys.env` first, env var as fallback; never committed), run `npm run sync-assets`, add an entry in `universes.ts` (id = slugified folder name).

**Universe progression** (`src/data/universeProgress.ts`): gentle gamification — a child's day counts (`routineDayCount`, once per local day via `lastRoutineDay`) only when **all of today's scheduled routines that apply to that child were started today AND completed** (`childDayComplete`, same rule as the home "new day" banner; a day with no scheduled routines counts as soon as any routine/mission completes). `advanceDayProgress` applies this in `toggleTask`/`updateRoutine`/`deleteRoutine` — days never roll back. A child may own `allowedUniverseCount(dayCount)` universes: 1 at start, +1 at 2 days, then +1 every 5 days. When `pendingUniverseChoices() > 0`, the child picks a new universe (`UniverseUnlockOverlay.tsx`, offered after celebration and via a home banner). Parents can grant any universe early from `UniverseSelectScreen`. No streaks, no loss mechanics — keep it that way.

**Guard rails**: custom routines and "Mission express" tasks created from Home are `ephemeral: true` and purged on "Nouvelle journée"/stop (editing a custom one in the routine editor makes it permanent). Creating either unlocks images, so both sit behind the parent long-press gate (+ PIN if set) — kids can launch/complete but not mint reward sources. `launchRoutine` skips children with zero applicable tasks (no empty "instantly complete" routine). Caps: `MAX_ROUTINE_TEMPLATES` (30), `MAX_ACTIVE_TIMERS` (6). Onboarding drafts persist in localStorage (`routines-onboarding-draft`/`-step`) and are cleared on completion.

**Per-task child assignment**: `TaskTemplate.childIds` (undefined = all). Edited via `ChildTargetPicker.tsx` (chips: Tous • child…) in the routine editor (per task) and the Home custom/mission forms (per routine/mission). Selection logic lives in pure `src/data/childTarget.ts` (`toggleChildTarget` — from "Tous", tapping one child selects only that child; tested in `childTarget.test.ts`). `launchRoutine`/`updateRoutine` filter each child's tasks by `childIds`.

**Touch scrolling**: full-screen scroll containers carry `overflow-y-auto scroll-touch` (`.scroll-touch` in `index.css` = iOS momentum + `touch-action: pan-y` + `overflow-x: hidden`) so swipes scroll across the whole width on iPad, including the margins beside centered content.

**Asset pipeline**: Raw images live in `images_rewards/<Universe>/` folders (not committed). `scripts/sync-assets.js` copies and renames them to `public/rewards/{universe}/`, purges orphan destination pools, and generates two TypeScript manifests:
- `src/data/rewardManifest.ts` — per-child image arrays (auto-generated, do not edit)
- `src/data/musicManifest.ts` — music track list (auto-generated, do not edit)

Run `npm run sync-assets` after adding/removing source images or music files.

**Reward system**: Each child draws from the **union of all owned universes** (`unlockedUniverseIds`). `unlockReward()` in `useAppState` computes the pick BEFORE `setState` (deterministic return value) and awards the first still-locked image of the child's **daily draw order** (`dailyDrawOrder` in `src/data/mystery.ts` — deterministic seeded weighted shuffle per child+day); when every image across all owned universes has been collected, it clears those ids from `unlockedImages` and increments `completedCycles`. It also increments `totalUnlocked` (all-time counter, never decreases — sanctions and cycle resets don't touch it). Per-universe progress is still the intersection `unlockedImages ∩ pool`, so the gallery shows one filling section per owned universe.

**Rarity** (`src/data/rarity.ts`): each pool deterministically gets ~2 `legendaire` + ~5 `rare` images per 30 (djb2-hash ranking — same images are rare for everyone, no storage). `rarityWeight` biases the daily draw order so rarer images tend to come later. Shown in the celebration (badge + glow), gallery (gold frames/badges; locked legendary slots are teased with 🌟) and printed cards.

**Mystery image** (`src/data/mystery.ts`): `mysteryImageFor(child)` = first still-locked image of today's draw order — i.e. exactly what the next unlock will award. The home screen shows it blurred per child (anticipation without lying). After each unlock the mystery advances to the next locked image.

**Celebration ritual** (`CelebrationOverlay.tsx`): the reward arrives face-down (wiggling card back); the child taps to flip (3D flip + CSS confetti, more intense for rare/legendary). Auto-reveal after 15 s, auto-close 7 s after reveal. Flip/3D/confetti/print CSS lives in `index.css`.

**Bonus rewards** (`src/data/bonusRewards.ts`, V8): parents configure real-world rewards (`bonusRewards` in persisted state: label, emoji, `threshold`, optional `childIds`) triggered when a child's `totalUnlocked` reaches the threshold. Reached bonuses show as a home banner + gallery line until a parent taps "marquer remis" (`markBonusGiven` → `child.claimedBonuses`). Managed in `ParentPanel` (add/delete/mark given).

**Printing** (`PrintSheet.tsx`, from `ParentPanel`): a child's unlocked images as trading cards (63×88 mm, 9/A4 page, universe + rarity footer) or 35 mm sticker sheets. `@media print` rules in `index.css` isolate `.print-sheet`; `window.print()`.

**Sounds**: `src/hooks/useSound.ts` uses Web Audio API oscillators — no audio files needed. Three sounds: task complete (ding), routine complete (arpège), timer end (two tones).

**Music**: `src/hooks/useMusic.ts` uses HTML5 Audio. Triggered only when **both** children complete the `evening` routine. One random track plays looped at volume 0.3. The stock (copyrighted) tracks were removed on 2026-07-02 — `music/` is empty and `musicTracks` is `[]` (every hook path guards on that), pending original Suno tracks: drop `.mp3` files in `music/` (filename becomes the title) and run `npm run sync-assets`.

**Timer**: `ActiveTimer` objects stored in state with `startedAt` ISO timestamp — surviving page reloads. `useTimerTick` recalculates remaining time from elapsed ms. Timers are launched from `ParentPanel` and displayed in `ActiveRoutineScreen` as SVG ring animations.

**Key types** (`src/types.ts`): `Child` (has `unlockedImages`, `completedCycles`), `ActiveTimer` (has `childIds`, `durationSeconds`, `startedAt`), `AppState` (has `galleryReturnScreen` to know where gallery should return to).

**Gallery navigation**: `galleryReturnScreen` in state (`'routine'` or `'parent'`) controls where the back button goes. Set it before calling `setCurrentScreen('gallery')`.

**Migrations**: `migrateState()` in `useAppState` chains V1→V8 (V2 reward-id reset, V3 scheduledDays, V5 onboarding flag, V6 universeId assignment, V7 legacy-pool remap + dead reward-id cleanup + pipi preset removal + universe progression init, V8 bonus rewards: `totalUnlocked` seeded from current collection + `claimedBonuses` + `bonusRewards`). Bump `CURRENT_SCHEMA_VERSION` and add a guarded block + tests for any persisted-shape change.

---
> Source: [thibaultmathieu/app-familiale-routines](https://github.com/thibaultmathieu/app-familiale-routines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
