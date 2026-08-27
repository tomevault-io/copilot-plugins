## lod-bot

> This is an automation engine for the non-player bot flowcharts in **Liberty or Death** (GMT Games, COIN Series Vol. V). It handles 0–4 bot-controlled factions (British, Patriots, Indians, French) via an interactive CLI. The goal is faithful implementation of the published flowcharts — not a variation, not a reinterpretation, not a "simplified" version.

# CLAUDE.md — Liberty or Death Bot Engine

## Project Summary

This is an automation engine for the non-player bot flowcharts in **Liberty or Death** (GMT Games, COIN Series Vol. V). It handles 0–4 bot-controlled factions (British, Patriots, Indians, French) via an interactive CLI. The goal is faithful implementation of the published flowcharts — not a variation, not a reinterpretation, not a "simplified" version.

**Language:** Python (100%). ~43,000 LOC across ~130 files.
**Tests:** pytest. 1,136 tests currently passing.
**No external game references.** Do not consult BoardGameGeek, other COIN games, other GMT titles, or any historical sources outside the Reference Documents.

---

## Source of Truth (Strict Hierarchy)

These are **read-only**. Never modify them. All other code must conform to them.

### 1. `lod_ai/rules_consts.py` — Canonical Labels
Every string label for factions, pieces, markers, leaders, and space IDs used anywhere in the codebase **must** come from this file. If a label doesn't exist here, it is wrong.

**Canonical piece tags:**
```
British_Regular, British_Tory, British_Fort, British_Regular_Unavailable, British_Tory_Unavailable
Patriot_Continental, Patriot_Militia_A, Patriot_Militia_U, Patriot_Fort
French_Regular, French_Regular_Unavailable
Indian_WP_A, Indian_WP_U
Village, Squadron, Blockade, Propaganda, Raid
```

**Faction strings:** `BRITISH`, `PATRIOTS`, `INDIANS`, `FRENCH`

**If you encounter a string literal in the code that doesn't match one of these** (e.g., `"Continental"`, `"Militia"`, `"British_cube"`, `"Tory"` used as a piece tag, `"fort"`, etc.), it is a bug. Replace it with the correct constant from `rules_consts.py`.

### 2. `Reference Documents/card reference full.txt` — Card Behavior
The authoritative definition of every card's unshaded and shaded effects. Card handler implementations in `lod_ai/cards/effects/` must match this file exactly — same targets, same piece types, same destinations ("to Casualties" vs "to Available" vs "remove"), same conditions.

### 3. `Reference Documents/` — Everything Else
All files in the `Reference Documents/` directory are source-of-truth materials. Always check the full directory contents rather than relying solely on this list. Key files include but are not limited to:

- `*bot flowchart and reference.txt` — Non-player decision trees (one per faction)
- `Manual Ch 1.txt` through `Manual Ch 8.txt` — Full rules
- `Manual Glossary.txt` — Definitions
- `leader_capabilities.txt` — Leader special abilities
- `1775 Scenario Reference.txt`, `1776 Scenario Reference.txt`, `1778 Scenario Reference.txt` — Setup data
- `map_base.csv` — Map topology (CSV; the only non-.txt reference file)
- `setup instructions.txt` — Scenario setup procedure
- `random spaces map.txt`, `random spaces table.txt` — Random space selection tables

If a file exists in `Reference Documents/` and is not listed above, it is still authoritative. Any file added to this directory in the future is automatically a source of truth.

All files are plain text (except `map_base.csv`). No PDFs, no images.

---

## Lessons learned (for future COIN-bot projects)

`coin-implementation-lessons.md` in the repo root distills ~40+ bugs
encountered in this project into 11 anti-pattern families, with
architecture/testing/process recommendations.  Audience is future
projects implementing other COIN-series games (or any tabletop
simulation with bot AI).  Not part of the runtime; not a code
reference; safe to ignore unless you're starting a similar project.

---

## Supplementary strategic reference (not a source of truth)

`strategy.md` in the repo root is a faction-by-faction, scenario-by-
scenario strategy guide for human (or LLM) gameplay decisions.  It is
**not** part of the runtime, **not** authoritative for rules questions,
and **not** a substitute for the bot flowchart references in
`Reference Documents/`.  Bot implementations must follow the published
flowcharts, not strategic intuitions from this guide.

