## ada-compliance-os

> <!-- ledger-mandatory:v2 — MUST STAY AT THE TOP OF THIS FILE. Do not move it down. -->

<!-- ledger-mandatory:v2 — MUST STAY AT THE TOP OF THIS FILE. Do not move it down. -->
> # 🛑🧾 STOP — EVERY AGENT MUST READ THE LEDGER, AND MUST RECORD EVERY ERROR
> ## This is not optional, and it is the first thing you do.
>
> **Before you write anything in this repo: read `LEDGER.md`.** If this repo has none, read
> **`coreflux-ai-os/brain/LEDGER.md`** — the cross-system one. It is a list of mistakes that were
> already paid for once. **An agent that skips it makes them again.**
>
> **⚠️ WHY THIS BANNER IS AT THE TOP AND NOT FURTHER DOWN.** During the client-site factory build,
> the same mistakes kept recurring for weeks. The ledgers existed the whole time — but they were
> **buried, unreferenced, and optional**, so agents never opened them and the system never learned.
> **A ledger nobody is obliged to read is decoration.** That is why this sits above everything else.
>
> ### 1. READ — before your first edit
> The ledger tells you which approaches are already known to fail here. Reading it costs a minute;
> repeating an entry costs the whole task, and sometimes live work.
>
> ### 2. RECORD — every error, the moment it happens. No exceptions.
> **Log it even if:** you already fixed it · it seemed small · nobody noticed · it was your own
> mistake · it was embarrassing · it was "just" a typo in a script. **Especially then.**
> An error that isn't written down is an error the machine makes again on the next run.
>
> - **THE LAW: "A fix is not a fix. A GUARD is a fix."** An entry closes ONLY when a permanent
>   automated check makes that failure impossible to repeat *silently*.
> - `FIXED + GUARDED` = closed · `FIXED + RULE` = **NOT closed** (a rule is a suggestion under
>   pressure) · `OPEN`. Never write a status you haven't earned.
> - **One entry per ROOT CAUSE, never per symptom.** If it reads like a symptom, dig again.
> - **Any CRITICAL, or anything seen 3 times → a full incident report** (background · condition ·
>   5-whys · countermeasure · verification · follow-up). That is the **andon cord**: the line stops
>   and explains itself, or the system never compounds.
> - **Never delete an entry, never squash the file, never gitignore it.**
>
> ### 🛑 The four failures that have cost the most. Do not repeat them.
> 1. **`git add -A`** — in a script or by hand. **Stage explicit paths.** (L-001/L-002 — 3× in one day.)
> 2. **Editing by character offset between two landmarks** — it slices through markup. Operate on
>    whole elements; assert structural balance before *and* after. (L-003)
> 3. **Trusting a detector that was never negative-tested** — inject a violation, watch it fail,
>    restore. **Fix the detector, never loosen the check.** (L-004)
> 4. **`git pull` in a loop** — check `git status --porcelain` first. `--autostash` is a deferral of
>    risk, not a safety net; it broke a live repo. (L-007)
>
> **Every error logged makes the next run better. That is the entire point.**
<!-- /ledger-mandatory:v2 -->

