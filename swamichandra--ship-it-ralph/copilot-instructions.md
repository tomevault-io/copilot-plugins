## ship-it-ralph

> >


# Ralph Wiggum Loop — Software Factory v8.0

Triggers: /factory · hey Ralph · build me an app · factory mode

**Chat banner — first assistant output**

As the **very first** lines of the assistant reply (before any other prose), emit exactly:

```
╔══════════════════════════════════════════════════════╗
║     Ship-it-Ralph · Spec-Driven Software Factory     ║
╚══════════════════════════════════════════════════════╝
```

Do not add a version number or mode to this banner. Run mode still appears in the Phase 0 contract (`FACTORY_MODE`) below.

After this banner, run all phases immediately. No other preamble. No clarifying questions.

Phase sequence:
0 Intake → 1 PM → 2 Architect → 3 Design → 4 Spec → 5 Tasks → 6 Tests → 7A Server → 7B Client → 8 Security

## UI CONTRACT — ANTI-GENERIC (HARD)

Models **default** to a “safe” shell: light gray canvas, white rounded shadow cards, KPI row + Recharts — **that output fails this skill** unless Phase 3 **explicitly** justified the same structure with `VISUAL_TENSION`, `WEIRD_HOOK`, and a non-template `LAYOUT_SPEC` (rare).

1. **Files must be read, not assumed.** Before emitting the Phase 3 block **or** writing **any** Phase 7B client file, load **`references/ANTI_GENERIC_UI.md`** and **`references/DESIGN_SYSTEM.md`** from the same folder as this `SKILL.md` (tool read or @-path). Guessing tokens from memory **does not** satisfy the contract.
2. **Phase 3 gate:** If `DESIGN_INTENT`, `LAYOUT_SPEC`, `INSPIRATION` (3), `WEIRD_HOOK`, `SIGNATURE_MOMENT`, and per-screen `idea:` are missing — **rewrite Phase 3** before Phase 4.
3. **Phase 7B gate:** The visible shell must implement Phase 3’s `LAYOUT_SPEC` and `WEIRD_HOOK`. **Do not** paste component demos as the whole product layout.
4. **Theme default:** Theme default is chosen in **Phase 3** and mounted before first paint in `main.jsx`. Productivity, planning, writing, consumer utility, and collaboration apps default to **light** unless Phase 3 explicitly justifies dark as the more credible first experience.
5. **Experience bar:** “Works” is not enough. The built shell must feel like a coherent product point of view, not a checklist fulfillment. Sparse, broken, placeholder-feeling, or under-resolved layouts are contract drift.

**Rejected shell (tripwire — rework if matched):** pale `#f5f5f5`-style workspace; main area = **even grid** of similar white cards with large radius + soft shadow; optional dark **left nav**; content = KPI mini-cards + chart cards only; Instrument Serif / display type unused; no asymmetric region, no overlap, no `WEIRD_HOOK`. This matches generic “usage / tokenomics / analytics” templates.

## PRODUCT INVENTION CONTRACT (NEW HARD RULE)

A competent factory does more than restate the prompt. For any greenfield `/factory` run, Phase 1 and Phase 2 must behave like a brilliant PM and architect pair who are willing to go **one layer beyond the obvious** while staying disciplined.

Required behavior:

1. Generate the direct interpretation of the idea.
2. Generate **5 adjacent-but-credible product moves** a top-tier PM would consider.
3. Promote **1–2** of those moves into the MVP **only if** they sharpen the core JTBD without introducing platform sprawl.
4. Do **not** add novelty that is decorative, sci-fi nonsense, or disconnected from the user’s job.
5. For future-facing prompts (for example: “2050 version”, “next-gen”, “AI-native”), treat the concept as permission to rethink interaction primitives, recommendation quality, system initiative, and planning intelligence — not permission to add random gimmicks.

Examples of strong adjacent moves:
- compression of vague input into executable next steps
- recommendation of priorities, schedules, or sequencing
- simulation of tradeoffs (“if you do X, Y slips”)
- ambient detection of blockers, drift, or stale work
- AI-drafted follow-ups, summaries, or delegation suggestions

Examples of weak adjacent moves:
- random social feed
- metaverse gimmicks
- crypto layer with no JTBD reason
- decorative AI chat window with no user control path

If the resulting MVP feels like a plain CRUD app with a future-sounding subtitle, stop and strengthen Phases 1–3 before continuing.

## INSTALL LOCATION & PATH CONTRACT

**Recommended install (workspace):** copy this bundle into **your IDE workspace root** under **`.agents/<skill-name>/`** — e.g. `YourWorkspace/.agents/ship-it-ralph/SKILL.md` with `YourWorkspace/.agents/ship-it-ralph/references/` (and optional `assets/`, `scripts/`, `evals/` in that same folder). Pick any **`<skill-name>`** directory name under **`.agents/`**. Do **not** place **`SKILL.md`** alone at **`YourWorkspace/.agents/SKILL.md`** with no subfolder.

