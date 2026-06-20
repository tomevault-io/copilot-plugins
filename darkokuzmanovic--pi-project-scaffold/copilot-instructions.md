## pi-project-scaffold

> Use when scaffolding, backfilling, or upgrading a `.pi/project/` template. Triggers on 'scaffold pi-project', 'init pi-project', 'set up .pi/project', legacy '.pi-project' wording, and natural fresh-project prompts like 'create me a new project', 'scaffold an app', 'plan my new CLI', or 'grill me on a new project'. Classifies risk as A temporary/learning, B architecture risk, C too much scope, or D not sure.


# Pi-Project Scaffold

Interview the user about a new project, identify the main way the project could go wrong, then write the risk-appropriate subset of the `.pi/project/` v0.4.1 template in one batch. The user supplies thought; you handle file mechanics. **Right-sizing is the point** — small projects should NOT get the full scaffold. See `## Risk classification` below.

**This skill is self-contained.** Do not invoke brainstorming or writing-plans as a prerequisite. The interview below replaces them — it covers vision, scope, constraints, milestones, and decisions in one flow.

Templates live in `templates/` next to this file. Each `.tmpl` file uses HTML comments `<!-- PLACEHOLDER: description -->` to mark substitution points. Pure-literal files (VERSION, gotchas-link.md, watch.md, polish.md) are copied verbatim.

## When to use

**Explicit triggers** (always engage): user says one of the literal trigger phrases in the description above.

**Implicit triggers** (engage by default, but acknowledge): user asks to *create a project and write a plan*, *scaffold a new app/tool*, or *grill them on a new project*. CWD has no `.pi/project/` folder. The user's request is forward-looking project work, not a one-shot script. In this case, open with: "Before I write a plan, I want to use the `pi-project-scaffold` skill — it'll grill you on vision, scope, and milestones, then write a `.pi/project/` charter you can hand off to a cheap executor model. Sound right?" Wait for confirmation, then proceed with the interview. If the user declines, fall back to writing a plain `PLAN.md`.

**Past misses to learn from:** the `claude-export` project (May 2026) was a textbook implicit-trigger case ("Create me ~/code/claude-export and write me a plan for the app. Grill me.") but the skill didn't engage; the executor wrote ad-hoc uppercase `CHARTER.md`/`PLAN.md` without the scaffold's discipline. The user re-derived the milestone-oracle pattern manually. Engage on implicit triggers to avoid this miss.
(See triggers above.) Required precondition for either trigger path: CWD has **no** `.pi/project/` folder AND no legacy `.pi-project/` folder. If legacy `.pi-project/` exists, refuse with the migration command (see `## When NOT to use`).

## When NOT to use

- Project already has `.pi/project/` — refuse and explain. Never overwrite.
- Project has legacy `.pi-project/` (from v0.3.x or earlier) — **refuse and print the migration command**, do not scaffold alongside, do not auto-migrate. Use this verbatim:
  > "Legacy `.pi-project/` detected. v0.4.1 writes to `.pi/project/`. Migrate first, then re-invoke this skill:\n>\n>     mkdir -p .pi && git mv .pi-project .pi/project && echo v0.4.1 > .pi/project/VERSION\n>     # then ensure `.gitignore` has `/.pi/*` and `!/.pi/project/`\n>\n> Re-run the trigger after the move completes."

  Do NOT execute the migration yourself — the user owns destructive git operations on their own project.
- One-shot scripts (<1 session of work) — recommend skipping the template entirely.
- User just wants to discuss the methodology, not scaffold. Point to `Dev/Projects/pi-project/template.md` in their Obsidian vault.

## Risk classification

Before the interview, ask the user what kind of problem the scaffold should protect against. In plain language: **what is most likely to make this project go wrong?** The answer determines which `.pi/project/` files get scaffolded — small projects should NOT get the full template.

**Ask exactly this question (no preamble, no narration):**

> **What is most likely to make this project go wrong?**
>
> - **(A) Temporary / learning project** — quick experiment, one-off script, or code that probably won't matter next week
> - **(B) Architecture risk** — hard bugs may come from background processes, async/concurrency, lifecycle hooks, global state, security, or integration with systems/code we don't own
> - **(C) Too much scope** — the idea is clear, but we may keep adding features and make it too big before shipping
> - **(D) Not sure yet** — unclear how big or complex this project will become