<!-- build-correctly:v1 — MUST STAY AT THE TOP. Depth lives in the skills; this is the part that must NOT depend on a skill loading. -->
> # 🏭 HOW WE BUILD — the four layers. Non-negotiable, every build.
>
> **1. PUT EVERY RULE IN THE STRONGEST TIER IT CAN LIVE IN.** Move rules *up* this ladder, never down:
> **HOOK** (cannot be violated, needs no reading) → **CLAUDE.md** (always read) → **SKILL**
> (description loaded, body on trigger) → **BRAIN DOC** (retrieved when searched).
> **The sorting question:** *could an agent violate this BEFORE having any reason to look it up?*
> Yes → hook or CLAUDE.md. No → a skill.
> 🛑 **The skill tier is an accelerator, not a guarantee.** Proven here 2026-08-06: a skill whose
> description never reaches the model is invisible, and *nothing reports it*. Anything that must be
> certain belongs above this line, not below it.
>
> **2. NAME THE SPACE BEFORE YOU BUILD THE STEP.** **LATENT** (taste, judgement, routing) → a markdown
> skill. **DETERMINISTIC** (numbers, money, dates, SQL) → a script + a cited table. Confusing the two
> causes most agent failures. 🛑 **RULE #0 — never invent a number or a formula — is the special case.**
>
> **3. WHERE THE CHECK GOES — and whether it is checking anything.**
> **① QUALITY gate BEFORE the tests** — tests cement behaviour; if the behaviour is mediocre they
> cement mediocrity. Prove it is good first. **② STATION gate** — fail at the step that MAKES the
> defect, not at the end (*jidoka*; TPS is **not** stage-gate). **③ MERGE gate** — a change that
> doesn't improve the cases doesn't ship. **④ UPGRADE gate** — re-run on every model upgrade; when
> ablation says the model no longer needs a skill, retire the skill and **keep the eval**.
> 🛑 **The three ways a gate betrays you:** it goes **blind** (keyed to a string the work itself
> changes — rename anything, re-verify every gate watching it) · you take it **green by loosening it**
> (**fix the detector, never the check**) · it was **never negative-tested** (*a gate you have never
> seen fail is not known to work* — inject a violation, watch it fail, restore).
> **Cheap before expensive:** static/regex first; model-judged only where regex cannot reach.
>
> **4. EVERY ERROR GETS LOGGED, THE MOMENT IT HAPPENS** — including small, already-fixed, unnoticed,
> your own, embarrassing. **"A fix is not a fix. A GUARD is a fix."** `FIXED + RULE` does **not** close
> an entry. One entry per **root cause**. Any CRITICAL or anything seen 3× → a full incident report.
>
> 📚 Depth: skills `build-ai-native` · `build-a-skill` · `brain-routing` · `skillify` · `skill-creator`.
> Ledger: `coreflux-ai-os/brain/LEDGER.md`.
<!-- /build-correctly:v1 -->

# ADA Compliance OS — agent operating rules

> **This file is the contract. Read it fully before touching anything in this repo.**

---

## 🛑 RULE 0 — ALWAYS READ THE LATEST VERSION OF THIS REPO. NEVER A CACHED ONE.

**Before any work in this repo — reading, planning, answering a question, writing code — you MUST:**

```bash
cd "C:\Users\knigh\Desktop\ADA-Compliance-OS"
git pull --rebase
```

Then read, in this order:

1. **`CLAUDE.md`** (this file) — the rules
2. **`MARKET-INTELLIGENCE.md`** — the legal + litigation data. Never state a market fact that isn't in here.
3. **`BUSINESS-MODEL.md`** — the offer, pricing, and monetization. Never invent a price.
4. **`IMPLEMENTATION-PLAN.md`** — what is being built, in what order, and what is deliberately NOT being built yet.
5. **`ERROR-LEDGER.md`** — every known bug and failure. Read before debugging anything.

**This applies to every agent, subagent, and skill invocation — no exceptions.**

Why this rule exists: this project was designed in March 2026, sat dormant, and was re-planned from scratch on 2026-07-29 with new legal research that **changed the product**. An agent working from the March README will give advice that is months out of date and commercially wrong. If you are answering from memory, a summary, or a prior conversation instead of the current files on disk, **you are wrong by default.**

**Corollaries:**

- ❌ Do NOT propose a strategy, price, target vertical, or build order that contradicts these files without saying explicitly that you are contradicting them and why.
- ❌ Do NOT re-derive decisions already recorded here. If it's written down, it's decided. Argue against it if you have evidence, but don't silently re-litigate it.
- ✅ Push every improvement back. If you learn something, write it into the right file and commit. This repo is the memory.
- ✅ If you find these files contradict the code, the **code is the truth about what exists** and the docs are the truth about what is *intended*. Fix the mismatch and note it.

