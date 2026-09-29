## audit-skills

> This file is pre-loaded by every agent working on this repo. It is the single source of truth for how to work here. The workflow is the **G0–G6 stage-gated pipeline** (§2): every unit of work passes every gate, and every gate is a checkable artifact — not a vibe.

# AGENTS.md — Runtime instructions for every agent in Audit-skills

This file is pre-loaded by every agent working on this repo. It is the single source of truth for how to work here. The workflow is the **G0–G6 stage-gated pipeline** (§2): every unit of work passes every gate, and every gate is a checkable artifact — not a vibe.

---

## 0. Before you do anything

### 0.1 Read `docs/lessons-learned.md`
That file captures every mistake we have made across 3 build waves and 3+ rounds of verification. The short version:
- **Never trust your own recall for factual claims.** Verify against live sources via `webfetch`.
- **Counts are always wrong by ±2.** Verify every count against the source document.
- **Every manifest URL must be live-checked.** URL rot is the #1 silent failure mode.
- **Build agents produce plausible-sounding fabricated identifiers.** The §5.11 verification gate is the only guardrail.

### 0.2 The ratchet rule
Every entry in `docs/lessons-learned.md` must map to a linter rule, a test, or a CI job. If you learn a new lesson, add it to lessons-learned **and** open a Linear ticket to mechanize it. A lesson that only lives in prose is not learned.

### 0.3 Know the repo structure
- `skills/<slug>/SKILL.md` — router, ≤300 lines, frontmatter + all 12 sections
- `skills/<slug>/chunks/NN-slug.md` — content chunks, ≤200 lines each
- `skills/<slug>/industries/` — 3-4 industry view files + `_index.md`
- `skills/<slug>/use-cases/` — 3-4 use cases + `_index.md`
- `skills/<slug>/tests/` — 6 skill-specific test files (oracle, grounding, trace, metamorphic, adversarial, telemetry) + `<slug>_stub.py`; lint + consistency run from root `tests/` parametrized over all skills
- `skills/<slug>/data/` — generators, seeds, crosswalks
- `skills/<slug>/docs/` — architecture, limits, changelog, acceptance-gate
- `skills/<slug>/telemetry/` — schema, instrument, redaction, baseline
- `data/registry/citations.json` — canonical citation registry: every §10 manifest label + URL. Add citations HERE first; manifests copy the registry URL verbatim (enforced by `tests/test_citation_registry.py`)
- `prompts/` — version-controlled agent prompts for every G4 review pass (use these verbatim; do NOT improvise review instructions)
- `tools/lint_skill.py` — Tier 0a linter (G3 gate)
- `tools/check_fact_sheet.py` — fact-sheet completeness checker (G1 gate)
- `tools/check_design_doc.py` — design-doc section checker (G2 gate)
- `tools/check_link_rot.py` — nightly URL liveness
- `tests/test_consistency_lib.py` — cross-document consistency checks
- `docs/fact-sheet-template.md` — Day 0 research template (with machine-readable data block)
- `docs/skill-design-template.md` — design doc template (15 sections)

---

## 1. Work types

Every change belongs to exactly one work type. All four run the same G0–G6 spine; they differ only in which G1–G4 artifacts are required.

| Work type | G1 Research | G2 Design | G3 Build | G4 Verify |
|---|---|---|---|---|
| **A. New CORPUS skill** | Fact-sheet (full) | 15-section design doc + file-requirements spec | Router + chunks + industries + UCs + 6 test files | 5-lens + §5.11 + persona vetting + consumer smoke test |
| **B. Skill edit** | Verify changed facts vs live sources; new/changed citations go into `data/registry/citations.json` first | — (PR description suffices) | Edited files | §5.11 on changed claims; re-run affected UC smoke test if routing/UC content changed |
| **C. ARGUS tool** (audit-skills-mcp repo) | Fact-sheet (formulas, authoritative params) | Golden reference cases (SOX-612 format) | Implementation + unit tests | Harness validation vs golden cases (Epic 6) |
| **D. GTM post** | Source artifact must exist in repo | — | Post draft in `claude-outputs/` | Artifact link resolves; claims match shipped skill content |

---

## 2. The G0–G6 pipeline