Use it when you need to understand *why* a faction prefers one play
over another — for example, to author better commit messages, investigate
bot-flowchart anomalies, or write human-side CLI prompts.  See the file's
own "What this file is and is NOT" section.

---

## Project Structure

```
lod_ai/
├── __init__.py, __main__.py     # Entry point: python -m lod_ai
├── rules_consts.py              # SOURCE OF TRUTH — canonical labels
├── engine.py                    # Main game loop / turn sequencing
├── interactive_cli.py           # Human player menus
├── cli_utils.py                 # CLI helper functions
├── dispatcher.py                # Routes faction turns to bot or human
├── victory.py                   # Victory condition checks
├── bots/                        # Non-player bot logic (flowcharts)
│   ├── base_bot.py              # Shared bot infrastructure
│   ├── british_bot.py           # British flowchart implementation
│   ├── patriot.py               # Patriot flowchart implementation
│   ├── indians.py               # Indian flowchart implementation
│   ├── french.py                # French flowchart implementation
│   ├── event_instructions.py    # Per-card bot special instructions
│   └── random_spaces.py         # Random space selection
├── cards/
│   ├── __init__.py              # Card registry and lookup
│   ├── data.json                # Card metadata (titles, order, icons, years)
│   └── effects/                 # Card effect handlers
│       ├── early_war.py         # Cards from 1775-1776
│       ├── middle_war.py        # Cards from 1777-1778
│       ├── late_war.py          # Cards from 1779-1780
│       ├── brilliant_stroke.py  # Brilliant Stroke / Treaty of Alliance
│       ├── winter_quarters.py   # Winter Quarters special cards
│       └── shared.py            # Common card helper functions
├── commands/                    # Game operations
│   ├── battle.py, march.py, muster.py, rally.py
│   ├── garrison.py, gather.py, scout.py
│   ├── rabble_rousing.py, raid.py
│   ├── hortelez.py              # Hortalez et Cie (French supply)
│   └── french_agent_mobilization.py
├── special_activities/          # Special activities per faction
│   ├── common_cause.py, skirmish.py, naval_pressure.py  # British
│   ├── partisans.py, persuasion.py, common_cause.py     # Patriot (shares Common Cause)
│   ├── war_path.py, trade.py, plunder.py                # Indian
│   └── preparer.py                                       # French
├── board/                       # Board state management
│   ├── pieces.py                # Piece placement, removal, movement
│   └── control.py               # Space control calculations
├── state/
│   ├── setup_state.py           # Scenario initialization
│   └── map_loader.py            # Map data loading
├── map/
│   ├── adjacency.py             # Adjacency calculations
│   ├── control.py               # Map-level control queries
│   └── data/map.json            # Processed map data
├── economy/
│   └── resources.py             # Resource tracking
├── leaders/
│   └── __init__.py              # Leader placement, movement, capabilities
├── util/                        # Shared utilities
│   ├── caps.py                  # Piece cap enforcement
│   ├── eligibility.py           # Faction eligibility tracking
│   ├── year_end.py              # Year-end / Winter Quarters procedures
│   ├── naval.py                 # Naval/squadron/blockade logic
│   ├── validate.py              # State validation
│   ├── normalize_state.py       # State normalization
│   ├── piece_kinds.py           # Piece type queries
│   ├── loss_mod.py              # Loss modification
│   ├── free_ops.py              # Free operation handling
│   ├── history.py               # Game history tracking
│   └── adjacency.py, normalize.py
├── tools/
│   ├── card_audit_fix.py        # Audit tool for card compliance
│   └── smoke_test.py            # Quick smoke test
└── tests/                       # pytest test suite
    ├── test_*.py                # Various test files
    └── commands/test_*.py       # Command-specific tests
```

---

## Project Status (as of May 2026)

Phases 1, 2, and 3 of the original task plan are complete.  See
`audit_report.md` for the per-phase summary and `GITHUB_ISSUES.md` for
the original phase definitions.

**Phase 1 — Label Compliance: COMPLETE.**  All `.py` files in `lod_ai/`
have been audited and string-literal violations of `rules_consts.py`
fixed.  Any new code must continue to import constants rather than
using piece/faction/marker/space-ID string literals directly.