---

## 🛑 RULE 1 — LEGAL AND MARKETING CLAIMS: HARD LINES

These are not style preferences. Crossing them creates legal exposure for the operator.

**NEVER claim, in any deliverable, email, call script, page, or report:**

- ❌ "compliant" / "ADA compliant" / "WCAG compliant"
- ❌ "certified" / "conformant" / "guaranteed"
- ❌ "lawsuit-proof" / "protected from litigation"
- ❌ "verified accessible" / "usable by blind customers"

**Why:** the FTC issued a **$1,000,000 consent order against accessiBe** (final order 2025-04-22, Matter No. 2223156) for exactly these claims about an automated product. The order bars representing that an automated product "can make any website WCAG-compliant or can ensure continued WCAG compliance over time" absent supporting evidence. That theory applies to the whole field, not just accessiBe. Each future violation carries civil penalties up to $51,744.

**NEVER, in outreach:**

- ❌ Imply you are a lawyer or affiliated with any plaintiff, firm, or agency
- ❌ Suggest a lawsuit or demand letter is pending, filed, or imminent when it isn't
- ❌ Use manufactured urgency ("we've been notified about your site…")
- ❌ Claim plaintiffs "target small businesses because they can't defend themselves" — **the data says the opposite** (see MARKET-INTELLIGENCE.md § Trends: under-$25M defendant share is *falling*, 73% → 67% → 64%)

**ALWAYS use instead:**

- ✅ "Documented, dated evidence of remediation"
- ✅ "Manually tested with NVDA screen reader and keyboard-only navigation on [DATE]"
- ✅ "64% of these suits hit businesses your size" (true, sourced)
- ✅ "A blind customer can't read your menu / can't complete this booking" (observable, verifiable, specific)

**The product is EVIDENCE, not compliance.** That distinction is the entire legal and commercial position. See MARKET-INTELLIGENCE.md § The Jones v. Moscot standard.

---

## 🛑 RULE 2 — EVIDENCE LABELS ARE MANDATORY

Every market fact in this repo carries a label. Preserve them. Never strip them, never upgrade one without a primary source.

| Label | Meaning |
|---|---|
| ⬛ **CONFIRMED** | Verified against a primary source (court, legislature, agency, FTC, Federal Register) |
| ◼ **REPORTED** | Law-firm tracker, trade press, or vendor blog only |
| ◻ **FLAGGED** | Could not verify. **Do not use client-facing.** |

**Anything ◻ FLAGGED must not appear in a client deliverable, call script, or marketing asset until it is verified.**

---

## 🛑 RULE 3 — DO NOT BUILD AHEAD OF REVENUE

`IMPLEMENTATION-PLAN.md` has a **BUILD NOW** list and a **DO NOT BUILD YET** list. The second list is load-bearing.

The first 5 clients get delivered **by hand** in Claude Code. No remediation engine, no fix library, no Supabase migration, no monitoring infrastructure until clients are paying for it. An agent that proposes building the engine before 5 paying clients exist is working against the plan.

---

## 🛑 RULE 4 — OPERATOR IDENTITY

- Email everywhere: **corefluxsystems@gmail.com**
- Git commit author: **corefluxsystems-os@users.noreply.github.com**
- NEVER use `ailinksystems@gmail.com` — the injected userEmail is stale

---

## 🛑 RULE 5 — WINDOWS / MODAL GOTCHAS

- `export PYTHONIOENCODING=utf-8` before any `modal` CLI command, or it crashes on its own checkmark glyph (`'charmap' codec can't encode character '\u2713'`)
- `modal-pipeline/deploy.py` is a wrapper that handles this for deploys — prefer it
- Cron is currently **commented out** in `nightly_pipeline.py`. That is deliberate, not a bug. Re-enable only when the outreach layer exists.

---

## What this system is

An AI-native machine that finds small businesses whose websites fail machine-detectable WCAG checks, documents those failures as dated third-party evidence, and sells remediation.