Use `ask_user` with these four options. Do NOT auto-decide. Do NOT collapse this into a yes/no — the four-way answer is load-bearing. If the user is confused, explain: “risk” means “the main way this project could go wrong,” then recommend an option from their project description and ask them to confirm.

### Route by answer

**(A) Temporary / learning project — STOP and exit the skill.**

Respond verbatim: *"Pi-project-scaffold is overkill for a temporary or learning-only project. Suggest `git init`, write code, ship it. I can write a one-paragraph README at the end if useful. Want me to exit this skill?"* Wait for confirmation. If user says yes, exit. If user says "actually I want the scaffold anyway," treat as (D).

**(B) Architecture risk — full scaffold.**

Generate ALL files: `charter.md`, `plan.md`, `decisions.md`, `watch.md`, `polish.md`, `gotchas-link.md`, `session-log/<date>-001.md`, `VERSION`, plus `AGENTS.md` at repo root. Run the full interview (all 9 steps). Run all 7 pre-write checkpoint warnings. This is the heaviest path — the methodology pays for itself when technical complexity grows.

**(C) Too much scope — skinny scaffold.**

Generate ONLY: `charter.md`, `plan.md`, `session-log/<date>-001.md`, `VERSION`, plus `AGENTS.md` at repo root.

SKIP: `decisions.md`, `watch.md`, `polish.md`, `gotchas-link.md`. The interview skips Step 9 (Initial decisions); steps 1-8 still apply.

When writing the first session log, add a note: *"Skipped decisions.md, watch.md, polish.md, gotchas-link.md at scaffold time per Risk profile C. Run `pi-project-scaffold` in upgrade mode if a (B)-class architecture risk emerges."*

**(D) Not sure yet — skinny scaffold + upgrade nag.**

Generate the same files as (C). Add the same skip-note to the session log, plus a stronger upgrade trigger in the `AGENTS.md` `## Outstanding triggers` section: *"Risk profile is (D) Not sure yet. If any of these land, upgrade to (B): module-level mutation, async ordering bugs, lifecycle clobbers, security review needed, integration with code we don't own."*

### Record the answer

Write the answer into `charter.md`'s `## Risk profile` section (see `templates/charter.md.tmpl`). Format: one letter plus a one-line rationale.

Example: `**(B)** — intercepts `globalThis.fetch` and runs inside Pi's session lifecycle; if undici resets the dispatcher mid-session, the wrapper silently bypasses.`

The rationale matters more than the letter — future-you reading this in 6 months wants to know *why* this risk shape, not just which option was picked.
## Retroactive mode

When the user asks to scaffold `.pi/project/` for a project whose initial milestone has already shipped (git history shows real work, no `.pi/project/` exists yet), apply these deviations to the standard flow:

1. **Skip the forward-looking interview.** Vision, scope, constraints, and shipped milestones are derivable from code, README, and `git log`. Ask the user only for what is genuinely future-facing: which milestone to pursue next, what is *out* of scope going forward, any constraints not visible in code.

2. **`plan.md`:** the shipped milestone goes under `## Done`, not `## Currently working on`. "Currently working on" gets the *next* milestone the user wants to pursue. See `templates/plan.md.tmpl` for the retroactive hint.

3. **`session-log/`:** the milestone line and Summary section both deviate from the literal template — see `templates/session-log.md.tmpl` for the retroactive variants. Structure Summary as TWO bullet groups: one for the shipped milestone's work (anchor to commits where useful), one for the scaffold session itself.

4. **`decisions.md`:** retroactively inferred decisions are honestly labeled, not faked. If "alternatives considered" was never an explicit comparison at the time of writing the code, write `**Inferred from code; alternatives not actively compared at decision time.**` instead of fabricating rejected options. Honest gaps beat fabricated rigor.

5. **Skip Step 7's "M0 must ship user-visible value" grill rule.** M0 has already shipped — its nature is settled in git history, not negotiable. Do not challenge the user about whether M0 counts as "real work."

