## excalibur

> This file is the starter contract for building a new Excalibur-like system from this repository.

# AGENTS.md

This file is the starter contract for building a new Excalibur-like system from this repository.

Read [`INVOCATION.md`](./INVOCATION.md) first. Use this file as the operating standard while the system is being instantiated and extended.

## Core Shape

Keep the system simple and explicit:

- one primary spirit at first: `lapis`
- one top-level `spirits/` tree for spirit identity, prompts, rituals, and memories
- one top-level `artifacts/` tree for durable outputs
- one top-level `questbook/` tree for obligations and continuity
- one internal `grimoire/` for spellbooks, runtime code stubs, and summoner docs
- one universal always-open spellbook in `grimoire/spellbooks/adept/`
- optional spellbooks under `grimoire/spellbooks/<book>/`
- one cast per folder at `grimoire/spellbooks/<book>/<cast>/spell.md`
- one shared charge ledger at `chargebook.md`
- one top-level `vessel/` tree for machine-local operation

Do not flatten the hierarchy. The nested spellbook layout is part of the design.

## Primary Spirit

Use `lapis` as the primary orchestrator spirit unless the summoner explicitly wants a different starting shape.

The primary spirit should be:

- responsive in the foreground
- honest about state changes
- explicit before and after casts
- able to hand long work off to workings

Keep the primary spirit useful without making it overloaded.

## Foreground Thread

Every real system built from this scaffold needs an explicit foreground thread.

That means:

- the summoner has a concrete way to talk to the primary orchestrator spirit
- the transport is named and configured, not implied
- every inbound and outbound foreground turn is mirrored into `vessel/state/<spirit>/conversations/<local-date>.jsonl`
- rituals and workings append checkpoints and durable outcomes into that same daily thread

The daily thread is not optional bookkeeping. It is how foreground conversation, background work, and ritual continuity stay joined.

## Behavioral Laws

These should hold across the system:

- answer the summoner before deeper work begins
- acknowledge before a cast, emanation, working, or file change
- report what actually happened after it lands
- perform real lookups when a request depends on external state
- keep the foreground thread responsive
- move long or cast-heavy work into workings
- never claim a write, send, search, or publish happened unless it actually succeeded
- correct implied but false state changes plainly
- treat the shared daily thread as the canonical ledger
- allow a spirit to maintain its own `cornerstone.md` and `memories/` when the design calls for it
- treat ritual files as read-only during execution

## Spellbook Design

Keep the spellbook surface disciplined:

- `adept` is always open
- all other capability families are optional spellbooks
- a spirit advertises optional access through `available_spellbooks` in `identity.md`
- rituals may widen through `additional_spellbooks` and preload through `open_spellbooks`
- each cast lives in its own folder beside its manifest

Design spellbooks by capability family, not by convenience.

Some spellbooks are especially foundational:

- `adept` carries orchestration, messaging, emanations, emergency halt, and ritual charge recovery
- `artifact` is the durable output layer for reports, notes, drafts, guides, exports, and other work meant to outlive the current thread
- `working` is where long or cast-heavy work should go
- `portal` is the always-on surface primitive and should stay general rather than being tied to one specific device

The artifact surface should also have an honest filesystem shape. A new system does not need every project-specific folder up front, but the shared generic roots should usually include:

- `artifacts/archive/`
- `artifacts/library/`
- `artifacts/media/`
- `artifacts/network/`
- `artifacts/notes/`
- `artifacts/reports/`
- `artifacts/research/`

`essays/`, `nightly/`, `dreamtime/`, and `tests/` are also common and should be added when the system actually uses them.

## Ritual Design

Rituals should be:

- readable
- narrow in purpose
- quiet by default
- durable in their effects

Use rituals for continuity, consolidation, refresh work, and recurring stewardship. Keep them explicit in markdown.

## Charge Design

Charge is the run-level continuation primitive.

Use it to make expansion visible:

- local management casts can often cost `0`
- acquisition, generation, or widening paths can cost more
- emanations may have base cast cost `0` while still allocating charge from the source run

Emanations are the clearest example of the model. They show that widening work is not the same thing as being free. A spirit may cast an emanation at base cost `0`, but the source run still spends part of its own budget to buy that extra pass.

The charge model should stay easy to understand at a glance.

Keep `chargebook.md` as the single tuning surface for cast costs. If a ritual is over-branching or spending too freely, adjust the relevant cast cost there instead of scattering cost logic through ritual prose.

## Memory Design

Keep memory layered:

- the daily thread is the live working ledger
- `memories/long-term.md` is compact top-of-head durable memory
- `memories/window/` is the rolling recent-memory window
- `memories/archive/` is for lower-signal memory residue that is still worth preserving
- the rest of `memories/` is durable searchable storage that does not need to stay prompt-attached

The correct flow is:

- foreground conversation lands in the daily thread first
- memory rituals distill that ledger into the rolling window and long-term memory
- artifacts hold durable outputs that should outlive the thread

Do not let prompt context bloat into an undifferentiated pile.

## Storage Design

Keep storage roles distinct:

- `artifacts/` for the main shared work product surface between spirit and summoner
- `questbook/` for the main shared obligation and continuity surface between spirit and summoner
- `grimoire/portals/` for always-on surfaces
- `spirits/<name>/cornerstone.md` and `spirits/<name>/memories/` for spirit-local self-maintenance
- `spirits/<name>/rituals/` for spirit-local ritual law that should stay stable during ordinary execution
- `vessel/state/` for machine-local runtime records, including daily conversation ledgers
- `vessel/backups/` for local repository mirrors and backup worktrees
- `vessel/venvs/` for isolated helper runtimes used by local utilities

`grimoire/` and `vessel/` are spirit-side internals.

`artifacts/` and `questbook/` are the main shared surfaces.

That separation is structural. Do not collapse everything into one generic content folder.

## Optional Surfaces

These are good optional widening paths when the summoner wants them:

- questbook
- portals
- network artifacts
- corpus retrieval
- media work
- celestial context routed through portals

Keep them available in the repository. Keep them optional in the initial build unless the summoner asks for them.

## Security and Cleanliness

Build secure defaults:

- fail closed where authority, transport, or exposure rules matter
- do not hardcode secrets into markdown or code
- do not mix private operational residue into the public structure
- keep human-editable manifests legible and reviewable

## Standard

A new summoner should be able to read this repository and quickly understand:

- where spirit identity lives
- where casts are defined
- how rituals work
- why charge exists
- where durable outputs belong

If that is not clear, the system is too muddy.

## Maintenance Rule

When the taxonomy, hierarchy, or operating laws change, update this file in the same turn.

---
> Source: [viemccoy/excalibur](https://github.com/viemccoy/excalibur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