**Phase 2 — Card Compliance: COMPLETE.**  All card handlers in
`lod_ai/cards/effects/` have been audited against `Reference Documents/
card reference full.txt`.  31+ specific card mismatches documented and
fixed.  Brilliant Stroke / Treaty of Alliance interrupt chain
implemented per Manual §2.3.8-9 and §8.3.7.

**Phase 3 — Rules & Bot Compliance: COMPLETE.**  Bot flowcharts for
all four factions have been audited node-by-node against their
reference docs.  Commands, special activities, year-end / Winter
Quarters, victory conditions, and scenario setup have all been
verified against the Manual.  Zero-player mode runs to completion
across all three scenarios without crashes or hangs.

**Phase 4 — UI / Usability: PARTIALLY COMPLETE.**  Zero-player mode is
solid (see "Smoke matrix" below).  The human-player CLI (1–3 humans) now
has an automated QA harness (`lod_ai.tools.human_qa`, see below) that
drives the real game loop and exercises the previously-untested paths;
the last full cycle found zero defects.  Remaining human-mode work is
best done as live playtests for feel/UX rather than crash-finding.

### Smoke matrix (current main)

`python -m lod_ai.tools.batch_smoke --large` runs 150 zero-player games
(50 seeds × 3 scenarios) and produces:

- 0 crashes
- 0 hangs / timeouts (200-card safety cap never hit)
- 0 interactive-prompt leaks
- 0 illegal-action rejections
- 0 unhandled bot exceptions

### Invariant gate (per-card)

`python -m lod_ai.tools.clean_sweep_gate --seeds 1-20` (the CI gate, 60
games) asserts two invariants after **every card**, on top of the
zero-bot-error / zero-illegal-action / zero-free-op-skip checks:

1. **Canonical-schema validation** -- `validate_state()` must hold after
   every card, not just where it was previously called.
2. **Save/load round-trip** -- serializing the live state to the on-disk
   JSON form and loading it back must reproduce the same canonical state
   *and* the same RNG internal state. This catches silent state
   corruption and fields that do not survive persistence -- the class of
   bug that survives playtests.

Implemented in `lod_ai/tools/invariants.py`. Any violation writes a
crash-repro dump under `crash_dumps/` (scenario + seed + card number +
traceback + full serialized state) and the gate fails. The same
machinery wraps zero-player crashes in `batch_smoke`, so a crash also
emits a dump. Every dump names the one command that reproduces it:

    python -m lod_ai.tools.batch_smoke --repro <scenario>:<seed>

`batch_smoke --invariants` runs the 60-game default matrix with the same
per-card checks enabled.

### Soak + free-op decline audit

`python -m lod_ai.tools.soak --games 1000 --out soak.jsonl` runs a broad
zero-player soak across all scenarios with seed coverage beyond the gate
(default base seed 1000), to flush rare-path crashes. It is resumable
(re-run the same command; it continues from the JSONL) and supports a
`--max-seconds` budget and `--invariants` for full per-card checking.
Last run: 1000 crash-focused + 300 invariant-on games, zero crashes and
zero invariant violations.

`python -m lod_ai.tools.gather_decline_audit --seeds 1-20` verifies that
every Indian free-Gather *decline* (planner returns "no legal plan") is
grounded in the actual space state -- it independently reconstructs why
no space was legal (West Indies / support outside the eligible band /
no War Party or Village in or adjacent) and fails if any decline
coincides with a legal target the planner missed. The ~25 declines per
60 games are all genuine: most are provinces pinned at full support
(+/-2), where Gather is illegal.

### Human-mode QA harness

`python -m lod_ai.tools.human_qa --seeds 1-5` drives the *real* interactive
game loop (`interactive_cli._game_loop`) with a scripted input provider in
place of a person, across five seatings (incl. 1775 pre-Treaty French and
a 1776 three-human game). It deliberately exercises the documented blind
spots: meta-commands injected mid-wizard, **undo during Winter Quarters**,
the Brilliant Stroke declaration prompt from a human seat, French
pre-Treaty flow, and a full save -> reload-into-fresh-engine -> resume
cycle. Any crash or post-game invariant violation writes a crash-repro
dump (scenario + seed + seated factions + the scripted input log) and the
run fails. Last full cycle: 25 games x (full play + resume), zero crashes,
zero invariant violations, every game exercising a WQ undo.