**Fidelity caveat to surface to the user:** retroactive scaffolds work best *within the same session as the shipped work*, when alternatives and trade-offs are still in conversation memory. Weeks-old work means decisions are reconstructed, not recorded — surface this and let the user choose between "high-fidelity decisions they remember" and "best-effort reconstruction with caveats."

**Never overwrite:** the "When NOT to use" rule still binds. If `.pi/project/` already exists, refuse and explain. Retroactive mode is for projects that never had `.pi/project/`, not for re-scaffolding.

## Upgrade mode

When the user asks to upgrade a project scaffolded as Risk type (C) or (D) to full (B) coverage — typically because a (B)-class architecture risk has emerged mid-milestone — generate the missing files without re-running the interview.

**Trigger phrases:** "upgrade pi-project", "promote to full scaffold", "pi-project upgrade", "upgrade to risk B".

### Pre-flight checks (refuse if any fail)

1. `.pi/project/` exists. (Otherwise this is a fresh scaffold, not an upgrade.)
2. `.pi/project/VERSION` shows `v0.4.0` or later. (Pre-v0.4.0 layouts live at legacy `.pi-project/` and trigger the refuse-with-migration-command path above, not upgrade mode.)
3. `charter.md` has a `## Risk profile` section showing **(C)** or **(D)**, not **(B)**. (If already (B), the scaffold is already full; nothing to upgrade.)
4. At least one of `decisions.md` / `watch.md` / `polish.md` / `gotchas-link.md` is missing. (If all four exist, the scaffold is already full despite the risk-profile letter — ask the user what they actually want.)

If any check fails, refuse with a one-sentence explanation. Do NOT push forward.

### Generate missing files only

- `decisions.md` — write the template header plus a single seed entry:

  ```
  ## <date>-upgrade: Risk profile upgraded from <C|D> to B

  **Trigger:** <one-line reason from user, e.g., "M2 hit module-cache vs. /reload asymmetry; need decisions.md to track design responses.">

  **Caveat:** Earlier architectural decisions made under risk profile (C|D) were not formally recorded. Reconstruct from session logs and code where useful; label inferred decisions explicitly as `**Inferred from code; alternatives not actively compared at decision time.**`. Honest gaps beat fabricated rigor.
  ```
- `watch.md` — copy from `templates/watch.md` verbatim.
- `polish.md` — copy from `templates/polish.md` verbatim.
- `gotchas-link.md` — copy from `templates/gotchas-link.md` verbatim.

### Update `charter.md`

Replace the `## Risk profile` section's letter from (C)/(D) to **(B)**. Append a one-line upgrade note: `Upgraded from (<original>) on <date>: <reason>.`

### Update `AGENTS.md`