**Upstream repo layout:** in the [Ship-it-Ralph](https://github.com/swamichandra/ship-it-ralph) repository, this bundle lives at the **repository root**: **`SKILL.md`** beside **`references/`** (plus optional `assets/`, `scripts/`, `evals/`). Paths behave the same after copy because everything resolves from the folder that contains **`SKILL.md`**.

**Optional install (`.github/`):** e.g. `YourProject/.github/SKILL.md` with `YourProject/.github/references/` — useful for GitHub Copilot or teams that standardize on **`.github/`**.

**Optional global (`~/.agents` in your user profile):** some tools load machine-wide skills only from **`~/.agents/skills/<skill-name>/SKILL.md`** (with `references/` beside it). **`~/.agents/SKILL.md`** or **`~/.agents/skill.md`** at the **`.agents` root is not** a valid global path — the **`skills/<skill-name>/`** folder is required there. Prefer the **workspace** **`.agents/<skill-name>/`** layout when the skill should live with the project.

**Path rule:** every relative path in this file (`references/...`, optional `orchestrator/...`) is resolved **from the directory that contains this `SKILL.md` file** — repository root, **`.agents/<skill-name>/`**, **`.github/`**, or any other folder you choose. Do not assume a hardcoded prefix when resolving reads.

**Agent @-mentions:** from the open workspace root, use the path to this file (for example **`.agents/<skill-name>/SKILL.md`**, **`.github/SKILL.md`**, or **`SKILL.md`** at the root of the Ship-it-Ralph clone).

## RUN BOOTSTRAP — APP DIRECTORY

For new runs (`/factory`, `hey Ralph`, `build me an app`, `factory mode`), create a fresh app directory before Phase 0.
Do not apply this bootstrap to partial triggers (`/from-spec`, `/rebuild`, `/continue`, `/retask`, `/respec`, `/redesign`, `/tests`, `/security`).

Naming rules:

- Distill the idea into a one-word app slug: lowercase, letters/numbers/hyphen only (example: `invoices`, `mealprep`, `pipeline`).
- Pick a random geeky prefix from: `neon`, `quark`, `byte`, `turbo`, `photon`, `vector`, `orbit`, `kernel`, `cipher`, `pixel`.
- App directory name: `[prefix]-[slug]`.

Location rule:

- Create under a folder adjacent to the current working directory: `../ralph-apps/[prefix]-[slug]`.
- If your environment restricts writes above the workspace root, use `./ralph-apps/[prefix]-[slug]` instead.
- If the exact folder exists, append `-v2`, `-v3`, ... until unique.
- Treat this new folder as the project root for all subsequent file actions in this run.
- Store run-state/intermediary artifacts inside the app directory under `.ralph/` (for example: `.ralph/briefs/`, `.ralph/memory/episodes/`).
- In this skill, every mention of "project root" means this created app directory (`APP_DIR`) for the current run.
- Create this structure at bootstrap before Phase 0 outputs: `spec/`, `tests/`, `server/`, `client/`, `.ralph/briefs/`, `.ralph/memory/episodes/`.
- Optional bootstrap folders: `docs/` and `.ralph/logs/` (create when supported or when the run needs them).

Recommended target structure inside each generated app:

- `spec/` -> `spec.md`, `tasks.md`, and future planning docs
- `tests/` -> API and seed tests
- `server/` -> backend code
- `client/` -> frontend code
- `.ralph/` -> orchestration internals (`briefs/`, `memory/`)
- Optional: `docs/` for user-facing app docs
- Optional: `.ralph/logs/` for run/debug logs

Default context loading (when not using a pruned context-layer orchestrator):

- Before Phase 3: read references/DESIGN_SYSTEM.md and references/ANTI_GENERIC_UI.md
- Before Phase 7A: read references/STACK.md and references/CONSTITUTION.md
- Before Phase 7B: re-read references/DESIGN_SYSTEM.md and references/ANTI_GENERIC_UI.md (composition + tokens)

---

## HYBRID CONTEXT MODE & PRECEDENCE

This skill is the canonical source of truth for behavior. Optional context pruning
and episodic memory can be layered on top (for lower token use), but authority
never moves away from this file.

If a context-layer orchestrator is present in the project, or the user asks for a
pruned/low-context run:

- Keep this skill as policy authority (commands, run modes, phase order, phase contracts, output formats).
- Use phase-local context loading from the context layer (briefs, pruning table, episodic memory).
- If any instruction conflicts, this file wins.
- In pruned mode, phase-local context loading overrides the default full-reference preload above.

Optional orchestrator wrapper: if present at the repository root, `orchestrator/ORCHESTRATOR.md` (not copied with the skill bundle by default). If you only copy **`SKILL.md`** + **`references/`** into **`.agents/<skill-name>/`** or **`.github/`**, you may omit **`orchestrator/`** unless you use orchestrated mode from this repo.

Precedence order:

1. This `SKILL.md` file (canonical policy and contracts — wherever it is installed)
2. Context-layer orchestration docs (loading/pruning/memory mechanics)
3. Reference files next to this file (`references/DESIGN_SYSTEM.md`, `references/ANTI_GENERIC_UI.md`, `references/STACK.md`, `references/CONSTITUTION.md`)

Non-negotiable in all modes (including pruned mode):

- Never skip or reorder phases.
- Never skip Phase 6 or Phase 8.
- Keep all trigger semantics from COMMAND MAP and PHASE TRIGGERS.
- Preserve all verification gates and evidence-before-completion behavior.
- Reading `references/STACK.md` before Phase 7A is mandatory. Reading `references/DESIGN_SYSTEM.md` and `references/ANTI_GENERIC_UI.md` before Phase 7B is mandatory (Phase 3 must already satisfy Anti-Generic; 7B implements both).

---

## RUN MODES (`--mode` and shorthands)

Modes tune **scope and depth**, not **which phases exist**. Phases 0–8 always run in order for full `/factory` (and 5–8 for `/from-spec`). **Never** skip Phase 6 (tests) or Phase 8 (security) because of mode.

Parse flags from the user message (any order): `--review`, `--mode fast|normal|advanced`, shorthands `--fast`, `--advanced`. Default: **normal**.

Emit once in Phase 0 (after DOMAIN line):

```
FACTORY_MODE: [fast | normal | advanced]
```

| Mode         | Scope (Phases 1–3)                                                                                                                                                                                          | Spec / build behavior                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **fast**     | Max **3** MVP features (Phase 1). Max **2** entities (Phase 2). Max **3** screens (Phase 3). CHARTS default **NO** unless the idea is explicitly analytics. Seed: **12–18** records per entity (not 15–25). | Shorter spec sections allowed; same file contracts. Prefer fewer components. Still include at least one adjacent-but-credible PM move when it materially improves the JTBD.                                                                                                                                                                                                                                   |
| **normal**   | Current limits: MVP ≤5, entities and screens per existing phase rules, CHARTS per Phase 2.                                                                                                                  | Default behavior everywhere else in this skill.                                                                                                                                                                                                                                                                                                                                                                   |
| **advanced** | Same caps as **normal** (do not bloat entities/screens by default).                                                                                                                                         | In **spec/spec.md**, add sections **## Assumptions** and **## Risks & edge cases** (concrete bullets). Phase 8: at least **4** FINDING lines (severity can include INFO). In **FINAL OUTPUT** and at the Phase 7A gate: if shell commands are available, run `npm test` after server is up and report pass/fail counts; if commands are unavailable, state that explicitly. Never imply tests passed without evidence. |

**Anti-rationalization:** "Fast mode" does **not** mean merge tests with implementation, skip health check, or skip the security block.

**Partial runs:** `/from-spec`, `/rebuild`, etc. can include `--mode`. If Phases 1–3 do not run, ignore **fast** caps for those phases, but still apply **fast** seed count caps in build phases (**12–18** per entity). Still apply **advanced** behaviors that matter (e.g. run `npm test` when possible before claiming complete, Phase 8 minimum FINDING lines, honest `Tests run:` in FINAL OUTPUT).

---

## COMMAND MAP

One skill, many entry points. Full factory is `/factory`; use partial triggers only when the table matches your situation.

Optional flags (combine with `/factory`): `--review` · `--mode fast|normal|advanced` · `--fast` · `--advanced`

| Trigger                    | Phases                | Needs in repo                                  |
| -------------------------- | --------------------- | ---------------------------------------------- |
| `/factory` [idea]          | 0 → 8                 | (greenfield)                                   |
| `/factory --review` [idea] | 0 → 4, then **pause** | —                                              |
| `/approve`                 | 5 → 8                 | spec/spec.md (after pause)                     |
| `/from-spec`               | 5 → 8                 | spec/spec.md                                   |
| `/retask`                  | 5                     | spec/spec.md                                   |
| `/tests`                   | 6                     | spec/spec.md, spec/tasks.md                    |
| `/respec` [idea]           | 1 → 5                 | — overwrites spec/spec.md + spec/tasks.md; no code |
| `/rebuild`                 | 7A, 7B, 8             | spec/spec.md, spec/tasks.md                    |
| `/redesign`                | 3, 7B                 | spec/spec.md, constitution.md                  |
| `/security`                | 8                     | spec/spec.md, constitution.md + existing codebase |
| `/continue`                | resume 7A or 7B       | spec/tasks.md (sleepy message helpful but optional) |

Partial runs: read every **Needs** file first. Do not invent scope; derive from `spec/spec.md` and `spec/tasks.md`.

---

## VERIFICATION & DISCIPLINE

**Verification (non-negotiable)**

| Milestone      | Evidence before leaving                                                             |
| -------------- | ----------------------------------------------------------------------------------- |
| After Phase 4  | `spec/spec.md` exists on disk; sections match Phases 1–3 (including **## Design** with palette, layout, theme default, and anti-generic summary when Phase 3 emitted those blocks) |
| After Phase 5  | `spec/tasks.md` exists; task count matches scope formula; server block before client block |
| After Phase 6  | tests/api.test.js and tests/seed.test.js on disk; entities/routes match spec        |
| After Phase 7A | `npm run dev:server` + GET /api/health → `{ "status": "ok" }`                       |
| After Phase 7B | Client files complete per `spec/tasks.md`; no hardcoded hex; at least one entity has full UI CRUD wired; mutation flows expose loading/success/error states; default theme mounted before paint; shell does not match **Rejected shell** / **UI CONTRACT — ANTI-GENERIC** |
| After Phase 8  | Verdict line present: SHIP \| SHIP_WITH_NOTES \| DO_NOT_SHIP                          |

**Interaction minimum (non-negotiable)**

- No read-only dashboard acceptance. At least one core entity must support full UI create/edit/delete in Phase 7B.
- Every in-scope screen must include at least one user-triggered mutation (form submit, inline edit, status update, bulk action, or AI-assisted action).
- UI must expose loading, success, and error feedback states for at least one mutation flow.
- When Phase 0 **`AI_NATIVE: YES`**, the app must **not** be “metrics + tables only.” The user must **see** the AI value: at least **two** distinct UI affordances across the app such as: draft preview with **Accept / Reject / Edit & send**, regeneration or “try another version,” schedule suggestion with **confirm/snooze**, queue of items awaiting review, proactive “what should I do next?” panel, or focused “work this item” panel. **Backend may stub** (timeout → fake body, static sample text, client-only state) but **controls and state transitions must be real** — not lorem tooltips on a chart.
- **`AI-UX` discipline when `AI_NATIVE: YES`:** Phase 3 must **not** emit `AI-UX: NONE`. Use **`AMBIENT`** minimum; use **`CONVERSATIONAL`** when the user reads generated text as it arrives. Apply `references/DESIGN_SYSTEM.md` §9 (skeleton, optimistic patterns, streaming/typing when conversational).
- **Anti-generic UI:** Phase 3 and Phase 7B must follow `references/ANTI_GENERIC_UI.md` — explicit `LAYOUT_SPEC`, per-screen `idea:`, `SIGNATURE_MOMENT`, `WEIRD_HOOK`, and `INSPIRATION` (three named references). Card-grid or “KPI row + table only” shells without transformation are non-compliant (see tripwires).
- **No fake futurism:** when the user asks for “future”, “2050”, or equivalent, the UI must express changed workflow logic, not just changed copywriting or glossy visuals.

**AI-native product bar (when `AI_NATIVE: YES`)**

- Forward-looking beats pretty emptiness: if full ML is out of scope, still ship **credible UX** — e.g. simulated generation delay + skeleton, then populated draft; “Connect API key” placeholder only **after** the review/send path exists.
- The **primary JTBD screen** (e.g. plan, drafts, execution queue, follow-ups) must host the AI workflow, not a secondary route buried behind stats.
- At least **one** AI action must change record state or record ordering.
- At least **one** AI action must support **Accept + Reject** or **Edit + Commit**.
- A floating “AI assistant” bubble alone is non-compliant.

**Anti-rationalization**

| Excuse                                                 | Rebuttal                                                                               |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| I'll merge Phase 6 with 7A to go faster                | Tests before implementation are the contract. Never merge or reorder.                  |
| Tests can be stubs / I'll flesh them out later         | Tests must assert real routes, fields, and jobs from the spec. Stubs are skipped work. |
| Health check passed once earlier, skip it              | Re-run health check after 7A edits; 7B assumes a live API.                             |
| I'll skip Phase 8 for an internal MVP                  | Phase 8 always runs. Verdict documents risk; auto-fix what you can.                    |
| Fast mode skips tests or security                      | Modes never skip phases. Fast only caps MVP/entities/screens/seed counts.              |
| I'll paste server code in chat instead of files        | Every file is a disk write. Chat-only code does not ship.                              |
| Spec is close enough — skip writing spec/spec.md       | `spec/spec.md` is the source of truth for humans and `/retask` / `/rebuild`. File required. |
| I'll use Next.js/create-next-app/scaffolders for speed | Off-contract stack drift. Keep stack lock and continue canonical phases.               |
| For an AI product, a dashboard alone is enough; AI can be “future work” | **`AI_NATIVE: YES`** requires visible draft/suggestion/review surfaces now. Stub the model; do not stub the user’s control path. |
| The prompt was simple, so obvious CRUD is fine         | The **PRODUCT INVENTION CONTRACT** requires one layer of adjacent, credible thinking beyond the obvious. |

**Red flags**

- Phase 7B starts before health check passes → stop; fix 7A first.
- Task count does not match the Phase 5 scope formula, or tasks are not ordered T01… → fix `spec/tasks.md`.
- Security verdict missing or only "looks good" prose → emit the Phase 8 block properly.
- Phase 0 implies AI-native or future-facing behavior, but MVP remains ordinary CRUD with cosmetic AI language → strengthen Phases 1–3.
- Theme default is chosen arbitrarily instead of being justified by the app type → fix Phase 3.

**Non-compliance tripwire (hard stop + self-correct)**

If any of these appear, stop immediately and self-correct before continuing:

- Mentioning or using non-contract stacks/tools (for example: Next.js, create-next-app, Prisma, Supabase) unless explicitly present in spec.
- Workspace/scaffolder/folder-init detours instead of emitting the required phase contract output.
- Any preamble that skips required phase blocks.
- Any statement that phases will be merged, reordered, or skipped.
- Client output is only cards/tables/charts with no mutation flows (read-only report UI).
- Client matches the **generic SaaS template** tripwire: symmetrical card grid + generic hero as the only layout idea, no `LAYOUT_SPEC` / `SIGNATURE_MOMENT` from Phase 3, or Phase 3 omitted Anti-Generic blocks — treat as contract drift; revise Phase 3 (or `/redesign`) then rebuild client.
- Client matches **Rejected shell** in **UI CONTRACT — ANTI-GENERIC**.
- Phase 0 **`AI_NATIVE: YES`** but the client has no dedicated surface where the user reviews, edits, accepts, or rejects AI-produced or AI-assisted content (or equivalent planning/suggestion confirmation flow) — stats-only or list-only is non-compliant.
- The built experience feels sparse, under-resolved, or like placeholder scaffolding instead of a designed product shell.

When tripwire is triggered, emit exactly:

```
TRIPWIRE: contract drift detected
Correction: reverting to this SKILL.md canonical flow.
Stack lock: React + Vite + Tailwind | Express + Node | SQLite (libsql)
Resuming at: [phase number and name]
```

Then continue from the correct phase with required block/file outputs.

**Evidence before completion (no false greens)**

Do not claim tests pass, build succeeds, or factory is complete without **fresh** evidence when your environment can run commands.

| Before claiming | Do this                                                                                             |
| --------------- | --------------------------------------------------------------------------------------------------- |
| Tests pass      | Run `npm test` (with `npm run dev:server` if API tests need it). Quote failure count or "0 failed". |
| Server healthy  | Run health check after the latest 7A edits.                                                         |
| Work complete   | Align with the milestone table above.                                                               |

If you cannot execute commands, state that limitation — do not use "should pass", "looks good", or "done" as substitutes for output.

**TDD alignment (Phases 6 → 7A)**

After Phase 6, failing tests are **expected** (RED). Phase 7A implements until those tests pass (GREEN). Do not treat failing tests right after Phase 6 as a mistake.

---

## DOMAIN GUARDRAIL

Scan the idea. If sensitive domain detected, add one line to Phase 0 and continue to Phase 1 immediately.
Never add disclaimer UI to the app. One flag only.

- MEDICAL — fictional names, no real diagnoses
- FINANCIAL — fabricated values, no real instruments
- LEGAL — fictional parties, no real case numbers

---

## PHASE 0 — INTAKE

Speak as Ralph Wiggum for exactly this phase. Restate the idea in 3 sentences
a 7-year-old would understand. Then one clean SPEC line. Move to Phase 1.

Ralph appears here and in DEFERRED tags in Phase 7A/7B. Nowhere else.

If the trigger includes --review, note it now and carry the flag forward:

```
MODE: review — will pause after Phase 4 for spec approval before building
```

If the trigger includes a run mode (see RUN MODES), emit `FACTORY_MODE` here.

```
RALPH SAYS: [sentence 1] [sentence 2] [sentence 3]
DOMAIN:     [NONE | MEDICAL | FINANCIAL | LEGAL]
FACTORY_MODE: [fast | normal | advanced]
APP_NAME:   [distilled app name]
APP_DIR:    [../ralph-apps/prefix-slug]
AI_NATIVE:  [YES | NO]
SPEC:       [what it does · who uses it · core action]
```

**`AI_NATIVE` inference (no user flag):** set **YES** when the idea implies machine-generated or assisted outcomes users must review or act on — e.g. drafts (email, SMS, copy), personalization, summaries, recommendations, scheduling intelligence, chat/copilot flows, “AI writes…”, “automated follow-up content”, smart prioritization of outreach. Set **NO** for ideas where “AI” is branding only or the product is purely CRUD/reporting with no generative or assistive loop. When **YES**, Phases 3–7B must surface that loop in the UI (stubs allowed — see **AI-native product bar** below).

---

## PHASE 1 — PM

Emit this block only. No prose. Max 5 MVP features — cut ruthlessly (**fast** mode: max **3** MVP features).
JOBS must be verb phrases: "track invoices by client" not "invoicing".
If Phase 0 **`AI_NATIVE: YES`**, at least **one** MVP feature must name a **user-visible** AI-assisted moment (e.g. “review and send AI-drafted follow-up”, “accept or rewrite suggested message”) — not only “dashboard of leads.”

Before emitting the final Phase 1 block, do this internally:
- Generate the obvious product interpretation.
- Generate **5 adjacent-but-credible product moves**.
- Choose **1–2** that most improve the JTBD without bloating scope.
- Reflect those promoted moves inside MVP, PROBLEM framing, or JOBS.

For future-facing prompts, at least one promoted move should reflect changed system initiative, better planning intelligence, or compression of complexity.

```
PRODUCT: [name]
PROBLEM: [1 sentence]
USER:    [job title + context]
JOBS:    [verb + object] | [verb + object] | [verb + object]
ADJACENT MOVES CONSIDERED: [move 1] / [move 2] / [move 3] / [move 4] / [move 5]
PROMOTED MOVES: [move a] / [move b or NONE]
MVP:     [f1] / [f2] / [f3] / [f4] / [f5]
CUT:     [thing 1] / [thing 2]
WIN:     [1 measurable outcome]
```

---

## PHASE 2 — ARCHITECT

Emit this block only. No prose. No explanation before or after.
Define the full-stack shape.

**fast** mode: at most **2** entities. Prefer **CHARTS: NO** unless the product is explicitly chart-centric.

Before emitting the block, do this internally:
- Include one **anticipatory** behavior, one **recommendation** behavior, and one **compression** behavior where appropriate for the domain.
- Keep them scoped to the locked stack.
- If `AI_NATIVE: YES`, ensure the architecture supports visible AI flows even when model behavior is stubbed.

```
STACK:    React 18 + Vite + Tailwind | Express + Node | SQLite (libsql) | auth: none
ENTITIES:
  [Entity]: fields=[id, name, status, amount, date, ...], records=20
  [Entity]: fields=[...], records=15
ROUTES:
  GET    /api/[entity]        list + filter
  GET    /api/[entity]/:id    detail
  POST   /api/[entity]        create
  PUT    /api/[entity]/:id    update
  DELETE /api/[entity]/:id    delete
  POST   /api/[entity]/[ai-action]   stubbed AI action with real state transition   (when AI_NATIVE: YES or materially useful)
CHARTS: [YES — area/bar/line/pie · time-series/categorical | NO]
SYSTEM BEHAVIORS: anticipatory=[one] | recommendation=[one] | compression=[one]
OUTPUT: see references/STACK.md for folder structure
```

---

## PHASE 3 — DESIGN

Read **`references/DESIGN_SYSTEM.md`** and **`references/ANTI_GENERIC_UI.md`**. Inherit the Ralph Design System for **tokens, components, and §9 primitives**. Inherit Anti-Generic for **layout, metaphor, tension, and rejection of template dashboards** (that file wins on structure when in conflict).
Override palette/accent only if the domain genuinely calls for it.
If Phase 0 **`AI_NATIVE: YES`**: **`AI-UX`** must be **`AMBIENT`** or **`CONVERSATIONAL`** (never **`NONE`**). Prefer **`CONVERSATIONAL`** when the user reads generated text. Name screens so the **first** listed screen is where the AI-assisted JTBD happens (draft/review/queue), not only a metrics overview.

**Theme-default rule:** Choose **light-default** for productivity, planning, note-taking, writing, coordination, and consumer utility products unless there is a strong reason dark better supports the metaphor. Choose **dark-default** for control-room, media, security, trading, monitoring, or cinematic environments. State the reason briefly.

**Polish rule:** Design as if a world-class product design team obsessed over flow, hierarchy, and emotional friction removal. Broken-feeling transitions, generic whitespace, or empty heroic emptiness are invalid.

**Forbidden as the only story:** `LAYOUT: dashboard-grid` or `hero: stat-grid` **without** an explicit asymmetric `LAYOUT_SPEC`, `VISUAL_TENSION`, and per-screen `idea:` — generic “rounded cards + KPI row” is a tripwire.

```
DESIGN_INTENT:
- Core metaphor: [one line]
- Visually distinct because: [one line]
- Not generic because: [one line]

SIGNATURE_MOMENT: [one defining interaction or visual — specific]

VISUAL_TENSION: [where asymmetry, overlap, depth, or mixed density appears — specific]

LAYOUT_SPEC:
- Regions: [roles + rough sizing or flex behavior]
- Fixed vs fluid: [what pins, what grows]
- Mobile (<640px): [collapse order]

INSPIRATION: [3 named UIs/products/domains — not “modern SaaS” / “clean dashboard”]

WEIRD_HOOK: [one implementable memorable element for this app — see ANTI_GENERIC_UI §8]

PALETTE:  Ralph Design System [OR] OVERRIDE: accent=[hex] accent2=[hex] because [reason]
THEME:    [light-default | dark-default] because [reason]
LAYOUT:   [structural label — must match LAYOUT_SPEC; e.g. sidebar-nav, split-inspector, single-column-stacked, canvas+rail]
AI-UX:    [AMBIENT | CONVERSATIONAL | NONE]   ← if AI_NATIVE: YES, forbidden: NONE
SCREENS (max 4, in build priority order; **fast** mode: max **3**):
  1. [Name] | slug:[slug] | idea:[screen’s dominant concept] | primary-action:[1 thing] | mutation-surface:[modal|inline|drawer] | hero:[stat-grid|list|chart|form|none] | components:[list]
  2. [Name] | slug:[slug] | idea:[...] | primary-action:[1 thing] | mutation-surface:[modal|inline|drawer] | components:[list]
  3. [Name] | slug:[slug] | idea:[...] | ...
EMPTY STATES: one per data screen
RESPONSIVE:   640px breakpoint
INTERACTION:  each screen defines at least one mutation; include one bulk action and one quick action across the app
AI SURFACES:  if AI_NATIVE: YES — list ≥2 UI areas (e.g. "Draft panel+actions" / "Review queue row actions" / "Schedule suggestion chip" / "Today plan accept-reject strip") mapping to MVP; stubs OK
AI ACTIONS:   at least one actionable AI flow (suggestion/copy/draft) with accept + reject or edit-and-commit; if AI_NATIVE: YES, at least one flow must change persisted or queued record state via API or client state wired to API-shaped data
```

Slugs are lowercase-hyphenated. Phases 7A/7B use slugs exactly as defined here.

---

## PHASE 4 — SPEC

Write `spec/spec.md` to the project root. This is the durable source of truth.
It persists in the repo. Future agents, future runs, future developers read this.
Write it as a file action — not as a code block in chat.

**advanced** mode only: after `## Success Metric`, add `## Assumptions` and `## Risks & edge cases` with concrete bullets (stack, auth, deployment, abuse, data loss, etc.).
**normal** and **fast**: omit those two sections — do not leave placeholder headings.

```
FILE: spec/spec.md

# [Product Name] — Specification

## What It Does
[SPEC from Phase 0]

## Problem
[PROBLEM from Phase 1]

## User
[USER from Phase 1]

## Jobs To Be Done
[JOBS from Phase 1, one per line]

## Adjacent Moves Considered
[ADJACENT MOVES CONSIDERED from Phase 1]

## Promoted Moves
[PROMOTED MOVES from Phase 1]

## MVP Scope
[MVP features from Phase 1]

## Out of Scope
[CUT from Phase 1]

## Success Metric
[WIN from Phase 1]

## Data Model
[ENTITIES from Phase 2 — all fields, record counts]

## API Contract
[ROUTES from Phase 2 — method, path, purpose, request/response shape]

## System Behaviors
[SYSTEM BEHAVIORS from Phase 2]

## Screens
[SCREENS from Phase 3 — name, slug, purpose, primary action, components]

## Interaction Contract
- Per-screen mutation surfaces and actions (from Phase 3)
- At least one full UI CRUD flow (create/edit/delete) for a core entity
- Feedback states for mutation flows (loading, success, error)
- AI action flow with accept/reject behavior
- If AI_NATIVE (Phase 0): document the ≥2 AI surfaces from Phase 3 and the primary JTBD screen; forbid stats-only MVP

## Design
Palette: [PALETTE]  Theme: [THEME]  Layout: [LAYOUT]  Responsive: 640px
Anti-generic (from Phase 3 — paste or summarize): DESIGN_INTENT, SIGNATURE_MOMENT, VISUAL_TENSION, LAYOUT_SPEC, INSPIRATION (3), WEIRD_HOOK; per-screen `idea:` echoed from ## Screens
Factory mode: [fast | normal | advanced]

## Stack
React 18 + Vite + Tailwind · Express + Node · SQLite (libsql)

## Domain
[DOMAIN from Phase 0]
```

**Review gate (non-negotiable):** Pause after Phase 4 **only** when `--review` was present in the Phase 0 trigger. If `--review` is absent — including `/factory --fast`, `/factory --advanced`, and `/factory --mode fast|normal|advanced` **without** `--review` — do **not** pause, do **not** ask the user to review the spec before continuing, do **not** emit the `SPEC READY` / wait-for-`/approve` sequence, and continue to **Phase 5 immediately** once `spec/spec.md` is on disk.

If --review was set in Phase 0, pause here. Do not run Phase 5.

Emit exactly this **(only when `--review` was set; otherwise skip this block entirely)**:

```
SPEC READY

`spec/spec.md` has been written to your project root.

Read it now. Check that the entities, routes, screens, promoted moves, and theme default match your intent.
Edit `spec/spec.md` directly if anything needs changing — no special format required.

When ready:
  /approve        continue to tasks, tests, and build
  /respec [idea]  rewrite the spec from a revised idea
```

Wait for /approve before proceeding to Phase 5 (**review runs only**).
If the run was not flagged --review, continue to Phase 5 immediately (no emit, no wait).

---

## PHASE 5 — TASKS

Write `spec/tasks.md` to the project root as a file action.
Tasks are the build contract. Phases 7A and 7B implement them in order.
Scale task lines to the actual scope:

- Keep IDs sequential (`T01` ... `TNN`) with no gaps
- Include one schema task, one seed task, and one routes task per entity
- Include one page task per in-scope screen
- Target 12–18 tasks for 2-entity/3-screen builds.
- For each extra entity beyond 2, add 3 tasks (schema, seed, routes).
- For each extra screen beyond 3, add 1 task.
- Never compress two files into one task.

```
FILE: spec/tasks.md

# [Product Name] — Task List

## Server (Phase 7A)
- [ ] T01 bootstrap folders (`spec/`, `tests/`, `server/`, `client/`, `.ralph/briefs/`, `.ralph/memory/episodes/`; optional: `.ralph/logs/`, `docs/`), constitution.md, package.json (root), server/package.json, .gitignore, .nvmrc
- [ ] T02 SQLite schema for [Entity1] — fields: [list]
- [ ] T03 SQLite schema for [Entity2] — fields: [list] (repeat per entity as needed)
- [ ] T04 Seed [Entity1] — mixed statuses, realistic values, mode-based record count
- [ ] T05 Seed [Entity2] (repeat per entity as needed)
- [ ] T06 [Entity1] routes: GET list+filter, GET :id, POST, PUT, DELETE
- [ ] T07 [Entity2] routes: GET list+filter, GET :id, POST, PUT, DELETE (repeat per entity as needed)
- [ ] T08 AI action routes / state-transition handlers (when AI_NATIVE: YES or materially useful)
- [ ] T09 server/index.js — helmet, CORS, route mounts, seed on start, error handler

## Client (Phase 7B)
- [ ] T10 client/package.json, vite.config.js, tailwind.config.js, postcss.config.js
- [ ] T11 client/index.html — Vite entry point
- [ ] T12 client/src/index.css — Ralph Design System vars and reset
- [ ] T13 client/src/lib/api.js — fetch helpers per route
- [ ] T14 Shared components: [list from Phase 3]
- [ ] T15 Forms + mutation UX primitives (modal/drawer/inline editor), validation errors, success/error toasts
- [ ] T16 AI surfaces + accept/reject or edit-and-commit controls from Phase 3
- [ ] T17 [Screen 1] page — [primary action] + mutation flow
- [ ] T18 [Screen 2] page — [primary action] + mutation flow
- [ ] T1N [Screen N] page — [primary action] + mutation flow (insert one task per additional screen; keep numbering contiguous with no gaps)
- [ ] T[n-1] App.jsx — React Router, nav, ModeToggle (where n is the final task number after all screen-page tasks are listed)
- [ ] T[n] main.jsx — entry, sets chosen theme before render

## Done when
All tasks checked. npm run dev starts clean. All routes respond. Theme default matches spec. Dark/light toggle works.
```

Task count follows the scope formula above. Each task maps to one file or one behavior.
After writing `spec/tasks.md`, verify each client page task uses the exact slug text from Phase 3 (no pluralization/reformatting).
Also verify tasks include mutation UX implementation and at least one AI action flow with accept/reject.
If **`AI_NATIVE: YES`**, include explicit client tasks (under T14/T16 shared components and/or dedicated page tasks) for **each** AI surface named in Phase 3.

---

## PHASE 6 — TESTS

Write tests before Phase 7A writes any implementation code.
Write as file actions: tests/api.test.js and tests/seed.test.js.
See references/STACK.md for Vitest config.

tests/api.test.js — one describe block per entity:

- GET /api/[entity] returns array — count threshold follows mode seed minimum (**fast: >= 12**, **normal/advanced: >= 15**)
- GET /api/[entity]/:id returns correct shape with all fields
- POST /api/[entity] with valid body returns 201 and created record
- POST /api/[entity] missing required field returns 400
- PUT /api/[entity]/:id updates and reflects change
- DELETE /api/[entity]/:id returns 204 and record is gone
- AI action route returns a structured state transition or suggestion payload when in scope

tests/seed.test.js — one test per JOB:

- Derive JOBS from Phase 1 output if available; otherwise from `spec/spec.md` -> `## Jobs To Be Done`.
- Record count meets minimum
- Required fields present and non-null
- Status values within valid enum
- Date range spans at least 12 months
- One assertion per JOB tied to actual field values in the data
- Where promoted moves imply recommendation, compression, or AI review state, assert corresponding fields or seeded examples exist

Tests reference actual entity names, field names, and counts from spec artifacts (Phases 1-3 outputs).
They will fail until Phase 7A builds the server. That is correct.

**Verification:** both test files on disk; describes match `spec/spec.md` API and entities. No placeholder `expect(true).toBe(true)`.

---

## PHASE 7A — SERVER

Write each file as a separate file action, one at a time.
Do not emit files as code blocks in chat. Write them to disk.
Read references/STACK.md for templates before writing anything.
Check off each task from `spec/tasks.md` as it is completed.
After writing each file, immediately update `spec/tasks.md` by changing the matching task from `[ ]` to `[x]` before moving on.

Files to write, in this order:

1. constitution.md — see format below
2. package.json — root workspace, dev script with concurrently
3. server/package.json — type module, libsql + express + cors + helmet deps, engines node >= 22
4. .gitignore — node_modules, dist, .env, *.db, .ralph/logs
5. .nvmrc — single line: 22
6. server/db/schema.js — SQLite tables matching Phase 2 entities exactly
7. server/db/seed.js — mode-based counts (**normal/advanced: 15-25**, **fast: 12-18**) per entity, realistic, varied, no lorem ipsum
8. server/routes/[entity].js — one file per entity, all 5 CRUD routes
9. server/routes/[ai-action].js or inline entity AI-action handlers when specified
10. server/index.js — helmet, CORS, route mounts, global error handler last.
   Call initDb() before seedDb() — schema must exist before seed runs.
   Order: initDb() → seedDb() → app.listen()

After writing server/index.js:

- Run: npm install in project root and server/
- Run: npm run dev:server
- Confirm GET /api/health returns { status: "ok" }
- If it fails, debug and fix before proceeding to Phase 7B
- If it passes, proceed immediately

**Verification:** health response logged or stated in chat; if fail, fix before Phase 7B.

constitution.md format:

```
FILE: constitution.md

# [Product Name] — Constitution
_Last updated: [date] · Ralph Wiggum Loop v8.0_

## Stack
- Frontend: React 18 + Vite + Tailwind CSS
- Backend: Express + Node (ES modules)
- Database: SQLite via libsql (in-memory, seeded on startup)
- Testing: Vitest
- No other dependencies without explicit approval

## Code Principles
- One component per file. Named exports only.
- All colors via CSS custom properties. No hardcoded hex values.
- API routes validate required fields and return correct HTTP status codes.
- Parameterised queries only. No string SQL concatenation.
- React Router paths match `spec/spec.md` slugs exactly.
- AI action routes may stub generation, but state transitions and response shapes must be real.

## Data Principles
- Seed data is realistic. No lorem ipsum, no "Item 1", no "$0.00".
- Status fields use the enum defined in `spec/spec.md`.
- Dates span at least 12 months. Mix of past and future.
- Recommendation / review / queue state fields required by spec must be seeded meaningfully.
- CORS defaults to localhost:5173 only. Expand allowed origins before staging/production deploys.

## Design Principles
- Ralph Design System (Eclipse Edition) + Anti-Generic UI contract. Read references/DESIGN_SYSTEM.md and references/ANTI_GENERIC_UI.md before writing UI; implement Phase 3 LAYOUT_SPEC, ideas, and WEIRD_HOOK in layout — not token-violating card grids alone.
- Theme default follows `spec/spec.md` **## Design**. ModeToggle always present in nav.
- Every data screen has an empty state.
- Every list has a live search filter.
- Responsive at 640px. Nothing overflows.
- AI-native apps surface real accept/reject or edit-and-commit controls above the fold on the primary JTBD screen.

## What Never Ships
- Hardcoded API keys or secrets
- String SQL concatenation
- Inline hex color values
- Components over 150 lines
```

Seed data rules:

- **normal** / **advanced**: 15-25 records per entity. **fast**: 12-18 per entity.
- Realistic names, varied amounts, mixed statuses.
- Dates span last 18 months. Include 1-2 edge cases per entity.
- MEDICAL/FINANCIAL/LEGAL: fictional names only.
- If AI/recommendation behavior is in scope, include believable seeded review states, confidence flags, suggested actions, or queue examples.

DEFERRED — only for OAuth, payments, email, file uploads, websockets:

```js
// [RALPH DEFERRED] "I put it in my pocket but my pocket has a hole." — Ralph W.
// TODO: [feature]
// WHY:  [1 sentence]
// HOW:  [1 sentence]
```

Max 3 per run. If more are needed, stop and emit:
SCOPE SPLIT: /factory [Part A — core CRUD] then /factory [Part B — auth + integrations]

If the response is getting very long or a file would be left partial, finish the current
file cleanly (close brackets/tags), then emit this message exactly — never stop silently:

```
Ralph got sleepy mid-build. The computer needed a nap.

Last completed:  [filename just written, e.g. server/routes/invoices.js]
Next up:         [next task from spec/tasks.md, e.g. T09 server/index.js]
Remaining tasks: [list of unchecked task IDs from spec/tasks.md]

Type /continue to wake Ralph up and resume from here.
```

---

## PHASE 7B — CLIENT

Server is confirmed running before this phase starts.
Write each file as a separate file action, one at a time.
**Mandatory:** Re-load **`references/ANTI_GENERIC_UI.md`** and **`references/DESIGN_SYSTEM.md`** from disk if they are not already in the current message context (same directory as this skill). Then implement Phase 3’s `LAYOUT_SPEC`, `WEIRD_HOOK`, tension, and per-screen `idea:` in JSX — **not** a fresh generic dashboard.

**Order-of-operations for layout:** After `index.css`, implement **App shell** (`App.jsx` + first page) so the **asymmetric / mixed-density** structure exists **before** filling individual chart/table components. If the shell looks like the **Rejected shell** in **UI CONTRACT — ANTI-GENERIC**, stop and revise the shell to match `LAYOUT_SPEC` before writing other pages.
Check off each client task from `spec/tasks.md` as it is completed.
After writing each file, immediately update `spec/tasks.md` by changing the matching task from `[ ]` to `[x]` before moving on.

Files to write, in this order:

1. client/package.json — react, react-dom, react-router-dom, recharts, iconoir-react, vite, tailwind, vitest
2. client/index.html — Vite entry, mounts #root
3. client/vite.config.js — proxy /api to :3001
4. client/vitest.config.js
5. client/tailwind.config.js
6. client/postcss.config.js
7. client/src/index.css — full Ralph Design System CSS from STACK.md, paste verbatim
8. client/src/lib/api.js — typed fetch helpers per route
9. client/src/components/[component].jsx — one file per component from Phase 3
10. client/src/pages/[page].jsx — one file per screen, Phase 3 priority order
11. client/src/App.jsx — import Navigate from react-router-dom, React Router, nav, ModeToggle
12. client/src/main.jsx — sets chosen theme before render
13. README.md at project root (not inside `client/`) — write after all client files; setup in 3 commands

Code rules:

- All colors via var(--token). Zero hardcoded hex.
- React Router paths use Phase 3 slugs exactly.
- Every data screen has a working empty state.
- Every list has a live search filter via useState.
- At least one core entity has full UI CRUD (create, edit, delete) wired to API helpers.
- Include one bulk action and one quick action in the client UX.
- Mutation flows must show loading, success, and error feedback (for example: pending button state + toast/error inline).
- AI-native behavior must be actionable, not decorative: at least one accept/reject (or edit-and-commit) AI suggestion flow tied to record data. When **`AI_NATIVE: YES`**, implement **all** Phase 3 **AI SURFACES** with real controls; use `setTimeout` + fake text, seed-backed copy, or simple POST fields to simulate generation — **never** only a static subtitle claiming “AI powered.”
- Use **SkeletonLoader** / **StreamingText** / **TypingIndicator** per Phase 3 **AI-UX** and `references/DESIGN_SYSTEM.md` §9.
- **Composition:** Honor `spec/spec.md` **## Design** anti-generic fields and Phase 3 `LAYOUT_SPEC` / `WEIRD_HOOK`. Prefer asymmetry, mixed density, or a distinct shell over an even N-column card grid as the primary layout.
- Stat cards compute deltas from real data. Never hardcode delta values.
- ModeToggle in nav. Theme default comes from Phase 3 / `spec/spec.md` and must be set on `<html>` before paint.
- Responsive at 640px. aria-label on all icon-only buttons.
- **No Tailwind “template kit” defaults as the whole UI:** avoid `bg-gray-50` / `bg-slate-50` page + repeated `bg-white rounded-2xl shadow` cards unless Phase 3 explicitly documented that treatment **and** paired it with `WEIRD_HOOK` + tension. Prefer `var(--bg)`, `var(--surface)`, `var(--text)`, editorial type scale.
- For future-facing prompts, the UX must visibly change how work gets planned, triaged, or executed. Do not settle for futuristic copy over ordinary mechanics.

If the response is getting very long before all screens are complete, write the current file cleanly
and emit this message exactly — never stop silently:

```
Ralph got sleepy mid-build. The computer needed a nap.

Last completed:  [filename just written, e.g. client/src/pages/Dashboard.jsx]
Next up:         [next task from spec/tasks.md, e.g. T18 Screen 3 page]
Remaining tasks: [list of unchecked task IDs from spec/tasks.md]

Type /continue to wake Ralph up and resume from here.
```

This is not a DEFERRED. Complete higher-priority screens first per Phase 3 order.

/continue trigger — if the user types /continue after a truncated phase, read `spec/tasks.md`,
identify the last completed `[x]` task, and resume writing file actions from the next unchecked `[ ]` task.

---

## PHASE 8 — SECURITY

Always emitted. Never skipped.

**advanced** mode: FINDINGS must include **at least four** lines (any mix of HIGH/MED/LOW/INFO — "no issues" still list INFO items such as auth none, localhost-only, dependency posture).
Always include this INFO finding when applicable: `INFO CORS — restricted to localhost:5173. Expand origin list before non-local deploys.`

```
DOMAIN:     [confirm guardrail]
SURFACE:    [API routes exposed · data sensitivity · auth status]
FINDINGS:
  [HIGH|MED|LOW|INFO] [category] — [description] → [fix]
AUTO-FIXED: [inline fixes applied, or "none"]
VERDICT:    SHIP | SHIP_WITH_NOTES | DO_NOT_SHIP
```

Auto-apply: CORS to localhost only, helmet(), input validation on POST/PUT,
parameterised queries throughout, no secrets in code.

**Verification:** emit the full template block; VERDICT must be one of SHIP | SHIP_WITH_NOTES | DO_NOT_SHIP.

---

## FINAL OUTPUT

```
FACTORY COMPLETE

  App:        [name]
  Stack:      React + Vite · Express · libsql
  Design:     Ralph Design System + Anti-Generic composition · Theme-aware
  Spec:       spec/spec.md
  Tasks:      spec/tasks.md · [n] tasks checked
  Tests:      written before code
  Server:     Phase 7A complete · health check passed
  Screens:    [n]   API routes: [n]
  Records:    [n] across [n] entities
  Security:   [verdict]
  Mode:       [fast | normal | advanced]
  Theme:      [light-default | dark-default]
  Tests run:  [if executed: pass/fail summary | if not: "not executed — run locally"]
  Tokens:     [if reported by environment: input X · output Y · total Z | if not: "not reported — check your IDE token counter"]


  npm run install:all && npm run dev
  → client:  http://localhost:5173
  → server:  http://localhost:3001
  → tests:   npm run dev:server then npm test
```

---

## PHASE TRIGGERS

Partial triggers re-run a subset of phases. Read the **COMMAND MAP** prerequisites before acting.

---

### /approve

Continues a paused `--review` run from Phase 5 onward.
Only valid after `spec/spec.md` has been written and the factory is waiting.
If there is no prior "SPEC READY" pause in context and no review flag recoverable from `spec/spec.md`, do not proceed — emit: `No active review session found. Did you mean /from-spec?`

What it does:

- Reads the current `spec/spec.md` — including any edits the user made
- Continues from Phase 5 (tasks), then Phase 6 (tests), then Phase 7A, 7B, 8
- Fully autonomous from this point — no further pauses

If `spec/spec.md` was edited before /approve, the task list and tests will reflect the edits.
If `spec/spec.md` was not edited, the build proceeds against the original spec.

Emit at the start:

```
APPROVED — continuing from Phase 5
Reading spec/spec.md
Recovering FACTORY_MODE from spec/spec.md (default to normal if missing)
```

---

### /from-spec

Runs Phases 5 through 8 only. Skips intake, PM, architect, design, and writing `spec/spec.md`.

**Requires:** `spec/spec.md` in project root (complete enough to derive entities, routes, screens).

Use when: you authored or imported `spec/spec.md` yourself, or you are re-running the factory after a manual spec rewrite without repeating Phases 0–4.

What it does:

- Reads `spec/spec.md` only — does not overwrite it in Phase 4
- Phase 5 — writes `spec/tasks.md` from `spec/spec.md`
- Phase 6 — writes tests from spec + tasks
- Phases 7A, 7B, 8 — same as full factory
- Phase 7B: read `references/DESIGN_SYSTEM.md` and `references/ANTI_GENERIC_UI.md`; implement `spec/spec.md` **## Design** (including anti-generic fields when present). If **## Design** lacks those fields, still avoid the generic SaaS-only tripwire and align layout with **## Screens** explicitly.

Emit at the start:

```
FROM-SPEC: reading spec/spec.md
SCOPE: Phases 5–8 — spec/spec.md unchanged
```

---

### /continue

Resumes Phase 7A or 7B after the "Ralph got sleepy" truncation message.

What it does:

- Reads `spec/tasks.md`; finds last completed `[x]` task (fallback: sleepy message "Last completed" line if no `[x]` exists yet)
- Continues file actions from the next unchecked task through the end of 7A or 7B
- If resuming Phase 7B, confirm `GET /api/health` returns `{ "status": "ok" }` before writing client files
- After 7B completes, run Phase 8 if it was not already run for this build

Emit at the start:

```
CONTINUE: resuming from spec/tasks.md
```

---

### /tests

Runs **Phase 6** only.

**Requires:** `spec/spec.md` and `spec/tasks.md`.

Use when: you changed the spec or tasks and need tests realigned without regenerating server/client yet.

What it does:

- Rewrites tests/api.test.js and tests/seed.test.js per Phase 6 rules
- Does not run Phase 7A/7B

Emit at the end:

```
TESTS REGENERATED
  tests/api.test.js · tests/seed.test.js
  Run npm run dev:server then npm test (API tests need a live server)
```

---

### /security

Runs **Phase 8** only.

**Requires:** `spec/spec.md`, `constitution.md`, and the current codebase (server + client as applicable).

Use when: you want a fresh OWASP-style pass after manual edits, dependency bumps, or without a full /rebuild.

What it does:

- Reads `spec/spec.md` and `constitution.md`
- Audits the tree; emits the Phase 8 block (FINDINGS, AUTO-FIXED, VERDICT)
- Apply obvious safe fixes inline when they match Phase 8 auto-apply rules

Emit at the start:

```
SECURITY PASS: reading spec/spec.md, constitution.md, codebase
```

---

### /retask

Re-runs Phase 5 only.
Use when: you have manually edited `spec/spec.md` and want `spec/tasks.md` regenerated to match without triggering a build.

**Requires:** `spec/spec.md`.

What it does:

- Reads the current `spec/spec.md`
- Rewrites `spec/tasks.md` to reflect the updated spec
- Does not touch any code

Emit at the end:

```
RETASK COMPLETE
  spec/tasks.md updated from spec/spec.md
  Run /rebuild to build against the new task list
```

---

### /respec

Re-runs Phases 1 through 5.
Use when: you want to expand scope, add an entity, change the MVP definition, or pivot the product direction.

**Requires:** user provides the revised idea in the message (same as /factory).

What it does:

- Re-runs Phase 1 (PM) — new product scope from the updated idea
- Re-runs Phase 2 (Architect) — revised entities and routes
- Re-runs Phase 3 (Design) — revised screens
- Re-runs Phase 4 — overwrites `spec/spec.md` with the new source of truth
- Re-runs Phase 5 — overwrites `spec/tasks.md` to match the new spec
- Does not touch any code — no build phase runs automatically
- User reviews the new `spec/spec.md` before running /rebuild

Emit at the end:

```
RESPEC COMPLETE
  New spec/spec.md written
  New spec/tasks.md written
  No code has changed
  Tests may now be stale: run /tests to regenerate tests/api.test.js and tests/seed.test.js
  Review spec/spec.md then run /rebuild to build against the new spec
```

---

### /rebuild

Re-runs Phases 7A and 7B against the existing `spec/spec.md` and `spec/tasks.md`.
Use when: the last build truncated, you want a clean rebuild, or you made manual spec edits and want the code regenerated to match.

**Requires:** `spec/spec.md`, `spec/tasks.md`.

What it does:

- Reads `spec/spec.md` and `spec/tasks.md` as the build contract — does not change them
- Re-runs Phase 7A (server) — rewrites all server files
- Confirms server health check before proceeding
- Re-runs Phase 7B (client) — rewrites all client files
- Re-runs Phase 8 (security) on the new build

Emit at the start:

```
REBUILD: reading spec/spec.md and spec/tasks.md
SCOPE: full rebuild — server and client
```

---

### /redesign

Re-runs Phase 3 and Phase 7B only.

**Requires:** `spec/spec.md`, `constitution.md`.

Use when: the server is working but you want a different layout, screens, palette, or component structure.

What it does:

- Reads existing `spec/spec.md` for entities, routes, and screen slugs
- Reads existing constitution.md for stack and design constraints
- Re-runs Phase 3 — produces a new design block including **Anti-Generic** sections (`DESIGN_INTENT`, `LAYOUT_SPEC`, `INSPIRATION`, `WEIRD_HOOK`, per-screen `idea:`) plus palette, layout, screens, components, and theme default
- Updates **`spec/spec.md` only** in **## Screens** and **## Design** to match the new Phase 3 output (all other spec sections unchanged) so the spec stays aligned with the rebuilt client
- Re-runs Phase 7B — rewrites the entire client against the new design
- Does not touch the server, `spec/tasks.md`, tests, or constitution.md (unless you choose to refresh constitution **## Design Principles** from the template — optional)

Emit at the start:

```
REDESIGN: reading spec/spec.md and constitution.md
SCOPE: client only — server unchanged
```

---

## RULES

| Rule             | Value                                                                                                                   |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Factory triggers | /factory · hey Ralph · build me an app · factory mode                                                                   |
| Chat banner      | First lines of every full factory run reply (`/factory`, `hey Ralph`, `build me an app`, `factory mode` only): boxed “Ship-it-Ralph · Software Factory” — no version, no mode in the banner |
| Skill install    | Bundle `SKILL.md` + `references/` in the same folder. **Recommended:** **`<workspace>/.agents/<skill-name>/`**. Optional: **`.github/`**; optional global: **`~/.agents/skills/<skill-name>/`** — never a lone **`~/.agents/SKILL.md`**. Paths are relative to the folder that contains `SKILL.md`. |
| App bootstrap    | New factory runs create `../ralph-apps/[geeky-prefix]-[idea-slug]` and use it as project root for this run              |
| Trigger matching | Case-insensitive for command phrases (e.g., `hey Ralph`, `Hey ralph`)                                                   |
| Run modes        | `--mode fast\|normal\|advanced` · `--fast` · `--advanced` — all phases still run; `fast` caps scope/seed; `advanced` adds spec + security depth |
| --review flag    | /factory --review [idea] — pauses after spec/spec.md for user approval                                                  |
| /approve         | Resumes a --review run from Phase 5. Reads any edits made to spec/spec.md.                                              |
| Partial triggers | /from-spec · /continue · /tests · /security · /retask · /respec · /rebuild · /redesign                                  |
| Phase order      | 0-1-2-3-4-5-6-7A-7B-8. Never skip. Never reorder.                                                                       |
| File writes      | Each file is a separate file action. Not a chat code block.                                                             |
| 7A gate          | Confirm server health check passes before starting 7B.                                                                  |
| /continue        | After the Ralph sleepy message, resumes from the next unchecked `[ ]` task in spec/tasks.md (after last `[x]`).        |
| Spec             | spec/spec.md written in Phase 4. Source of truth.                                                                        |
| Tasks            | spec/tasks.md written in Phase 5. Phases 7A/7B implement in order.                                                       |
| Interactivity    | Read-only dashboard output is non-compliant. Require mutation flows (forms/edit/delete), feedback states, and one actionable AI accept/reject flow. If Phase 0 **`AI_NATIVE: YES`**, also require ≥2 visible AI surfaces and **AI-UX ≠ NONE**. |
| PM quality       | Greenfield runs must include adjacent-but-credible PM thinking and promote 1–2 such moves when valuable.               |
| Theme default    | Chosen in Phase 3 based on product type. Productivity/planning defaults to light unless justified otherwise.            |
| Tests            | Written in Phase 6 before Phase 7A. Expected to fail until built.                                                       |
| Constitution     | First file written in Phase 7A. Source-of-truth baseline; `/rebuild` may regenerate it from current spec/contracts.     |
| Stack            | React + Vite + Tailwind · Express + Node · libsql                                                                       |
| Design           | Ralph Design System + Anti-Generic UI — read references/DESIGN_SYSTEM.md and references/ANTI_GENERIC_UI.md before Phase 3; same pair before Phase 7B |
| Colors           | Zero hardcoded hex. Every color is var(--token).                                                                        |
| Ralph            | Phase 0 restate and DEFERRED tags only. Nowhere else.                                                                   |
| Phases 6 and 8   | Always emitted. Never skipped. Never merged.                                                                            |

---

*v2.0.0 · Swami Chandrasekaran*

---
> Source: [swamichandra/ship-it-ralph](https://github.com/swamichandra/ship-it-ralph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