This harness surfaced (and the same commit fixed) a contract gap:
`pause_for_player` only honored `status`, although the CLI advertises
meta-commands "at every input prompt." It now routes through the meta
handler, so save/status/history/victory/deck/help/undo/quit all work at
pause points -- which is what makes the **undo-during-Winter-Quarters**
path reachable (the WQ branch of `_game_loop` already had an
`except UndoException` restart that nothing could previously trigger,
because Winter Quarters resolution is fully automatic and never prompts a
human).

Per-faction win rates have stabilized at approximately:

| Scenario | PAT | BRI | FRE | IND |
|----------|-----|-----|-----|-----|
| 1775     | ~75%| ~15%| ~0% | ~10%|
| **1776** | **~98%** | ~2% | ~0% | ~0% |
| 1778     | ~40%| ~5% | ~50%| ~5% |

**1776 is heavily Patriot-favored by design.**  Full investigation in
`audit_report.md` (Session 17, May 2026): the four bot flowcharts as
published produce this outcome; it is not a coding bug.  Patriots
start with Opp − Sup = 2 (only 8 points from Margin 1 victory), 4
Persuasion-eligible spaces, and 5 Rabble-Rousing-eligible spaces, and
the British bot's published flowchart prescribes a conservative
opening that does not directly counter the Patriot RR engine.  Do
**not** rebalance by altering bot priorities — that would deviate from
the references.

### Remaining open items

Small, surgical work — none crash-class.  Most of the originally-
listed items have been closed; see `audit_report.md` Sessions 17-19.

- Ch 8 traceability backlog T2-T11 lives in TRACEABILITY.md; the
  overall plan is ROADMAP.md. (T1 Commands-Not-Limited was FIXED in
  Session 23 along with three latent §3.2.1 Muster bugs it exposed;
  33/60 baseline winners flipped, see audit_report.md.)

- Middle/late-war event handlers still contain alphabetical / dict-order
  subset space picks (`late_war.py` ~lines 155/167/205/795/1005; audit
  `middle_war.py` too). Route them through
  `shared.select_support_shift_spaces` (§8.3.6) or
  `shared.pick_random_spaces` (§8.2) per card text, as Session 21 did
  for the early-war set. Each needs its card reference read first
  (agents.md card workflow).

- `_battle`'s B12 selection now uses the resolver's *exact* Force Level
  (§3.6.2-3.6.3) and Loss-Level modifiers (§3.6.5-3.6.6) via the shared
  `battle.bot_battle_scores` / `battle.force_level` helpers, instead of a
  hand-rolled approximation (which had real modifier bugs, e.g. an
  always-true "half defending cubes are Regulars" test). The
  approximation is gone -- the compromise is closed. A sandboxed
  benchmark (`lod_ai.tools.battle_benchmark`) measures the residual
  outcome: faithful B12 is slightly *more* aggressive than the old
  approximation, and ~13% of selected British battles are likely losses.
  That residual is INHERENT to B12 -- its Force-Level comparison ignores
  the Loss-Value asymmetry (Regulars count as two losses) and dice
  variance -- and cannot be reduced without deviating from the reference.
  Balance stayed within band (3 winner shifts/60 games; baseline
  refreshed). B12's "Use Common Cause to increase British Force Level"
  is also folded into the selection: each candidate space is scored with
  the War Parties it would commit as Tory-equivalents (B13: Active first,
  up to Regulars, keeping the last Underground), and `_try_common_cause`
  now loans exactly that capped amount so prediction and resolution agree
  (previously it over-loaned all War Parties). Zero balance drift.

- Patriot (P4) and French (F16) Battle selection now use the same shared
  `battle.bot_battle_scores` / `battle.force_level` helpers as the British
  B12 selection, so all three attacker factions score "Rebel/Royalist
  Force Level + modifiers exceeds ..." with the resolver's exact maths.
  This removed real bugs in the old hand-rolled versions -- the French
  selection had added Patriot *Forts* to the *attacking* Force Level
  (§3.6.2: Forts count only when Defending) and used un-halved Militia /
  War Parties with no §3.6.5-6 modifiers; the Patriot version counted War
  Parties as Royalist "cubes" for the half-Regulars test. Ally cubes are
  included only when the ally is paid (§8.5.1 French Resources > 0 for a
  Patriot Battle; Patriot Resources > 0 for a French Battle). Balance
  within band (6 winner shifts/60, mostly 1778; baseline refreshed).