No structural change needed — the full close-out sequence is already present for all risk types. If `## Outstanding triggers` contains a (D)-mode upgrade nag, remove that bullet (it's done).

### Write a session-log entry

Use the next sequence number for today's date (e.g., `2026-05-23-002.md` if `001.md` exists). Heading: `Risk profile upgrade (C|D → B)`. Body: trigger reason, files added, notes about reconstructing decisions from code.

### Commit

Single commit: `chore: upgrade .pi/project risk profile (<C|D> → B)` with a body listing the added files.

### Don't

- Re-run the interview. (Charter, plan, milestones are already in place.)
- Backfill decisions.md from scratch by mining git history. Acknowledge the gap; let future decisions accrue normally.
- Bump `VERSION`. Upgrade mode is a risk-profile change, not a template-version change.
- Generate `AGENTS.md` unless missing entirely (which would be a bug — `AGENTS.md` ships in both (C) and (D) scaffolds).
## Pre-flight: environmental scan

Before asking anything, run a single batched scan of the cwd:

1. List top-level files. Look especially for:
   - **JS/TS:** `package.json`, `tsconfig.json`, `deno.json`, `bun.lock`
   - **Rust:** `Cargo.toml`
   - **Python:** `pyproject.toml`, `requirements.txt`, `setup.py`
   - **Go:** `go.mod`
   - **Dart/Flutter:** `pubspec.yaml`
   - **Godot:** `project.godot`
   - **Ruby:** `Gemfile`
   - **PHP:** `composer.json`
   - **Swift:** `Package.swift`, `*.xcodeproj`
   - **.NET:** `*.csproj`, `*.sln`, `*.fsproj`
   - **JVM:** `pom.xml`, `build.gradle`, `build.gradle.kts`
   - **Elixir:** `mix.exs`
   - **Zig:** `build.zig`
   - **Nix:** `flake.nix`
2. **Check git context.** Run `git rev-parse --show-toplevel 2>/dev/null`. Empty/non-zero exit → not in any repo (closing actions Case A: `git init`). Output equals CWD → CWD IS a repo root (Case B: commit scaffold into it). Output is an ancestor of CWD → CWD is *inside* a larger repo (Case C: ASK before committing; never `git init` inside an existing repo — that nests repos and breaks parent tooling). Also note current HEAD and whether the working tree is clean; a dirty tree in Case B means the scaffold commit would bundle unrelated changes — ask first.
3. Check for existing `AGENTS.md` at root **and** `.pi/project/`. If either exists, **stop** and report to the user.
4. Run `date +%Y-%m-%d` and remember the result — used for the session-log filename and the date field in `decisions.md`.

Use the scan to **prefill** answers in the interview. Examples:
- `package.json` with `"type": "module"` → "Looks like a Node ESM project. Confirm?"
- `project.godot` → "Godot project — what version, and what's the target platform?"
- `pyproject.toml` with `[tool.poetry]` → "Python with Poetry. Confirm target Python version?"
- `Package.swift` → "Swift Package Manager. Library or executable target?"

Surface what you found in your first message so the user can correct misreads.

## Interview protocol

Run these steps **in order**. One question at a time (or one tight cluster of related questions per turn). Don't batch — the user needs space to think.

### Step 1 — One-liner

Ask: *"In one sentence, what is this project?"*

**Grill:** If the answer is under 8 words or vague ("a tool", "an app", "a thing for X") — push back. Demand specificity: *what kind of tool? for whom? doing what?*

### Step 2 — Tech stack + primary platform

Confirm prefilled tech stack from env scan. Ask:
- *Primary platform?* (Linux / macOS / Windows / Android / iOS / Web / cross-platform)
- If multiple platforms named: *which is primary — which one ships first?*

**Grill:** "Cross-platform" alone is not an answer. Force a primary target.

### Step 3 — Vision paragraph

Build the vision from the one-liner via 2–3 follow-ups:
- *Why does this need to exist?* (the **why**, not the **what**)
- *Who's the user, and what changes for them when this works?*
- *What's the single most important thing it must get right?*

**Grill:** If user answers with features, redirect: *"That's the what; I'm asking why."* The vision is invariant; features change.

### Step 4 — Scope (in)

Ask: *"For the first milestone (M0), name the features that must ship. Cap: 3."*

**Grill:** If user lists more than 3, force prioritization: *"Pick the 3 most important. The rest go in M1+."* If user lists abstract goals ("be performant"), demand concrete features.

### Step 5 — Scope (out)

Ask: *"What's tempting to build but you're explicitly NOT building? Name at least 2."*

**Grill:** If user says "nothing comes to mind", probe: *"Multiplayer? Auth? Cloud sync? Plugin system? Mobile? Localization? Themes? Configurability?"* — they always have one.

### Step 6 — Constraints

Ask: *"What are the non-negotiables beyond tech stack?"* Surface examples if needed:
- Performance budgets (FPS, memory, bundle size, latency)
- Offline-first / online-only
- Data sovereignty (local-only, no telemetry)
- Accessibility level
- Forbidden dependencies (no npm packages above X size, no GPL, etc.)

**Grill:** "I want it fast" is vague — push for a number or a comparable benchmark.

### Step 7 — Milestone breakdown

Ask: *"Sketch the milestones M0..MN. One line each. Target 4–8."*

**Grill:**
- **M0 must ship user-visible value.** If user says M0 is "set up project" or "scaffolding", reject — scaffolding is *this* session. M0 is the first thing the user can actually use.
- Each milestone should be one-line specific. "Polish" is not a milestone. "Animate piece-drop with 200ms ease-out" is.
- If only 1–2 milestones named, probe for more. If 10+ named, probe for what's really M1 vs. parking lot.

### Step 8 — M0 sub-tasks

Ask: *"Break M0 into 3–7 concrete sub-tasks."*

**Grill:** Each task should be one anchored edit, one file creation, one test run, or one verification. Reject "implement game logic" — too big. Demand: which file, which function, which test.

### Step 9 — Initial decisions (risk-conditional)

Ask: *"During this interview, you made choices: framework, language, platform target, maybe specific libraries. Walk me through them — I want to record the rationale and what you considered."*

For each choice mentioned, capture:
- The decision
- The alternative(s) considered
- Why this won

These become the first entries in `decisions.md` (M0-a, M0-b, …).

**Skip this step entirely for risk profile (C) or (D).** Skinny scaffolds don't write `decisions.md`; the rationale conversation can still happen briefly, but record it in the first session log under `## Architectural notes` instead of in a dedicated decisions file.

## Pre-implementation checkpoint for generated projects

The generated `AGENTS.md` includes a pre-implementation gate for future milestones. For risk (B) projects, future agents must record any new milestone decisions in `decisions.md` before writing tests/code, or explicitly note "No new decisions for M{N}" in the session log. This is a template behavior change in v0.4.1; it prevents design choices from living only in conversation history.

## Pre-write checkpoint (soft warnings)

Before writing files, audit the interview against these markers. Each is a **signal of likely under-specification, not a hard block**. The verbosity of the warning is the point: it makes the user pause both at the section *and* at the rule itself.

For each marker that fires:

1. **Quote** the relevant interview answer back to the user.
2. **Explain** what's typically under-specified at this point and why it matters six months from now.
3. **Offer** 1–2 concrete probing questions to flesh it out.
4. **Ask:** *"Reconsider this section, or proceed anyway?"*

If the user proceeds, note their decision to proceed in the session-log's `Open questions for next session` section so the rough edge isn't forgotten.

### Markers

| Section | Warning fires when | Typical underlying gap |
|---|---|---|
| Vision | <2 sentences, OR features-not-purpose | User conflated *what* with *why* |
| Scope (in) | >3 features, OR abstract goals | Hidden M1+ work smuggled into M0 |
| Scope (out) | empty | Implicit unlimited scope |
| Constraints | only tech stack named | Missing non-functional requirements |
| Milestones | M0 is "setup", OR <3 milestones, OR >10 milestones | Wrong granularity for the project |
| M0 sub-tasks | <3 tasks, OR vague ("implement X") | Tasks too big to track |
| Initial decisions | none recorded | Choices made implicitly; provenance lost |

**Note on rigidity:** if you find yourself hitting the same warning across multiple projects and overriding it every time, that's a signal the rule itself is wrong, not the project. Capture that observation in the session log so the skill can evolve.

## File generation

Once warnings have been resolved or explicitly overridden, write all artifacts in **one batch**. State what you're about to write, then write it.

### Files to create (depends on risk profile)

The set of files depends on the risk classification recorded earlier. **(A) temporary/learning should have exited the skill before reaching this step — if you're here, the user must have changed their mind to (D).**

| Target path | Source template | Risk (B) | Risk (C) | Risk (D) | Substitution |
|---|---|---|---|---|---|
| `.pi/project/VERSION` | `templates/VERSION` | ✅ | ✅ | ✅ | none (literal copy) |
| `.pi/project/charter.md` | `templates/charter.md.tmpl` | ✅ | ✅ | ✅ | risk profile + steps 1, 3, 4, 5, 6, 7 |
| `.pi/project/plan.md` | `templates/plan.md.tmpl` | ✅ | ✅ | ✅ | steps 7 (M0+M1) and 8 |
| `.pi/project/session-log/<TODAY>-001.md` | `templates/session-log.md.tmpl` | ✅ | ✅ | ✅ | today's date + N decisions count + arch notes + skip-note if (C)/(D) |
| `AGENTS.md` (at repo root) | `templates/project-AGENTS.md.tmpl` | ✅ | ✅ | ✅ | project name + identity paragraph + (D) upgrade-nag if risk=D |
| `.pi/project/decisions.md` | `templates/decisions.md.tmpl` | ✅ | ❌ skip | ❌ skip | step 9 (one block per decision) |
| `.pi/project/watch.md` | `templates/watch.md` | ✅ | ❌ skip | ❌ skip | none |
| `.pi/project/polish.md` | `templates/polish.md` | ✅ | ❌ skip | ❌ skip | none |
| `.pi/project/gotchas-link.md` | `templates/gotchas-link.md` | ✅ | ❌ skip | ❌ skip | none |

### How substitution works

The `.tmpl` files contain HTML comments like `<!-- VISION: ... -->`. For each one:

1. Replace the **entire HTML comment** with the substituted content.
2. Some comments are on the same line as a markdown bullet (`- **Tech stack:** <!-- ... -->`) — replace the comment, keep the bullet.
3. If a section needs multiple entries (milestones, decisions, sub-tasks), expand the template's example rows to N rows.
4. Leave nothing un-substituted. Don't ship a file with a placeholder comment still in it.

For the session-log: use the **shell-anchored date** from pre-flight step 4 for both the filename (`YYYY-MM-DD-001.md`) and the `# Session YYYY-MM-DD — 001` header.

For `charter.md`'s `## Risk profile` section: write the letter chosen earlier plus a one-line rationale. Format: `**(X)** — <rationale from user's words during classification>`.

For risk (C) and (D) skinny scaffolds: the session log's `## Summary` should append a note: *"Risk profile (C|D) — skipped decisions.md, watch.md, polish.md, gotchas-link.md. Run `pi-project-scaffold` upgrade mode if a (B)-class architecture risk emerges."*

For risk (D) only: in `AGENTS.md`'s `## Outstanding triggers` section, replace `(None yet.)` with the upgrade-nag bullets listed in `## Risk classification`.

## Self-verify scaffold integrity

Before the closing actions, verify the batch wrote what you think it wrote. Admonitions in the anti-patterns section ("don't leave HTML comments un-substituted") are not enough — model output is fallible, so gate it deterministically.

Run **one** batched check:

1. **No unsubstituted placeholders.** Search every file just written for `<!-- ` patterns matching the substitution markers (`<!-- VISION:`, `<!-- M0_`, `<!-- DECISIONS:`, `<!-- PROJECT_NAME -->`, `<!-- IDENTITY:`, `<!-- TODAY`, `<!-- ONE_LINE_DECISION -->`, etc.). Any hit = the substitution failed; **fix that file and re-verify** before continuing. HTML comments that are *deliberate template guidance* (e.g. the close-out comment block in `session-log.md.tmpl` and the date-semantics comment block in `plan.md.tmpl`) should be deleted from the shipped output too — they were instructions for *you*, not the user.
2. **Session-log filename matches pre-flight date.** Confirm `.pi/project/session-log/<TODAY>-001.md` exists where `<TODAY>` is the shell-anchored date from pre-flight step 4. A mismatch means a stale date was substituted from training data; rewrite the file with the correct name and delete the wrong one.
3. **Decisions block expanded for risk (B).** `decisions.md` must contain at least one `## M0-a:` heading with substituted content (not the literal template block). If step 9 of the interview produced N decisions, expect N `## M0-{a,b,c…}` headings.
4. **M0 sub-tasks expanded.** `plan.md`'s `## Currently working on` section must contain the actual task list from interview step 8, not the template's three empty `- [ ]` bullets.
5. **Identity paragraph populated in `AGENTS.md`.** Should be ~2–3 sentences pulled from charter Vision + Constraints, not the literal `<!-- IDENTITY: -->` comment.
6. **Version bumped.** `.pi/project/VERSION`, `AGENTS.md` Project specifics, the first session log, and suggested scaffold commit messages must all agree on the same template version.

If any check fails, fix and re-verify. **Do not print the success tree in closing actions if the scaffold is incomplete** — silent partial scaffolds are worse than visible failures.

## Closing actions

After all files written:

1. **Print a tree** of what was created (use `find .pi/project -type f` plus `ls AGENTS.md`).
2. **Remind the boot ritual:** *"Next session, read in order: `charter.md` → `plan.md` → latest session-log → last 5 entries of `decisions.md` → `watch.md` triggers."*
3. **Initialize git, or commit to the parent repo, whichever applies.** Three cases per pre-flight step 2:

   **Case A — not inside any git repo.** (`git rev-parse --show-toplevel 2>/dev/null` empty/non-zero.)
   - Write a sensible `.gitignore` for the detected tech stack (Node: `node_modules/`, `dist/`, `*.log`; Rust: `target/`; Python: `__pycache__/`, `*.pyc`, `.venv/`; etc.). Always include editor/OS noise (`.DS_Store`, `.vscode/`, `.idea/`, `*.swp`). If the project will write local runtime state outside the repo (e.g. `~/.pi/agent/*.json` for Pi extensions), add a defensive entry that catches accidental copies into the repo.
   - Run `git init -b main`, `git add -A`, commit with `chore: scaffold .pi/project (v0.4.1)` plus a body listing what was created and the risk classification. Use `-c user.email=… -c user.name=…` only if `git config user.email` is empty; otherwise inherit ambient config.
   - Confirm: `git log --oneline` shows one commit, `git status` is clean.

   **Case B — CWD is an existing repo root.** (`show-toplevel` equals CWD.) Skip `git init`. `git add -A` the scaffold files and commit with the same `chore: scaffold .pi/project (v0.4.1)` message. If the working tree was dirty before scaffolding, ASK whether to include those changes in this commit or stash them — bundling unrelated work into a scaffold commit is usually wrong.

   **Case C — CWD is inside a larger repo (monorepo / workspace / extension dir under a versioned parent).** (`show-toplevel` returns an ancestor of CWD.) **Never `git init` here.** That creates a nested repo — git tolerates it but treats it inconsistently (inner files become untracked from the outer repo's POV; outer hooks/CI/tooling won't see them; submodule semantics aren't auto-applied).
   - Default: commit the scaffold into the parent repo with `chore(<subdir>): scaffold .pi/project (v0.4.1)` to scope by subproject.
   - If the user explicitly wants this subproject to be its OWN repo (rare): tell them to move the directory outside the parent first, then re-run scaffold. Don't try to nest.
   - **Ask before committing** in Case C — it's rarer than A/B, and the parent's working-tree state matters more.

   In all three cases: never auto-confirm. A 2-second `ask_user` prompt before committing saves a 20-minute rebase later.

## Anti-patterns

- **Don't write files mid-interview.** The checkpoint exists for a reason — partial answers produce partial charters that rot.
- **Don't accept "I don't know" without probing.** If the user genuinely doesn't know the answer to a charter question, that's a *real* finding — record it as a known-unknown decision entry, with a re-check trigger in `watch.md`. Don't bury it with a placeholder.
- **Don't suggest extending the template** during scaffolding (risks.md, glossary.md, etc.). Wait until the user feels the pain — that's the v0.4.1 template policy.
- **Don't auto-migrate** legacy `.pi-project/` folders. The refuse-with-migration-command path in `## When NOT to use` is the only sanctioned route — print the command, let the user run it. Same rule for older `.pi/project/` layouts the user mentions in passing: migration is explicit, not automatic.
- **Don't run shell commands beyond pre-flight checks, file writes, and the closing git step.** Git handling at close is mandatory but adapts to context (Case A/B/C in Closing actions step 3); **never blindly `git init` inside an existing repo**. Everything else — `npm init`, `cargo init`, `pip install`, `tsc --init`, running tests, anything that touches dependencies or generates code — stays out of scaffolding. The scaffold's job ends at "project state captured in version control"; the project's job begins after.
- **Don't leave HTML comments un-substituted** in shipped files. The templates use them as substitution markers, not as documentation.

---
> Source: [DarkoKuzmanovic/pi-project-scaffold](https://github.com/DarkoKuzmanovic/pi-project-scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