### G0 — Intake
- Every unit of work starts from a Linear ticket (SOX-NNN). No ticket → create one before starting.
- Branch from `main`: `<type>/SOX-NNN-short-slug` (e.g. `feat/SOX-567-iso-27001-build`).
- The eventual PR description must contain `Closes SOX-NNN` (or `Part of SOX-NNN` for partial work) so Linear transitions automatically.

### G1 — Research (Day 0)
- Copy `docs/fact-sheet-template.md` → `docs/<slug>-fact-sheet.md`.
- A research agent **with webfetch access** populates it: every identifier, every count, every crosswalk row, every URL (live-checked, status recorded), version/supersession info, exact terminology.
- The fact-sheet's **machine-readable data block** (§0 of the template) must be filled — it is what G3 tests assert against.
- **Gate:** `python3 tools/check_fact_sheet.py docs/<slug>-fact-sheet.md` passes. Do not proceed until it does. If a build agent later needs a fact that isn't in the fact-sheet, the fact-sheet is incomplete — return to G1.
- **Licensed sources (process v3.1, SOX-618):** when a framework's normative text is licence-restricted (ISO, COSO): (a) the licensed file lives OUTSIDE the repo (`~/Standards-licensed/`) and is NEVER read by any agent, entered into any prompt/pipeline, or quoted beyond short attributed excerpts — ISO's licence has an explicit AI/ML prohibition (no prompting, querying, extraction; survey: `docs/builds/sox-618/iso-source-survey.md`); (b) the fact sheet builds its machine-verifiable layer from free sources and marks requirement-semantics rows `pending_human_verification` with a clause pointer; (c) generate a human-verification worksheet (`prompts/human-verification-worksheet.md`), the human ticks/corrects against the licensed copy and signs; (d) rows flip to `verified_by: human-licensed-copy` — the G1 gate fails while any `pending_human_verification` remains; (e) acquisition is recorded in `source-texts/manifest.json` (à-la-carte at each skill's Day-0; EVS for EN ISO adoptions).

### G2 — Design
- Copy `docs/skill-design-template.md` → `docs/<slug>-design.md`; fill all 15 sections. Write the file-requirements spec (`docs/<slug>-file-requirements.md`).
- **Gate:** `python3 tools/check_design_doc.py docs/<slug>-design.md` passes.

### G3 — Build
- 1 router agent for SKILL.md (knows full architecture) + N parallel chunk/industry/UC agents.
- **Eval lane ships with the build (process v3.1, M4):** `evals/<slug>/cases/` with at least the UC substance cases + 1 idempotence invariant + the perturbation set (refusal contract), oracle-labeled via the stub. They run in CI against the stub executor from the build PR onward.
- Build agents cite **only the fact-sheet** — never their own recall. Build agents do NOT get webfetch (they hallucinate anyway; building is for structure, G1/G4 are for facts).
- **Gate:** `tools/lint_skill.py` passes; per-skill tests + consistency lib pass; every count/identifier in chunks matches the fact-sheet data block.

### G4 — Verify (use `prompts/` verbatim)
Run in order; each pass uses its version-controlled prompt:
1. **5-lens review** — `prompts/five-lens-review.md`. 5 agents in parallel, one per lens.
2. **§5.11 source-of-truth verification** — `prompts/s511-verification.md`. 1+ agent **with webfetch**, every factual claim vs live sources.
3. **Fix pass** — `prompts/fix-pass.md`. 1 agent per skill, exact findings with line numbers. Fix agents do NOT commit or push.
4. **Re-verify** — rerun `prompts/s511-verification.md` on the fixed files. Fix agents introduce new errors; re-verification is not optional, and it happens **before the PR merges**, never after.
5. **Consumer-ready gate (G4.5)** — required for work type A; see §3.

**Verification tiers — what counts as "verified" (learned 2026-06-09, nist-800-53-rmf vetting):**
- **Tier 1 — mechanical:** proved by computation on disk (counted, diffed, grepped, executed). Strongest; always do this first for internal-consistency claims.
- **Tier 2 — live source:** a webfetch agent fetched the authoritative source and quoted the supporting sentence verbatim. Required for EVERY external framework fact that is added or changed.
- **Tier 3 — concurrence:** the reviewer's knowledge agrees with other agents' knowledge. **Concurrence is a triage signal, NOT verification.** A fix justified only by Tier 3 is an unverified fix — it must be Tier-2 verified before the PR merges, or written as a verify-against caveat instead of a specific claim.

**Process v3 rules (learned 2026-06-10, M1 sweep — 6/6 vetted, PRs #32–#41):**
- **Currency and negative-existence claims are ALWAYS Tier-2.** No LLM source (persona, builder, reviewer — or the dispatcher) can establish that something "doesn't exist," "is the current version," or "was never published." The M1 sweep refuted persona CONSENSUS four times, every time on a post-training-cutoff event (ITAF 5th Edition exists, ISACA is the CMMC CAICO, CISA CPG 2.0 exists, the tiers-not-maturity quote lives in the 1.1-era FAQ). Editions, versions, sunsets, supersessions, org roles, and program governance are recency-sensitive by definition: fetch or caveat, never assert.
- **The seed + oracle are the contract.** When UC documents, README examples, chunks, the stub, and the tests disagree, the tested fixture (seed + oracle assertions) is the source of truth: rebuild the docs to it, or consciously change the contract everywhere at once. Never patch one artifact in isolation (nist-csf-2 shipped ALL THREE UC docs describing different companies than their own seeds).
- **Inventory-diff for catalog-bearing skills.** When a skill encodes a framework catalog (TSC criteria, COBIT objectives, CSF subcategories, 800-171 requirements), verify by transcribing/parsing the FULL official inventory once (PDF text-extraction or official JSON export) and diffing the skill against it. Spot-checking misses offsetting errors (an invented BAI12 masked a missing MEA04 because the total still summed to 40).
- **Anti-hallucination and limits sections get priority verification.** Fabrications concentrate exactly where trust anchors live — M1 found invented statutes, fake SLAs, wrong editions, and fabricated citations INSIDE anti-hallucination sections in 4 of 6 skills. Verify every specific claim in these sections first.
- **House conventions must be labeled, never attributed.** Any formula, scale, weighting, threshold, or rollup not verbatim in a cited source (risk scores, tier averages, status scales, %-gap math, design-factor weights) is labeled "house convention / illustrative" with an attribution note. Attributing invented math to a named publication is a CRITICAL.
- **Verification artifacts must carry evidence, or they are noise.** A ✓-verified row without a verbatim quote is Tier-3 concurrence. A gate containing unresolved text ("need recount", bracketed `[VERIFY:` author-TODOs) is a failed gate — the linter now rejects `[VERIFY` markers outside the changelog; shipped caveats are written as prose ("verify X against Y before client use"), never as author TODO tags.

Output: `skills/<slug>/docs/acceptance-gate.md` — fact | source | retrieval date | verifier | status, **with a verbatim source quote for every Tier-2 row**. ≥20 rows. Any FAIL blocks release.

### G5 — Ship
- **Everything merges via PR. No direct pushes to `main`, no admin bypass — including docs-only changes.**
- PR title follows `type(skill-slug): description`; PR body uses the template, pastes linter + pytest output, and contains `Closes SOX-NNN`.
- All 3 CI checks green before merge. Squash-merge.

### G6 — Release
- Update the skill's `docs/changelog.md` and bump its semver (per the architecture constitution, SOX-608).
- **README + pointer sync (process v3.1, mandatory):** a new skill changes the skill count, the test totals, and the framework list. Run `python3 tools/check_readme_sync.py` and fix every drift item before merge — it asserts the README skills table, banner/CI/quickstart test counts, and the category-pointer body+description all match what is actually on disk (CI-enforced via `tests/test_readme_sync.py`). When the library version bumps, add the new `## What's in this release` section. This catches the partial-update rot where a PR fixes some numbers but not all.
- **Reliability rows (process v3.1):** before the build PR merges, run a calibration sweep (`evals/harness/runner.py --executor llm`, N=2, Haiku) and fix contract-grammar findings. Before any release/GTM claim about the skill, run the publication sweep (N>=20, Haiku + Sonnet — never Fable/Opus, measurement-mismatched) and publish the rows in `docs/acceptance-gate.md`. "Fully tested" means a measured pass rate.
- README version table updated when the library-level version bumps.

---

## 3. The consumer-ready gate (G4.5) — usable, tested, vetted from the get-go

A skill is not done when its tests pass. It is done when a stranger can use it correctly in their first session. Four requirements (tracked in SOX-633):

**What this gate certifies — and does not.** Passing G4.5 means the skill passes every defined check with an auditable evidence trail. It does NOT guarantee consumer success: the smoke test is N=1 on one model (reliability requires the Epic 6 harness: ≥95% pass rate over N≥20 runs, ≥2 models); persona vetting is LLM agents role-playing practitioners (a filter, not a certification — real-practitioner sign-off comes from the design-partner loop); §5.11 facts are point-in-time and decay (a verified URL went 404 within 2 days once). State claims accordingly: "passes all defined gates," never "guaranteed." Full measurable DoD: SOX-633.

### 3.1 Usable from the get-go
- The skill's `README.md` has a quickstart: install → first useful output in under 10 minutes, with a copy-paste invocation example and an input template per UC.
- No quickstart, no release.

### 3.2 Fresh-agent smoke test
- Load the skill into a **clean agent session** (no repo context), run each UC's input verbatim, and check the output against the UC oracle.
- This proves the skill works as a *prompt*, not just as linted markdown. The test files validate the stub; the smoke test validates the skill.
- Use `prompts/consumer-smoke-test.md`. Record results as rows in `docs/acceptance-gate.md`.
- When the Epic 6 harness lands (SOX-600), this becomes automated: N runs per golden case, pass-rate ≥95%, cross-model matrix. Until then it is manual and mandatory.

### 3.3 Persona vetting (perspective-vetted)
- The 5 practitioner personas from the design template §7 are not a design-time thought exercise — they run **against the built skill** as review agents using `prompts/persona-vetting.md`.
- Output: `skills/<slug>/docs/persona-review.md` — persona | finding | severity | resolution. Blocking findings fixed before release.

### 3.3.1 Per-skill vetting runbook (the corrected order — run it exactly like this)
1. **Persona vetting + smoke tests** in parallel (5 persona agents + 1 clean session per UC). Smoke agents self-grade against the UC oracle and report **every cross-file contradiction** they notice — smoke contradiction lists were the highest-yield seed/doc-drift detector in M1.
2. **Verify every CRITICAL/HIGH finding before fixing it** — findings are hypotheses, not facts: Tier 1 (mechanical) for internal claims, Tier 2 (live source) for external claims. Drop findings that don't verify. **Persona "X doesn't exist / X is current" claims are currency claims — mandatory Tier-2 even at full consensus** (refuted 4 times in M1).
3. **UC-doc ↔ seed/oracle coherence check (Tier 1):** diff every UC document's stated facts (org name, sizes, counts, IDs, expected values, file pointers) against its `data_refs` seed and its `tests` assertions. The seed + oracle are the contract; docs rebuild to them.
4. **Fix pass** — verified findings only. Where a correct value can't be Tier-2 verified yet, write a prose verify-against caveat (never a `[VERIFY:` author tag), not a new specific claim. Label all non-source formulas/scales as house conventions.
5. **§5.11 re-verify (live, webfetch) every external fact the fix pass touched** — BEFORE the PR merges. Evidence rows with verbatim quotes go into acceptance-gate.md; re-derive (don't merely edit) any gate row the fix pass invalidated.
6. Re-run any FAILed smoke test in a fresh clean session after the fix.
7. Structural findings that can't be inline-fixed safely get tickets; persona-review.md records every finding's resolution (fixed PR-N / ticketed SOX-N / accepted-with-rationale). Zero unresolved CRITICAL/HIGH to pass the gate.

### 3.4 Industry vetting
- Each `industries/<industry>.md` file gets a persona review from that industry's practitioner persona (same prompt file, industry section).
- Human loop: design-partner reviewers recruited from the drip audience get early access in exchange for structured review; their findings feed the next revision.
- Telemetry from real usage (per-skill `telemetry/`) feeds the next revision.

---

## 4. Commit and PR conventions

```
type(skill-slug): description
```

Types: `feat` (new content), `fix` (correction), `docs` (non-content docs), `chore` (tooling/CI).

Examples:
- `feat(iso-27001): scaffold SKILL.md and 7 chunks`
- `fix(nist-800-53-rmf): replace fabricated Rev 5.1.1 with Rev 5`
- `chore: add fact-sheet completeness checker`

Never use: `Merge`, `WIP`, `stuff`, mixed-case scopes, periods at end of subject.

Branches: `<type>/SOX-NNN-short-slug`. PRs: template filled, test evidence pasted, `Closes SOX-NNN` present.

---

## 5. Test commands

```bash
# G1 gate — fact-sheet completeness
python3 tools/check_fact_sheet.py docs/<slug>-fact-sheet.md

# G2 gate — design doc sections
python3 tools/check_design_doc.py docs/<slug>-design.md

# G3 gate — per-skill linter
python3 tools/lint_skill.py skills/<slug>

# Per-skill tests
python3 -m pytest skills/<slug>/tests/ tests/test_consistency_lib.py -q -p no:asyncio

# Full test suite
python3 -m pytest skills/ tests/ -q -p no:asyncio

# All linters
python3 tools/lint_skill.py skills/nist-800-53-rmf skills/isaca-audit-methodology \
  skills/coso-internal-controls skills/aicpa-soc-reporting \
  skills/audit-workpapers skills/nist-csf-2
```

---

## 6. Common mistakes (do NOT do these)

### 6.1 Fabricating identifiers
Bad: writing `GV.SR`, `GV.MT`, `PR.AC` from recall.
Good: copying identifiers from the Day 0 fact-sheet (populated from live sources).

### 6.2 Wrong counts
Bad: "108 Subcategories" (CSF 1.1), "81 Points of Focus" (wrong).
Good: verify every count against the fact-sheet data block.

### 6.3 Writing to config path
Bad: agent writes to `~/.config/opencode/skills/<skill>/` instead of the repo.
Prevention: always `git diff` after agent completion. If diff is empty but agent says it edited files, the files went to the wrong path.

### 6.4 Not running tests before declaring done
Bad: "All done, 266/266 pass" without actually running `pytest`.
Good: run the full suite and paste the output.

### 6.5 Stub telemetry/instrument.py without full API surface
Every `telemetry/instrument.py` must export `instrumented`, `SkillInvocation`, `skill_invocation` to prevent module shadowing in CI.

### 6.6 Skipping re-verification after a fix pass
Fix agents introduce new errors. After every fix pass, rerun the §5.11 prompt.

### 6.7 Pushing directly to main
Bad: `git push origin main` with admin bypass — even for docs.
Good: branch → PR → 3 green checks → squash-merge.

### 6.8 Improvising review instructions
Bad: writing a fresh 5-lens or §5.11 prompt from memory each run.
Good: use the version-controlled prompt in `prompts/` verbatim. If the prompt is inadequate, improve the file in a PR — that's the ratchet.

---

## 7. Agent dispatch patterns

### 7.1 Build agents
- Parallel for independent tasks (one per chunk, one per industry file)
- Single router agent for SKILL.md (must know full architecture)
- Build agents read from the fact-sheet, NOT from their own recall; no webfetch

### 7.2 Review agents
- 5-lens review: 5 agents in parallel, one per lens, prompts from `prompts/five-lens-review.md`
- §5.11 verification: 1 agent per skill, webfetch access required, prompt from `prompts/s511-verification.md`
- Persona vetting: 5 agents in parallel, prompt from `prompts/persona-vetting.md`
- All review agents receive explicit checklists — from the prompt files, never improvised

### 7.3 Fix agents
- 1 agent per skill, dispatched in parallel, prompt from `prompts/fix-pass.md`
- Each receives EXACT findings with line numbers and expected values
- Instructed: "Do NOT commit or push"
- After completion: verify `git diff` is in repo path

---

## 8. When in doubt

- Read `docs/lessons-learned.md`
- Run the gate checker for the stage you're in
- Verify against live sources via webfetch
- Ask before assuming

---
> Source: [amurthygithub/Audit-skills](https://github.com/amurthygithub/Audit-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