- Common-Cause Indian-leader-follow: FIXED. When a British March uses
  Common Cause, War Parties move (as Tory-equivalents) out of an Indian
  Leader's space, and the Leader now follows the largest group per OPS.
  The leader-follow logic was extracted to a module-level
  `indians.follow_indian_leaders_after_move(state)` (operating purely on
  post-move board state) and is invoked from `BritishBot._march` whenever
  Common Cause was used, in addition to the Indian bot's own commands.

- Phase 4 human-player CLI: the crash/invariant-class items (load/save
  round-trip, undo during Winter Quarters, multi-human cross-faction
  games, French pre-Treaty flow, Brilliant Stroke declaration from a
  human seat, meta-commands mid-wizard) are now covered by the
  `lod_ai.tools.human_qa` harness and pass cleanly. What remains is
  subjective UX/feel polish, which is best judged in live playtests
  rather than scripted runs.

### Items previously listed here that have been closed

(See `audit_report.md` for the commit and PR per item.)

- `bot_indian_trade` and `bot_leader_movement` wired into the engine
  (commit 9155880).
- Garrison-induced British leader movement + Indian leader movement
  after March/Scout/Gather/Raid wired (commit e6036ef).
- Four silently-broken leader capabilities fixed: Clinton, Cornplanter,
  Gage, Rochambeau (commit 83aff7a).
- CLI RNG seed menu restored + setup order matches README
  (commit f66d490).
- Dead `apply_leader_modifiers` hook system deleted (commit db4c17e).
- `_is_howe` / `_try_naval_pressure` multi-leader presence checks
  fixed, `_is_gage` / `_british_leader` deleted, `_march` extra
  Common-Cause post-March fallback removed (commit 9bf4f99).
- "Battle-induced leader movement" was an over-cautious item in the
  May 2026 audit: Battle does not move faction units between spaces
  (only to Casualties / Available), so the leader-movement rule
  ("follow largest group of own units that moves from") has no
  trigger.  Verified by inspecting `battle.py` — no `move_piece`
  calls, only `remove_piece` to `casualties`/`available`.

### What recent sessions established

(For context to future Claude sessions or contributors — do not redo
these investigations from scratch.)

- Non-player Event space selection now follows §8.3.5/§8.3.6/§8.2
  (Session 21, audit_report.md): support-shift events (cards 2, 10, 25,
  41, 46) select by pop-weighted gain via
  `shared.select_support_shift_spaces`, placement/removal events (2u, 6,
  32u, 43u, 46u, 84) follow §8.3.5 priorities, and §8.2 Random Spaces —
  now seeded from ``state["rng"]`` via
  `random_spaces.pick_random_spaces` — is used ONLY to break equal-
  priority ties. `base_bot._is_ineffective_event` implements §8.3.3's
  net-shift-favors-the-enemy clause. The "candidate set" question from
  the July 2026 handoff was resolved from the manual (QUESTIONS.md Q14)
  — it was not actually ambiguous. Alphabetical subset picks remain in
  `late_war.py` (~lines 155/167/205/795/1005) and possibly
  `middle_war.py`; they need the same card-by-card §8.3.5 audit (open
  item below).

