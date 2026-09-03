## mahjongindeler

> Organizer-side Blazor WASM PWA for a Dutch Mahjong club. Manages members, weekly sessions, tournaments, and the round-trip with the separate scoring app. No backend; everything is offline-first in `localStorage`.

# MahjongIndeler (organizer) — Claude session notes

## What this repo is

Organizer-side Blazor WASM PWA for a Dutch Mahjong club. Manages members, weekly sessions, tournaments, and the round-trip with the separate scoring app. No backend; everything is offline-first in `localStorage`.

Deployed at: `https://steffens-bridgemate.github.io/MahjongIndeler/`

The paired scoring app lives in the [MahjongScoring](https://github.com/Steffens-Bridgemate/MahjongScoring) repo and has [its own `CLAUDE.md`](../MahjongScoring/CLAUDE.md). The two together describe the full system.

## Projects

| Project | Type | Purpose |
|---|---|---|
| `Tsump` | Blazor WASM | The organizer app — members, weekly sessions, tournaments |
| `Tsump.Shared` | Razor Class Library | Models, codec, `ScoreTable`, `QrCodeModal`, `QrCodeRenderer`, `LanguageService`. Consumed by `Tsump` directly and by MahjongScoring via git submodule |
| `Simulation` | Console | Offline table-assignment test bench (not deployed): walk-forward replay of a real export across algorithm variants, and `--regen` to rewrite an export's assignments with the live algorithm (see [docs/domain.md](docs/domain.md#table-assignment-weekly)) |
| `Tsump.Scoring` (in this tree) | — | Build-output / IDE-restored stub only. **Source lives in the MahjongScoring repo.** Don't edit files here |

## Two-repo relationship & deploy dance

`Tsump.Shared` is the source of truth for both apps. MahjongScoring includes this whole repo as a submodule at `external/MahjongIndeler` and references the same `Tsump.Shared.csproj` from there.

**Wire-format note.** `ScoringInvite` / `ScoringResult` ([Tsump.Shared/Scoring/ScoringPayload.cs](Tsump.Shared/Scoring/ScoringPayload.cs)) are encoded by `ScoringPayloadCodec` into a **compact binary form** for QR-size reasons: each record is packed to a binary buffer (16-byte Guid, zig-zag varints, length-prefixed UTF-8), optionally Deflate-compressed (kept only when smaller), prefixed with a 1-byte type/compression header, then Base64Url'd into the URL fragment. There is **no JSON on the wire** — the `[JsonPropertyName]` short names on the records are now just documentation. `ContextId` (Guid) carries either a `Hanchan.Id` or a `TournamentSession.Id`. **No PlayerIds in either payload** — players are paired by position (index in `PlayerNames` = index in `table.PlayerIds`). The result's `Scores` is `List<int[]>` where each entry is `[endPoints, loan, penalty]`; use the `ScoreEndPoints` / `ScoreLoan` / `ScorePenalty` index constants on `ScoringPayloadCodec`. The invite's `Title` field carries **only the tournament's `ShortName`** (≤12 chars, or null for weekly) — *not* a full pre-formatted heading; the scoring app reconstructs `"Hanchan N — Table M"` from `SessionNumber`/`TableNumber` (keeps the QR small, see [docs/score-apply.md](docs/score-apply.md)). This was a value/interpretation change only — the binary layout is unchanged, so it was **not** a coordinated-deploy break (old links still decode). Breaking changes to the codec *layout*, however, do require coordinated deploys.

Push to `master` in this repo auto-triggers `.github/workflows/deploy.yml`, which bumps `Tsump/AppVersion.cs`, publishes, and deploys to Pages.

### Deploy commands — what to do when the user asks

These phrases map to **exact** procedures. Do all the steps, in order, in one go — don't stop to re-derive or investigate unless a command actually errors. A push is still gated on the user explicitly asking (which these phrases are).

| User says | Do exactly this |
|---|---|
| **"deploy organizer"** / "push organizer" | Commit + push organizer `master` (steps 1–2 below). Stop there. |
| **"deploy scoring"** / "push scoring" | Bump the MahjongScoring submodule + push (step 3 below). Stop there. |
| **"deploy both"** / **"full push"** / "full deploy" | **Both, in order: steps 1 → 2 → 3 below.** This is the default for any shared-code (`Tsump.Shared`) change, because a `Tsump.Shared` edit is not live until *both* repos redeploy. |

**"deploy both" / "deploy all" means the organizer + scoring app ONLY.** The standalone Riichi calculator app ([MahjongRiichiCalc](https://github.com/Steffens-Bridgemate/MahjongRiichiCalc), live at `https://steffens-bridgemate.github.io/MahjongRiichiCalc/`) is a *third* consumer of `Tsump.Shared`, but it rarely needs updating — so it is **never** included in "deploy both"/"deploy all". Redeploy it only when the user **explicitly and separately** asks (e.g. "deploy the calculator app"). To update it: `git submodule update --remote external/MahjongIndeler` in that repo, commit, push (auto-deploys).

**Step 1 — refresh docs (organizer).** Update the `docs/` files touched by the change (`docs/domain.md`, `docs/score-apply.md`, `docs/score-import-ui.md`, `docs/pages.md`, etc.) so they match what's deploying. Stage them in the same commit.

**Step 2 — commit + push organizer.**
```powershell
cd c:\Users\aners\source\repos\MahjongIndeler
git add <changed files>          # not .claude/settings.local.json
git commit -m "…"               # end with the Co-Authored-By trailer
git push origin master          # if rejected "fetch first": git stash push -- .claude/settings.local.json; git pull --rebase origin master; git push; git stash pop
```
The deploy workflow then adds its own `Bump version [skip ci]` commit — so the remote advancing under you is normal, and the rebase-on-reject above is the expected path, not a problem to investigate.

**Step 3 — bump the scoring submodule (only after step 2 has landed at `origin/master`).**
```powershell
cd c:\Users\aners\source\repos\MahjongScoring
git submodule update --remote external/MahjongIndeler
git submodule status               # verify pointer moved to the new organizer origin/master
# If any MahjongScoring callsite uses renamed payload fields / new classes,
# update Tsump.Scoring/Pages/ScorePage.razor in the same commit
git add external/MahjongIndeler [other-files]
git commit -m "Bump shared: …"     # end with the Co-Authored-By trailer
git push                            # auto-deploys the scoring app
```

For wire-format **breaking** changes, deploy MahjongScoring **first** *only if* active scoring links exist in WhatsApp; otherwise the 1→2→3 order above is fine.

**Never push without an explicit ask.** Push = deploy = live users.

## Coding practices

- **MUST READ before any UI/markup change: [docs/styles.md](docs/styles.md).** It is the house
  style (button colour semantics, sizing, flex action rows, `ExportButtonGroup`, inline-confirm,
  tabs, view-gating, auto-save). Match it rather than inventing new patterns; if a new pattern is
  truly needed, add it to that file in the same change.
- **Refactor to components and services to minimise duplication.** When the same logic appears in two pages, extract before adding a third. Recent precedents: [TableShareActions.razor](Tsump/Components/TableShareActions.razor), [ScoreImportPanel.razor](Tsump/Components/ScoreImportPanel.razor), [ScoreInviteService.cs](Tsump/Services/ScoreInviteService.cs), [IScoreContextResolver.cs](Tsump/Services/IScoreContextResolver.cs).
- **Use Segoe Fluent Icons / Bootstrap Icons** for button glyphs — not raw text or emoji.
- **Never bulk-rename strings** the user deliberately left at their current values. Ask if unsure.
- **Never commit or push without explicit approval.** Build locally first; then ask.
- **Don't launch the app yourself — no `run`/`verify` skills, no `dotnet run` to "check it works", unless explicitly asked.** The user does the manual/visual checks. Your job: build locally to confirm it compiles, then stop and report. (Building, and throwaway logic harnesses, are fine.)
- **Comments are for *why*, not *what*.** Skip docstrings that restate the signature. Note non-obvious invariants, workarounds, surprising decisions.
- **`Tsump.Shared` cannot reference `Tsump`.** Components that need `TournamentService` etc. must live in `Tsump/Components/`. Pure Razor (no Tsump-side deps) can live in `Tsump.Shared/Components/`.
- **Dutch term for "session" is "Zitting"** (not "Sessie"). Use it in `nl` strings in [LanguageService.cs](Tsump.Shared/Services/LanguageService.cs).

## Gotchas

- **Stale-reference after Save.** `SessionService.GetAllAsync` / `TournamentService.GetAllAsync` return freshly-deserialised objects on every call. After a save + reload, the in-memory `currentHanchan` / `tournament` you mutated is *not* the same instance as `hanchansOnDate[i]` / the freshly-fetched tournament. Re-point local references via `.FirstOrDefault(h => h.Id == …)` so subsequent edits land on the live entry. See `SaveHanchan` / `SaveScores` in [WeeklySessionPage.razor](Tsump/Pages/WeeklySessionPage.razor) and `ReloadTournament` in [TournamentDetail.razor](Tsump/Pages/TournamentDetail.razor).
- **`Score == null` tables render nothing.** [ScoreTable.razor](Tsump.Shared/Components/ScoreTable.razor) early-returns when `Table.Score` is null. Any path that reloads a container from storage (e.g. after import) must call `ScoreTable.InitializeScores(tables, …)` on every table — otherwise tables that never had scores saved disappear from the Scores tab until the user flips tabs and back.
- **QRs are pristine — no on-QR overlay.** Earlier iterations painted a coloured badge over the QR's centre or lower-right quadrant to distinguish organizer from scoring side; phones decoded fine but USB HID 2D scanners refused even small off-centre badges. Final design: the QR itself is unadorned; the distinguishing badge lives in the modal header (rendered as inline SVG by [QrCodeModal.razor](Tsump.Shared/Components/QrCodeModal.razor)) and as a coloured title band in the PNG copied to clipboard (handled by `copyQrImageToClipboard` JS, called with `headerBg` / `headerFg` arguments). `Overlays.Organizer` (blue clipboard) and `Overlays.ScoringResult` (green check) records still exist in [QrCodeRenderer.cs](Tsump.Shared/Scoring/QrCodeRenderer.cs) but only supply the glyph and colour to those two consumers.
- **QR SVG mechanics**: ECC level **M** (smallest matrix with a real safety margin — H was only needed for the since-removed centre badge; lower ECC ⇒ fewer modules ⇒ larger, more scannable modules, which matters for the small printed scoresheet QR), `width`/`height` stripped and a `viewBox` added — without viewBox, canvas rasterisers default to 300×150 and clip larger QRs.
- **Both pages share the import/share components.** [TournamentDetail.razor](Tsump/Pages/TournamentDetail.razor) and [WeeklySessionPage.razor](Tsump/Pages/WeeklySessionPage.razor) both use `<ScoreImportPanel>` (with `OnApplied` reloading the page's data) and `<TableShareActions>`. There is no longer any inlined import/share code — change the components once and both pages get it. The two pages differ only in placement: Tournament puts the panel in a Regenerate+Import row gated by its inner tab; Weekly renders it at the top of the Scores tab.
- **`TournamentSession.Id` back-fill.** Stored tournaments from before the Id field existed deserialise with `Guid.Empty`. `TournamentService.GetAllAsync` assigns a new Guid and re-saves on first load. Side effect: opening Tournaments after upgrading triggers a silent storage write.
- **PWA service worker — update behaviour now DIVERGES per app.** Both apps still use Blazor's `service-worker.published.js` (cache-name keyed on `assetsManifest.version`); the SW file itself is unchanged in both — it does **not** call `skipWaiting()` on install, `clients.claim()`s on activate, and keeps a dormant `message`→`skipWaiting` handler (responds to the literal string `'SKIP_WAITING'`). What differs is each app's **page registration script** in `index.html`:
  - **Organizer = passive restart-to-update (no in-place reload).** The new worker pre-caches and *waits*; it's never activated in place. The script detects the waiting worker and notifies the shared [AppUpdateBanner.razor](Tsump.Shared/Components/AppUpdateBanner.razor), which just *informs* ("Close the app and reopen to update") with a **Dismiss** button. There is **no** "Update now"/`controllerchange→reload` path: the earlier auto-reload risked corrupting data, since organizer saves are read-modify-write of a whole collection across `await`s (see [SessionService.cs](Tsump/Services/SessionService.cs)). `window.close()` is blocked everywhere, so the banner just reappears each launch until the user restarts. The dormant `SKIP_WAITING` handler stays for backward compat (an old deploy's "Update now" button posts it) — don't remove it.
  - **Scoring app = aggressive auto-update (reloads in place).** Its registration script posts `'SKIP_WAITING'` to the waiting worker the moment an update installs (and re-checks on `focus`), then reloads on `controllerchange`. `<AppUpdateBanner />` and the `registerAppUpdateCallback` bridge are **removed** from the scoring app ([MainLayout.razor](../MahjongScoring/Tsump.Scoring/Layout/MainLayout.razor)). Rationale: scoring is used by inexperienced players who won't know how to fully close a PWA on iOS/Android, and its saves are one small per-table key (not a whole-collection read-modify-write), so a mid-action reload is low-risk. Caveat: a reload mid-entry can lose *not-yet-balanced* typing (scoring only persists once a table balances) — accepted trade-off.
  Consequence: the two registration scripts now **intentionally differ** between repos — they are no longer kept in sync (only the SW file + `AppUpdateBanner`/`registerAppUpdateCallback` on the *organizer* side are shared).

## Detailed docs

Loaded on demand. Read the one(s) relevant to the current task:

- [docs/styles.md](docs/styles.md) — **UI house style (must-read for any markup change)**: button colour semantics, sizing, flex action rows, `ExportButtonGroup`, inline confirmations, tabs, view-gating, auto-save.
- [docs/domain.md](docs/domain.md) — data model: `Hanchan` vs `TournamentSession`, `Member` vs `TournamentParticipant`, `TableAssignment`, Mr. X, Uma, score-status classification.
- [docs/score-apply.md](docs/score-apply.md) — payload codec, `IScoreContextResolver` strategy, `ScoreImportService.ApplyAsync` flow.
- [docs/score-import-ui.md](docs/score-import-ui.md) — `ScoreImportPanel` state machine, four input methods (clipboard / HID scanner / camera / file), inactivity timer, auto-return, QR overlays.
- [docs/pages.md](docs/pages.md) — page-specific behaviour worth knowing (only where non-obvious cross-file behaviour exists).
- [docs/export-copy-print-download.md](docs/export-copy-print-download.md) — the Copy/Print/Download export pattern: the real `window.*` JS function names in `index.html`, `ExportButtonGroup`, the shared rankings builder in `RankingTable`, and the per-page `WrapAsHtmlDocument` gotcha.

---
> Source: [Steffens-Bridgemate/MahjongIndeler](https://github.com/Steffens-Bridgemate/MahjongIndeler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