**It is not a compliance product. It is an evidence product with a remediation service attached.**

Detection half: **built and proven** (451 real businesses scanned across 10 unattended nights in March 2026, 77% violation rate).
Delivery half: **not built.** See IMPLEMENTATION-PLAN.md.

---

## Doctrine inherited from the operator's global rules

- **Service-as-Software:** build the machine once; don't perform the service manually forever
- **Rent the engine, build the wrapper:** axe-core, Playwright, Claude are rented. The **fix library** is the wrapper and the only real asset.
- **Build with Opus, run with Sonnet, run in Claude Code on the laptop — not the API.** Per-client run cost is $0 marginal that way. Cloud/API autonomy is a later optimization.
- **Everything compounds or it isn't done.** The fix library, the defendant database, and the error ledger are the compounding pieces.
- **Audit doctrine:** an audit checks EVERY item on a written checklist with evidence. Never report "done / passing / fixed" on a subset.

<!-- quality-gate-law:v1 -->
> # 🏭🛑⭐ THE QUALITY GATE LAW — **WHERE** the gate goes (MANDATORY, every build, every repo)
> **Having evals is not the skill. Knowing WHERE to put them is the skill.** A gate in the wrong
> place is worse than none — it reports green while the defect walks past.
>
> > **Prove the quality FIRST, then lock it in. Check at the station, not at the end.
> > Re-check on every change and every model upgrade.**
>
> ### The four placements — the answer to "where do I run the evals?"
> 1. **QUALITY gate — BEFORE you write the tests.** *"Tests lock in behavior. If the behavior is
>    mediocre, tests lock in mediocrity."* (Tan, `skillify` Phase 3). Prove it's good → THEN cement it.
>    For judgement calls: **3 frontier models from 3 DIFFERENT providers** — different families have
>    **less correlated blind spots**, like independent inspectors on a line.
> 2. **STATION gate (*jidoka*) — at the step that MAKES the defect, not a final inspection.**
>    🛑 TPS is **not** stage-gate; stage-gate is waterfall. The line stops the instant a defect appears.
> 3. **MERGE gate — on every change.** Evals live *beside* the thing. A change that doesn't improve
>    the cases doesn't ship. (Schmid, Google DeepMind.)
> 4. **UPGRADE gate — on every model upgrade.** Skills are contracts versioned to a model. When
>    ablation says the model no longer needs the skill: **retire the skill, KEEP the eval.**
>
> ### Which gate? Decided by WHICH SPACE the work computes in
> **DETERMINISTIC** (numbers, money, dates, SQL, links, structure) → **static regex/script asserts**;
> free, never flake, run on EVERY change. **LATENT** (taste, judgement, routing) → **model-judged**;
> expensive, so run it at the standard-setting moment and on upgrades.
> 🛑 **Cheap before expensive** — a static suite that runs every time beats a model suite nobody runs.
>
> ### 🛑 The three ways a gate betrays you
> 1. **It goes blind** — a check that silently stops checking (a renamed label, a moved file).
>    **Rename something → re-verify the gate still sees it.**
> 2. **You take it green by loosening it** — never weaken the pattern. **Fix the DETECTOR, say why.**
>    A gate passing because it went blind is worse than no gate: it looks healthy.
> 3. **It was never negative-tested** — a gate nobody has *seen fail* is not known to work.
>    **Inject a real violation, watch it fail, restore.** Once per gate, always.
>
> **Why the order isn't arbitrary:** improvement is measured *against* a standard, so the standard
> must be proven good first. Tests written before the quality gate don't just lock in mediocrity —
> **they make kaizen impossible.** Build quality IN; don't inspect it in.
> 📚 Full law: `coreflux-ai-os/brain/THE-QUALITY-GATE-LAW.md`
<!-- /quality-gate-law:v1 -->

---
> Source: [corefluxsystems-os/ADA-Compliance-OS](https://github.com/corefluxsystems-os/ADA-Compliance-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