- `shared.adjust_fni` now enforces the §1.9 / §4.5.3 invariant "FNI may
  never exceed the number of Blockades that are Available" on *raises*:
  card effects that raise FNI (e.g. card 40 shaded, "FNI to 3") are capped
  at `naval.fni_ceiling(state)` = West Indies pool + on-City Blockades
  (markers in Unavailable don't count), so a card can no longer push FNI
  above the markers in play. The French Naval Pressure SA already enforced
  this; the card path didn't. Lowering FNI is unaffected; zero balance
  drift. `adjust_fni` now has a single implementation in `lod_ai.util.naval`
  (its natural home, beside `fni_ceiling`); `cards.effects.shared`
  re-exports it, so every caller -- cards, the Naval Pressure SA, and
  year-end -- routes through the one ceiling-aware function. (The previous
  duplicate, non-enforcing `shared.adjust_fni` is gone.)
- `setup_state.py` now reads cumulative casualty counters from the
  scenario JSON (`british_casualties`, `patriot_casualties`).
  Previously hardcoded to 0, which was wrong for 1776 (CBC=1, CRC=3)
  and 1778 (CBC=10).
- The §6.2.2 West Indies year-end battle is correctly called with
  `free=True` in `_supply_phase`.  Previously this crashed any game
  where French reached year-end with 0 Resources.
- `PatriotBot._try_persuasion` gates on a per-turn flag.  Persuasion
  fires from several flowchart nodes and was previously allowed to
  fire 2–4× per turn, violating Manual §4.1's one-SA-per-Command
  rule.
- `BritishBot._muster` correctly respects the Limited Command 1-space
  cap including the RL/Fort selection step.  Previously could append
  a 2nd space and get rejected as `limited_wrong_count`.
- `BritishBot._march`'s Phase-3 "March in place to Activate Militia"
  now pays §3.2.3 cost and tags the command properly.  Previously
  produced `no_affected_spaces` illegal-action rejections when it was
  the only viable action.
- `PatriotBot._execute_battle`'s Win-the-Day callback skips the free
  Rally when the battle space is a Reserve / West Indies (Rally is
  illegal there per §1.4.2 / §3.3.1).  Previously crashed with
  `ValueError: Cannot Rally in <space>`.

## Critical Rules

### Never Guess
If the Reference Documents are ambiguous, contradictory, or silent on a question:
1. **STOP** working on that specific issue
2. Document the question in `QUESTIONS.md` (create if missing), including:
   - What you were trying to implement
   - What the reference says (quote it)
   - What's ambiguous or contradictory
   - What options you see
3. Move on to other work
4. Do **not** implement a "best guess" — wait for the user to answer

### Rules-Accurate Over Simple
When faced with a choice between a simpler implementation and one that faithfully follows the rules, **always choose rules-accurate**. The flowcharts are complex on purpose. Do not simplify tie-breaking logic, do not skip edge cases, do not collapse decision branches that the flowchart keeps separate.

### No Outside References
- Do NOT consult BoardGameGeek.com
- Do NOT reference other GMT games or other COIN series titles
- Do NOT do historical research
- Do NOT consult any GitHub repository other than this one
- The Reference Documents folder is the complete universe of source material

### Testing
- Run `pytest -q` before every commit
- All tests must pass
- When fixing a bug, add a test that would have caught it
- When implementing a flowchart branch, add a test that exercises it
- Tests should verify behavior against the Reference Documents, not against assumed behavior

### Commit Discipline
- One logical change per commit
- Commit message should reference what was changed and why (e.g., "Fix card 24: unshaded was inverted vs reference")
- Never commit with failing tests

---

## Conventions

- **Python 3.10+** assumed
- **Imports:** Always import constants from `rules_consts.py` rather than using string literals
- **State:** Game state is a dictionary passed through functions. Do not use global state.
- **Map queries:** Use helpers in `lod_ai/map/` and `lod_ai/board/` — do not access space dictionaries directly with string keys like `sp.get("type")` or `sp.get("British_Control")`
- **Piece operations:** Use `lod_ai/board/pieces.py` for placement/removal — do not manipulate piece lists directly
- **Caps:** Use `lod_ai/util/caps.py` for piece cap enforcement
- **Control:** Use `lod_ai/board/control.py` for control calculations

---

## How to Run

```bash
# Setup
python -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt

# Run the game
python -m lod_ai

# Run tests
pytest -q
```

---

## For the Maintainer

This project was originally built with ChatGPT assistance, which
introduced systematic correctness issues.  As of May 2026 those have
all been worked through (Phases 1-3 complete; see Project Status
above).  The current bar for new work is the same as it has always
been: rules-accurate, faithful to the Reference Documents, no
guessing.

When in doubt, read the Reference Documents again.  Then read them
one more time.  When the Reference Documents are silent or
contradictory, add the question to `QUESTIONS.md` and stop — do not
implement a "reasonable interpretation".

---
> Source: [FeeSimple1/lod-bot](https://github.com/FeeSimple1/lod-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
