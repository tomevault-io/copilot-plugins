## oldworld-mptweaks

> Old World **XML-only mod that strips high-variance events from competitive multiplayer**.

# CLAUDE.md — owmptweaks

Old World **XML-only mod that strips high-variance events from competitive multiplayer**.
Read `README.md` first, then `WHATS-DIFFERENT.md` for the shipped list. This file holds the
grounding facts and the reasoning behind the shape.

## ► STATUS (2026-08-23) — v0.1 built, NOT yet GUI-verified

- ✅ 49 eventStory removals + 1 global + 1 event-option reweight, all generated from
  `tools/tweaks.py`, all verified against the shipped XML (`python3 tools/gen.py`).
- ✅ `./build.sh` deploys to `~/Library/Application Support/OldWorld/Mods/OwMpTweaks/`.
- ⏳ **Never launched.** Nobody has confirmed the game loads it, the `-change` files apply, or
  that a removed event actually stops firing. That is the next step and Claude cannot do it.
- ⏳ No `mod/modpicture.png`. Only needed to upload to the Workshop (≥256×256, <1MB).
- ⏳ `modplatform` is `Local` and `workshopFileID` is 0. **After the first upload the game
  writes the Workshop identity back into the installed copy — copy it into `mod/ModInfo.xml`
  by hand, or the next `build.sh` (`rm -rf "$DEST"`) severs the link and re-uploading creates
  a duplicate item.** Same trap as `../owtraitmod`.

### Verifying in game

Enable, restart, start a game, then check the removals took. Cheapest check: `INFERTILE_PROB`
is a global, so if the mod loaded at all the XML pipeline worked. For the events, the honest
test is a long hot-seat game, which is slow — consider borrowing `../owtraitmod/test/` (it
merges `mod/Infos/*.xml` into a symlinked copy of the base XML tree and boots the real
GameCore headless) and asserting that a removed zType has `miWeight == 0` after Infos loads.
That would catch a `-change` file silently not applying, which is the failure mode most
likely to go unnoticed.

## Design decisions (settled — don't relitigate)

- **One mod, preferences baked in, no game-option toggles.** An earlier draft proposed adding
  `GAMEOPTION_MPTWEAKS_*` entries and gating each story with `aeGameOptionInvalid`. Dropped
  deliberately: it prevents option hell, and splitting into per-tweak mods is worse still
  because **every player in an MP lobby has to enable the identical set** — one checkbox
  scales, eighteen do not. If people feel strongly, revisit then, not now.
- **Stressed and Unpopular are neutralised with `iRemoveProb 100`, not blocked outright.**
  Settled 2026-08-25: **do not** switch to `iMinAge 999`. The trait landing for a
  single turn and then clearing is the accepted behaviour; the knock-on effects of a hard block
  are not worth it. Full reasoning in the `TRAITS` comment in `tools/tweaks.py`. Don't re-offer
  this.
- **Generated, not hand-written.** `tools/tweaks.py` is the only file to edit; `tools/gen.py`
  emits the three XML files *and* `WHATS-DIFFERENT.md` from it, so the docs cannot drift.
  `gen.py --check` verifies without writing — wire it into CI if this ever gets one.

## Engine facts this is built on (verified in the shipped source)

Source of truth: the shipped `Old World/Reference/` tree (Source + XML) in your Steam
install. **Never edit it** — Steam-synced. `tools/ow.py` locates it automatically, or set
`OW_REFERENCE`. Installed build **1.0.84365 (2026-08-05)**,
Steam buildid 24569266.

### Disabling an event

`Player.cacheEventPools()` → local `canEverDoEvent()` (PlayerEvent.cs:12788) rejects a story
when **`miWeight <= 0`**, before it can enter a trigger pool. Every XML-driven story reaches a
player through a pool, so `<iWeight>0</iWeight>` is a complete disable.

Levers considered and rejected:
- `iMinTurns 200` — what the shipped Barbarian/Carthage mods use. Works, but expires in a long
  game and does not remove the story from the pool.
- `aeGameOptionInvalid` — also honoured by `canEverDoEvent`, and the right lever *if* toggles
  ever come back. Needs a matching `gameOption-add.xml`.
- `Player.isValidEventStory` (PlayerEvent.cs:730, `protected virtual`) — the C# chokepoint, but
  it is **only reached from the pool-selection path** (called once, at PlayerEvent.cs:13031).
  `doEventStory()` does not call it, so it is not a superset of the weight check.

### Mod XML load order (Infos.cs:840–890)

base → **`-add`** (`ADD`, `ADD_ALWAYS`) → **`-change`** → **append** (`AppendLists = true`) →
`-change` again. Two facts that matter:

- The suffix is **`-change`**, not `-modify`. 237 shipped examples under `Reference/XML/Mods/`.
  (`spmorelikemp1` in the local Mods folder uses `council-modify.xml`, which is very likely a
  silent no-op — worth telling the author.)
