## bwa-click

> **This repo does not host active development.** It exists so GitHub users

# BWA-Click — Read this before doing anything

## You are in a release-destination repo, not the dev surface

**This repo does not host active development.** It exists so GitHub users
can clone, browse, and reason about BWA-Click on its own without
seeing the rest of the BWA monorepo.

All real work on BWA-Click happens inside the BWA monorepo at:

```
~/Developer/bwa-fx-ecosystem/plugins/click/
    dsp/           ← authoritative DSP implementation (THE source of truth)
    ui/            ← UI surface (Metal-rendered)
    plugin/        ← VST3 wrapper → builds BWA-Click.vst3
    manifest.json  ← name, controls, license tier, ecosystem benefits
```

If you've been asked to "work on BWA-Click," **navigate to the
monorepo path above and work there**. Do not edit files in this repo
directly. Releases land here via `git subtree push` from the monorepo —
detail in [`BWA-Architecture/docs/PLAN-2026-Q2.md §8.1`](https://github.com/vmcguire/BWA-Architecture/blob/main/docs/PLAN-2026-Q2.md#81-decision-1--dual-surface-single-dsp-resolved).

If this repo has commits the monorepo doesn't, that's a bug to reconcile,
not a feature.

## Why this design

Every BWA plugin ships on two surfaces:

1. **Standalone VST3** (this repo's release artifact) — `BWA-Click.vst3`
2. **Embedded cell inside BWA-Mix** — same DSP, rendered as a cell

Both surfaces link against the same `dsp/` module. Single-source DSP is locked
in [`ECOSYSTEM.md §6 Principle 3`](https://github.com/vmcguire/BWA-Architecture/blob/main/ECOSYSTEM.md):
*"One DSP implementation. BWA-FX is the canonical source of every effect. No
plugin reimplements compression or EQ."*

Splitting development across two repos (this one + the monorepo) would
violate that principle — so it doesn't happen. The monorepo is the only dev
surface.

## What this plugin is

- **Category:** B (Generative / Groove)
- **Tech tree tier:** Tier 5
- **Role:** Plays shared/groove/ as audible click + emits MIDI sync markers.
- **Pain attacked:** If the human's groove is the canonical timing, the click has to play THAT — not the metronome.
- **Scope verdict:** project-global (one click per project)
- **Band tier behavior:** Not tier-gated.

Full row in
[`MARKET-PAIN-AND-ATTACKS.md`](https://github.com/vmcguire/BWA-Architecture/blob/main/docs/MARKET-PAIN-AND-ATTACKS.md)
— search for `BWA-Click`.

## Required reading before writing code (for LLMs and humans alike)

In order:

1. **[`BWA-Architecture/docs/LLM-ONBOARDING.md`](https://github.com/vmcguire/BWA-Architecture/blob/main/docs/LLM-ONBOARDING.md)**
   — the meta-thesis (emotion-first, natural-structure-first), the
   6-category taxonomy, locked decisions, anti-patterns. **Required.**
2. **[`BWA-Architecture/docs/USER-STORY-MAP.md`](https://github.com/vmcguire/BWA-Architecture/blob/main/docs/USER-STORY-MAP.md)**
   — where BWA-Click sits in the bedroom-producer journey
   (Stage 5 — ARRANGE).
3. **[`BWA-Architecture/docs/PLAN-2026-Q2.md`](https://github.com/vmcguire/BWA-Architecture/blob/main/docs/PLAN-2026-Q2.md)**
   — current build plan + repo inventory. Find BWA-Click in §9 and
   §4 to see what phase it's in.
4. **[`BWA-Architecture/docs/MARKET-PAIN-AND-ATTACKS.md`](https://github.com/vmcguire/BWA-Architecture/blob/main/docs/MARKET-PAIN-AND-ATTACKS.md)**
   — the row for BWA-Click. Internalize the pain + the attack before
   writing a single line of DSP.
5. **[`BWA-Architecture/ECOSYSTEM.md`](https://github.com/vmcguire/BWA-Architecture/blob/main/ECOSYSTEM.md)**
   — product map + architecture invariants.
6. **[`BWA-Architecture/UX.md`](https://github.com/vmcguire/BWA-Architecture/blob/main/UX.md)**
   — UX patterns shared across plugins (parameter model, X-ray view,
   scope selector, streaming presets).

## Common LLM anti-patterns when working on this plugin

- ❌ Committing to this repo. Go to the monorepo (`plugins/click/`).
- ❌ Reimplementing DSP that already exists in `shared/dsp/` or another
  plugin's `dsp/`. The principle is **one implementation, used everywhere**.
- ❌ Skipping the LLM-ONBOARDING.md read. Plugins that don't fit the
  natural-structure-first thesis don't ship; you need to know what fits.
- ❌ Designing for "pro engineers." The audience is bedroom producers.
- ❌ Adding new shared/* modules without checking if the capability folds
  into PluginKit, Engine, or MySounds. See LLM-ONBOARDING.md "Tech Tree
  Methodology Rule 3."
- ❌ Skipping the BWA-Click row in MARKET-PAIN-AND-ATTACKS.md. If the
  proposed work doesn't advance the BWA Attack for this plugin, push back.

## What "done" looks like

A `BWA-Click.vst3` that:

1. Implements the BWA Attack from `MARKET-PAIN-AND-ATTACKS.md` (not a
   me-too of competitor X).
2. Reuses `shared/dsp/` for any primitive (FIR, FFT, envelope follower,
   biquad) rather than reimplementing.
3. Auto-feeds the universal 24-band Goertzel analyzer on every block via
   `BaseProcessor` — this is non-negotiable architecture, baked into
   PluginKit.
4. Renders the **24-band X-ray view** for its operation (the BWA visual
   signature).
8. Passes all `shared/` contract tests.

## Release ritual (when shipping a version)

From inside the monorepo (`~/Developer/bwa-fx-ecosystem/`), after the
release commit on `main`:

```bash
git subtree push --prefix=plugins/click \
  git@github.com:vmcguire/BWA-Click.git main
```

Then on GitHub:

1. Create a Release from `main` in this repo.
2. Attach the notarized `BWA-Click.vst3` zip.
3. Reference the monorepo commit SHA in the release notes.

---

_End of LLM onboarding for BWA-Click._

---
> Source: [vmcguire/BWA-Click](https://github.com/vmcguire/BWA-Click) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