- **A `-change` REPLACES list fields.** `Infos.readTypes` / `readPairList` call `.Clear()`
  unless `ctx.AppendLists`, which only the append pass sets. So `eventOption-change.xml` must
  restate all ten `aiEventOptionProb` pairs, not just the one it lowers. `gen.py` verifies our
  pair list against the shipped one in both directions so a patch adding an eleventh outcome
  fails the build instead of silently deleting it.
- Scalars are safe: `readInt` assigns whatever parses, so an explicit `0` really is 0.

### Infertility is not a trait

`Character.mbInfertile`, set by `Character.checkMakeInfertile(bool bInit)` (Character.cs:7608),
which has three branches: the `bInit` roll on `INFERTILE_PROB` (10) for anyone created after
`CHARACTER_INIT_TURNS` (20); a hard cutoff at `gender().miMaxFertile + FERTILE_AGE_MARGIN`;
and `INFERTILE_PERCENT_PER_TURN` (25) past `miMaxFertile`. `miMaxFertile` is 38 female /
70 male. Setting `INFERTILE_PROB` to 0 disables exactly the first branch and needs no DLL —
overriding the method would have been equivalent and heavier.

### Free technology

A bonus with `aeTechs` calls `makeTechAcquired(eTech, true, false)` directly
(PlayerBonus.cs:6047) — **prerequisites are not checked**. 67 bonuses carry it, reaching ~30
stories. Only `EVENTSTORY_GP_THE_MECHANISM` is disabled so far; see the OPEN note in
`tools/tweaks.py`.

### Distant raids

`GAMEOPTION_NO_DISTANT_RAIDS` already returns early in `City.doDistantRaidTurn()`
(City.cs:8432). It does **not** stop events whose option carries a `bDistantRaid` bonus. Four
such stories create a raid from nothing; ten more are *reactions*, fired via
`doEventTrigger(CITY_DISTANT_RAID_EVENTTRIGGER)` from inside `doDistantRaidTurn` — if none
fires the game calls `doDistantRaid()` anyway, so disabling those would give the raid with no
card. Only the four are removed.

### Calamities are Occurrences

Wrath of the Gods content. `OCCURRENCECLASS_CALAMITIES` in `occurrenceClass.xml`;
`Game.doOccurrences()` (Game.cs:8290) rolls each type's `miStartChance`, then
`findOccurrenceTargetTile` picks **one** tile. `canStartOccurrence` already refuses a second
occurrence of the same class while one is active.

## Tooling (`tools/`)

`ow.py` indexes the shipped XML — 5284 eventStory, 11779 eventOption, 4141 titles, plus a
bonus index and `story_bonuses()` which walks a story's options and sub-option probability
tables transitively. The others are thin CLIs over it:

| tool | use |
|---|---|
| `gen.py` | verify + generate. `--check` to verify only |
| `find.py <regex>` | stories by display title or zType |
| `deep.py <regex>` | stories whose own XML *or any linked option's* XML matches |
| `byprop.py <regex>` | stories reachable to a bonus whose XML matches, e.g. `'<bKillCharacter'` |
| `selfkill.py` | stories that can kill one of **your own** characters, resolved positionally |

`byprop.py` and `selfkill.py` rely on the fact that **`aeBonuses` is positional against the
story's `aeSubjects`** — index *i* in the bonus list applies to subject *i*, which is why the
XML is full of empty `<zValue/>` padding. `selfkill.py` uses that to tell "kills your leader"
from "kills theirs"; a naive grep cannot.

## Deferred to a DLL release

Numbers and hook points are in `DEFERRED` at the bottom of `tools/tweaks.py`. Short version:

- **Character death** — 61 stories can kill your own leader, 281 including relatives. Too many
  to blocklist safely; the systemic hook is `Player.canDoBonusSingle` (PlayerBonus.cs:453),
  which already special-cases `mbKillCharacter`.
- **Synced calamities** — the largest item. Needs a `Game` subclass.
- **Birth variance** — `Character.getGiveBirthPercent` gives 30% per turn to a first child,
  10% to a second, then 5/3/2. Independent Bernoulli, so time-to-first-child is mean 3.3 turns
  with sd 3.0. A deterministic accumulator keeps the mean and removes the spread.
- **World religion once per game per player** — the option system has no per-player one-shot;
  v0.1 only makes it rarer.

## Gotchas

- Several removals target **Behind the Throne** (`eventStory-btt.xml`) and **Wrath of the
  Gods** (`-wog.xml`) content. A `-change` naming a zType from a DLC the player does not own
  should be inert, but this is **unverified** — watch for load errors in
  `~/Library/Application Support/OldWorld/Logs/output.txt` on a base-game-only install.
- `EVENTSTORY_RISING_STAR` (base, GENERAL_KILL) is not the BTT `RISING_GENERAL` /
  `RISING_GOVERNOR` / `RISING_POWER` / `RISING_DEMANDS` family. Only the former is removed —
  **confirm which was meant.**
- Verify every mechanic against the shipped XML/source before asserting it. In the parent
  project several confident claims in early drafts turned out to be wrong until checked.

---
> Source: [alcaras/OldWorld-MPTweaks](https://github.com/alcaras/OldWorld-MPTweaks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
