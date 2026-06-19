## vibe-coding-skill

> Help beginners and non-programmers do safer vibe coding by turning a rough software idea into a small, clear, AI-readable project plan before implementation. Use for new project ideas, MVP planning, URD/ADD/MDD/TDD/RMD docs, Python uv setup, git checkpoints, PR/merge safety, trace updates, and OKF AI notes.


# Vibe Coding Skill

Use this skill when a user has a new software idea and wants to use an AI coding assistant to build it from scratch without drifting, overbuilding, or losing track of decisions.

The public promise is simple:

```text
Turn a rough idea into a clear, testable project map for safer vibe coding.
```

The user does not need to know SDD, URD, ADD, MDD, TDD, RMD, axiomatic design, or coupling analysis. Those terms are internal machinery. When talking to beginners, use friendly names first and technical names only when useful.

## Beginner-facing names

Use these names in normal conversation:

| Friendly name | Internal file | Meaning |
| --- | --- | --- |
| Idea Brief | `docs/URD.md` | What the user wants, who it is for, what counts as success. |
| Design Split | `docs/ADD.md` | How the idea is split into independent parts before coding. |
| Building Blocks | `docs/MDD.md` | The modules, interfaces, data, and contracts. |
| Check Plan | `docs/TDD.md` | How we know the project works. |
| Build Path | `docs/RMD.md` | The safest order for the AI to implement things. |
| Project Map | `docs/TRACE.md` | Links from idea → design → code tasks → tests. |
| Project Setup | `pyproject.toml`, `.gitignore`, source and test folders | The starter files needed before coding. |
| Git Checkpoint | git branch, commit, pull request, merge | A safe save-and-review point after each build slice. |
| AI Notes | `okf/` | Short pages that help an AI retrieve the right context. |

The technical names may remain in file names because they are compact and stable. Do not force the user to learn the acronyms before they can use the skill.

## Directory layout

```text
pyproject.toml       # for Python projects, managed with uv
.gitignore
README.md
src/ or app/
tests/

docs/
  URD.md   # Idea Brief / User Requirement Document
  ADD.md   # Design Split / Axiomatic Design Document
  MDD.md   # Building Blocks / Module Design Document
  TDD.md   # Check Plan / Test-Driven Document
  RMD.md   # Build Path / Route/Runbook/Execution Map Document
  TRACE.md
  CHANGELOG.md
  PARKING_LOT.md

okf/
  index.md
  log.md
  terms/
  requirements/
  decisions/
  modules/
  interfaces/
  tests/
  paths/
  issues/
  references/

.vibe/
  trace.json
  doc_state.json
  coupling_history.json
  update_log.json
```

`docs/` is the source of truth. `okf/` is a derived AI-readable knowledge layer. `.vibe/` contains machine-readable state and trace files.

This skill is optimized for new projects. If the project already has a substantial codebase, say so and treat reverse-engineering existing code as a separate task.

## Core behavior

### 0. Operating guardrails

Use these guardrails in every mode. They prevent the AI from moving from planning to coding too early.

#### 🔴 CHECKPOINT moments

Pause and show the user the current decision before continuing when any of these happen:

| Moment | What to show | Continue only when |
| --- | --- | --- |
| Idea Brief is complete | target user, core task, scope, non-scope, constraints, assumptions, open questions | the user accepts or gives corrections |
| Design Split is classified | FR/DP list, design matrix, classification, retry log | uncoupled/decoupled design is accepted, or accepted coupling is explicitly recorded |
| Build Path is ready | first 3 RMD tasks, test commands, rollback points, Git checkpoint plan | the user agrees this is the implementation order |
| First real push or merge | remote name, branch, PR/merge action, tests run | the user or repo rules approve the remote action |
| Document rewrite would overwrite existing work | files to change, backup or diff summary | the user confirms overwrite or chooses a smaller edit |

#### 🛑 STOP conditions

Stop the current action and route the issue back to the right document when any condition below is true:

| Trigger | Required action |
| --- | --- |
| The user asks to code before URD has a user, task, scope, and success criterion | explain the missing decision and ask one concrete question, or create a labeled assumption if it does not block the next step |
| ADD remains coupled after 3 structural retries | create an Accepted Coupling record with risks and guard tests before implementation |
| A test has no oracle | update TDD before writing implementation code |
| MDD lacks an interface contract for a public API | update MDD before writing that API |
| OKF contradicts docs | write `okf/issues/PROB-xxxx.md`, then fix `docs/` first |
| Git working tree contains unrelated changes | show `git status`, avoid committing unrelated files, and ask before touching them |
| A push, merge, deletion, secret exposure, or irreversible operation is needed | ask for explicit permission before executing |

#### Failure handling table

| Failure mode | First response | Fallback if still blocked |
| --- | --- | --- |
| User gives a vague idea | ask one high-value question with 2–3 options | record assumptions and proceed with simple docs only |
| User wants a one-off script | offer `simple` mode and explain what will be skipped | if they still want no docs, do not force this skill; answer the coding request directly |
| Existing repo already has files | inspect before writing and preserve existing files by default | write proposed changes to new files or backups instead of overwriting |
| `uv` is unavailable | create `pyproject.toml`, `.gitignore`, `tests/`, and README fallback files | tell the user the exact `uv` command to run later |
| Tests cannot run | record the command, failure output, and affected RMD task | block merge and add a TDD/RMD note with next diagnostic step |
| No git remote exists | make local commits only | record `PR skipped: no remote` in RMD |
| OKF page is too long | split it by retrieval question | keep only navigation in `index.md` and move facts into concept pages |
| Trace link is missing | add the smallest missing link | if source or target ID is unclear, create a parking-lot or issue entry instead of inventing the ID |

#### Anti-pattern blacklist

Do not do these things when this skill is active:

1. Do not start feature implementation before the Idea Brief and Check Plan have enough facts to define success.
2. Do not turn OKF pages into copies of `docs/` pages.
3. Do not invent requirements, libraries, external services, users, or constraints to make the plan look complete.
4. Do not split modules into meaningless pieces only to make the ADD matrix diagonal.
5. Do not hide accepted coupling; record it with guard tests.
6. Do not commit secrets, `.env`, virtual environments, local databases, caches, or generated artifacts.
7. Do not push, merge, delete, or overwrite real project files without explicit approval.
8. Do not regenerate every document when a local update is enough.
9. Do not create IDs that are never referenced by TRACE or OKF.
10. Do not claim tests passed unless the exact command ran successfully; if not run, say so.

### 1. Start with a plain-language explanation when needed

If the user is a beginner, non-programmer, or seems unsure what the process means, first explain the workflow in 5–8 short sentences.

Do not start with acronyms. Say something like:

```text
Before coding, we will make a small project plan.
First, we clarify what you want and what should be left out.
Then we split the project into parts that can be built separately.
Then we write down how to check whether each part works.
Finally, we give the AI a build order so it does not jump around randomly.
```

When the user asks “why are you asking this?”, explain the immediate reason:

- “This affects what we test.”
- “This changes which data we need to store.”
- “This decides whether we need login.”
- “This prevents the AI from building a feature you do not want.”

Do not lecture. Keep explanations short, concrete, and tied to the current question.

### 2. Discuss the idea before design

In the Idea Brief phase, do not jump directly to architecture or implementation.

Ask questions only when the answer affects design, testing, or scope. Offer concrete options when helpful.

Ask when any of these are unclear:

- who will use the project
- what the user mainly wants to do
- what a successful result looks like
- what is in scope
- what is out of scope
- whether login, permissions, payments, private data, or external services are involved
- where the project should run
- whether speed, cost, privacy, reliability, or simplicity matters most
- whether requirements conflict

Do not ask endlessly. If an uncertainty does not block the next step, record it as an assumption or open question.

The Idea Brief may move to Design Split only after the first 🔴 CHECKPOINT and when all of these are present:

- at least one target user or user role
- at least one core user task
- at least one measurable success criterion
- explicit in-scope and out-of-scope lists
- main constraints
- assumptions and open questions separated from confirmed requirements

### 3. Apply Ockham's razor to all documents

The document set must stay small. More documents are not better; clearer decisions are better.

Do not write content unless it directly helps one of these tasks:

- confirm user intent
- reduce ambiguity
- preserve a design decision
- reduce coupling
- define a module, interface, or contract
- define a test oracle
- order development tasks
- help an AI retrieve the correct context later

Avoid these forms of document bloat:

- repeating the same idea in every document
- writing future features into the current version
- adding generic architecture lessons
- adding implementation detail before it is needed
- turning OKF concept pages into copies of docs pages
- creating IDs that are never referenced
- keeping stale assumptions after the user has answered them

If something is interesting but not needed for the current version, move it to `docs/PARKING_LOT.md`.

After every generation or update, run this check:

```text
For each newly added paragraph:
1. Does it serve the current phase?
2. Is it linked to a requirement, design part, module, interface, test, task, or OKF concept page?
3. Is it repeated elsewhere?
4. Does it describe a future feature rather than current scope?
5. Could it be derived reliably from another document?

If the answer shows low value, delete, merge, shorten, or move it to PARKING_LOT.
```

### 4. Use Design Split to reduce coupling, but do not fake zero coupling

Internally, Design Split uses Functional Requirements (FRs), Design Parameters (DPs), and a design matrix.

For beginners, explain it this way:

```text
We are checking whether each user need can be built by one clear part of the system.
If one part accidentally affects many unrelated needs, the AI is more likely to create messy code.
```

Matrix interpretation:

- diagonal matrix: uncoupled design
- triangular matrix: decoupled design with an execution order
- dense or irregular matrix: coupled design

When Design Split detects coupling, retry the design decomposition before accepting it. If the retry loop fails, trigger the ADD coupling 🛑 STOP condition.

Coupling retry loop:

```text
1. Try to split broad user needs into smaller independent needs.
2. Try to redefine design parts so each part satisfies one need.
3. Try to introduce explicit interfaces, events, adapters, pure data structures, or contract boundaries.
4. Try to move responsibility between modules.
5. Repeat for up to 3 structural retries.
6. If coupling remains, record it as Accepted Coupling.
```

For each retry, record what changed and why.

For accepted coupling, record:

- coupled requirements and design parts
- why the coupling is necessary or temporarily accepted
- affected modules
- risks
- tests required to guard the coupling
- future condition that would justify refactoring

Do not split the system into meaningless pieces just to make a pretty matrix. That violates Ockham's razor.

### 5. Keep docs authoritative and OKF derived

`docs/` contains project specifications and decisions. `okf/` contains an OKF v0.1 knowledge bundle derived from those docs.

OKF shape:

- `okf/` is a Knowledge Bundle: a self-contained directory tree of UTF-8 Markdown files.
- Every non-reserved `.md` file is a Concept document. Its Concept ID is its bundle-relative path without `.md`, such as `modules/parser`.
- Concept files start with parseable YAML frontmatter, followed by a Markdown body.
- Every concept frontmatter must include a non-empty `type` field. OKF has no central type registry; use descriptive types and tolerate unknown types when reading.
- Recommended frontmatter fields, in priority order, are `title`, `description`, `resource`, `tags`, and `timestamp`.
- Skill-specific extension fields such as `page_id`, `source_ids`, and `status` are allowed because OKF permits producer-defined keys. Preserve unknown keys when editing.
- `index.md` and `log.md` are reserved filenames at any level and must not be used as concept documents.
- `index.md` is for progressive disclosure. It lists nearby concepts and subdirectories with relative links and short descriptions. Only the bundle-root `okf/index.md` may use frontmatter, and only to declare `okf_version: "0.1"`.
- `log.md` is optional update history. Date headings use `## YYYY-MM-DD`, newest first.
- Use normal Markdown links between concepts. Prefer bundle-relative absolute links such as `/modules/parser.md`; relative links are allowed. Broken links are warnings, not fatal errors.
- When body text relies on external sources, collect numbered links under `# Citations`. External material may also be mirrored as first-class concept pages under `okf/references/`.

Rules:

- Change docs first, then update OKF.
- OKF must not introduce new requirements.
- OKF must not silently override docs.
- If OKF compilation exposes a conflict, create `okf/issues/PROB-xxxx.md`, then route the fix back to URD, ADD, MDD, TDD, or RMD.
- Prefer many short OKF concept pages over one long page.
- An OKF concept page should answer one retrieval question.
- Treat OKF conformance errors narrowly: missing or invalid concept frontmatter, missing `type`, or malformed reserved files. Treat optional fields, unknown fields, unknown types, and broken links as warnings.

### 6. Initialize the coding project before implementation

Before implementation begins, create a small, conventional project structure and pass the Build Path 🔴 CHECKPOINT. Do not let the AI write feature code into a messy or undefined folder.

For beginner-facing explanation, say:

```text
Before writing features, we will make a clean project folder.
That gives the AI a predictable place for code, tests, settings, and documentation.
For Python projects, we will use uv so dependencies live in pyproject.toml instead of random install commands.
```

Stack defaults:

- If the user already chose a stack, use it.
- If the project is automation, CLI, data processing, API, bot, backend, or a general small tool, default to Python unless another stack is clearly better.
- If the project is a static website or visual prototype, default to HTML/CSS/JavaScript unless the user already chose another stack.
- If the stack is unclear, ask one short question with 2–3 concrete options.

Python project defaults:

```text
- use uv for package and environment management
- create or preserve pyproject.toml
- create .gitignore before coding
- create README.md
- create src/ or app/ for source code
- create tests/ for tests
- do not commit .venv, __pycache__, secrets, local databases, or generated caches
```

Use `uv init` when uv is available. If uv is unavailable, create the same essential files and tell the user to install uv before dependency work.

For Python applications, prefer:

```bash
uv init --app .
uv add <runtime-dependency>
uv add --dev pytest ruff
uv sync
```

For Python libraries, prefer:

```bash
uv init --lib .
uv add --dev pytest ruff
uv sync
```

The exact dependencies must come from MDD/TDD needs. Do not add frameworks speculatively.

### 7. Use Git Checkpoints in the Build Path

Every RMD implementation slice must end with a Git checkpoint unless the user explicitly disables git. First remote push and first merge are 🔴 CHECKPOINT moments.

A checkpoint means:

```text
1. Start from a clean main branch.
2. Create a short feature branch for one RMD task or one small slice.
3. Implement only that slice.
4. Update affected docs and OKF.
5. Run relevant tests and checks.
6. Review git diff and git status.
7. Commit with a clear message linked to RMD/URD/TDD IDs.
8. Push the branch if a remote exists.
9. Open a pull request.
10. Merge only after tests pass and the user approves, or after the repo's configured review rules pass.
```

For beginners, explain it this way:

```text
A commit is a save point. A pull request is a review page. A merge puts the reviewed work back into the main version.
We do this after each small slice so mistakes are easier to find and undo.
```

Git safety rules:

- Never commit secrets, API keys, passwords, tokens, `.env`, `.venv`, local databases, caches, or build artifacts.
- Never merge directly to `main` while tests are failing.
- Never push or merge if the user asked for a local-only project.
- Ask for explicit permission before the first push or merge in a real remote repository.
- If there is no GitHub remote, still create local commits and record `PR skipped: no remote` in RMD.
- If the user is using another platform, use the same branch → review request → merge idea with that platform's terms.

Recommended commands for one slice:

```bash
git checkout main
git pull --ff-only
git checkout -b feat/rmd-task-001-short-name
# edit code and docs
uv run pytest
uv run ruff check .
git status
git diff
git add <changed-files>
git commit -m "feat: implement RMD-TASK-001 short name"
git push -u origin feat/rmd-task-001-short-name
gh pr create --fill
```

Merge policy:

```text
- Prefer squash merge for beginner projects to keep history readable.
- Merge only after checks pass.
- Delete merged feature branches when safe.
- After merge, update local main and continue with the next RMD task.
```

### 8. Maintain explicit traceability

Every important item gets a stable ID.

Use these prefixes:

```text
URD-GOAL-001
URD-ROLE-001
URD-REQ-001
URD-AC-001
URD-CON-001
URD-ASM-001
URD-Q-001
ADD-FR-001
ADD-DP-001
ADD-COUP-001
MDD-MOD-001
MDD-API-001
MDD-DATA-001
TDD-TEST-001
RMD-TASK-001
RMD-GIT-001
RMD-PR-001
RMD-STOP-001
OKF-PAGE-001
PROB-001
DEC-001
```

Maintain trace links in both `docs/TRACE.md` and `.vibe/trace.json`.

A normal chain looks like this:

```text
URD-REQ-001
  -> ADD-FR-001
  -> ADD-DP-001
  -> MDD-MOD-001
  -> MDD-API-001
  -> TDD-TEST-001
  -> RMD-TASK-001
  -> okf/modules/example.md
```

When a document changes, update affected downstream documents and trace records. If uncertain, create an issue page and trigger the relevant 🛑 STOP condition before inventing behavior.

## Document strength levels

Choose the smallest document set that can safely guide development.

### simple

Use for scripts, prototypes, single-page tools, and small experiments.

Create:

- `docs/URD.md`
- `docs/ADD.md`
- `docs/TDD.md`
- `docs/TRACE.md`
- minimal OKF bundle: root `okf/index.md` plus concept pages only when they help retrieval

MDD, RMD, and OKF log history can be omitted until the project needs them.

### standard

Use for ordinary apps, APIs, web services, tools, and small product projects.

Create full docs:

- URD
- ADD
- MDD
- TDD
- RMD
- TRACE
- CHANGELOG
- PARKING_LOT
- OKF concept pages

### strict

Use for projects involving money, identity, permissions, security, data loss, multi-user collaboration, irreversible actions, regulated data, or high reliability needs.

Use full docs plus:

- explicit contracts
- negative tests
- failure and rollback paths
- security and privacy constraints
- accepted coupling records
- stricter trace coverage

## Modes

### mode: explain_plainly

Goal: help a beginner understand what this skill is doing and why.

Inputs:

- user's project idea
- user's confusion or question
- current phase, if known

Outputs:

- short plain-language explanation
- one next action or one high-value question

Rules:

- Do not mention SDD first.
- Use friendly names: Idea Brief, Design Split, Building Blocks, Check Plan, Build Path.
- Keep explanations short.
- Explain the current step, not the whole theory.
- Use concrete examples from the user's project.
- End with the next useful question or action.

### mode: initialize_project_docs

Goal: create the directory structure and empty templates.

Actions:

1. Determine document strength: simple, standard, or strict.
2. Create `docs/`, `okf/`, and `.vibe/`.
3. Populate templates.
4. Tell the user what was created using friendly names.
5. Ask the next Idea Brief question.

Recommended command:

```bash
python scripts/init_project_docs.py --target . --level standard --force
```

### mode: initialize_coding_project

Goal: create a clean starter project before writing feature code.

Inputs:

- chosen or inferred stack
- project name
- document strength
- target folder
- whether a remote GitHub repository exists

Outputs:

- starter project files such as `pyproject.toml`, `.gitignore`, `README.md`, `src/` or `app/`, `tests/`
- initialized git repository when appropriate
- first local commit, if the user has approved git usage
- updated `docs/RMD.md` setup section

Rules:

- For Python, default to uv.
- Do not add heavy frameworks until MDD/TDD requires them.
- Create `.gitignore` before the first commit.
- Create tests folder before feature work.
- If uv is not installed, create files without installing dependencies and explain the next installation step.
- If a remote exists, do not push until the user has approved the remote target at a 🔴 CHECKPOINT.

Recommended command:

```bash
python scripts/init_python_uv_project.py --target . --name my-project --kind app --force
```

### mode: git_checkpoint

Goal: safely finish one RMD task or implementation slice.

Inputs:

- one RMD task ID
- changed code files
- changed docs/okf files
- relevant test commands
- current branch and remote state

Outputs:

- passing test/check output or recorded failure
- git commit
- pull request, if a remote exists and push is approved
- merge record, if checks pass and merge is approved
- updated `docs/RMD.md`, `docs/TRACE.md`, and `docs/CHANGELOG.md`

Rules:

- Work on one task branch at a time.
- Commit only after tests/checks and doc updates are complete, unless making an explicit WIP commit requested by the user.
- Commit messages should include the RMD task ID.
- PR description should include: what changed, how tested, affected docs, known risks.
- Merge only after tests pass and the user or repo rules approve at a 🔴 CHECKPOINT.
- If push/PR/merge is not possible, record the reason and keep the local commit.

### mode: discover_urd

Goal: turn the user's idea into a small Idea Brief.

Inputs:

- user idea
- prior URD if present
- current assumptions/open questions if present

Outputs:

- `docs/URD.md`
- updated `docs/TRACE.md`
- updated `.vibe/trace.json`
- possible `docs/PARKING_LOT.md`

Rules:

- Ask only design-relevant questions.
- Offer concrete options when helpful.
- Put uncertainty into assumptions or open questions.
- Keep implementation guesses out of URD.
- End with a short list of what is confirmed and what remains uncertain.

### mode: analyze_add

Goal: convert the Idea Brief into Design Split, then detect coupling.

Inputs:

- `docs/URD.md`
- `docs/TRACE.md`

Outputs:

- `docs/ADD.md`
- `docs/TRACE.md`
- `.vibe/coupling_history.json`

Rules:

- Extract FRs from confirmed URD requirements and acceptance criteria.
- Propose DPs without prematurely naming concrete libraries unless constraints require them.
- Build the matrix.
- Retry if coupled.
- Prefer uncoupled design; accept decoupled design when an execution order is clear.
- Record accepted coupling explicitly.

### mode: design_mdd

Goal: turn DPs into Building Blocks: modules, interfaces, contracts, and data structures.

Inputs:

- `docs/ADD.md`
- `docs/URD.md`
- `docs/TRACE.md`

Outputs:

- `docs/MDD.md`
- updated trace

Rules:

- Each module must have a narrow responsibility.
- Each public interface needs inputs, outputs, preconditions, postconditions, invariants, and side effects.
- Default to pure functions and immutable data where practical.
- Use data-oriented design only where access patterns or performance constraints justify it.
- Do not repeat URD prose; cite requirement IDs.

### mode: write_tdd

Goal: define the Check Plan before implementation.

Inputs:

- `docs/URD.md`
- `docs/ADD.md`
- `docs/MDD.md`

Outputs:

- `docs/TDD.md`
- updated trace

Rules:

- Every acceptance criterion should map to at least one test unless explicitly deferred.
- Include contract tests for public interfaces.
- Include negative tests for invalid inputs, permission failures, and boundary cases.
- For strict projects, include security/privacy failure cases.
- Tests must name their oracle: what exact result proves correctness.

### mode: plan_rmd

Goal: define a safe Build Path for implementation.

Inputs:

- URD, ADD, MDD, TDD, TRACE

Outputs:

- `docs/RMD.md`
- updated trace

Rules:

- Order tasks by dependency and risk.
- Prefer interface stubs and failing tests before implementation.
- Add 🛑 STOP conditions for unresolved requirements, coupling violations, missing contracts, unsafe Git actions, and missing test oracles.
- Add rollback points after interface design, test creation, and each completed module.
- Each RMD task must include a Git checkpoint: branch name, commit point, PR status, merge status, and test command.
- If the user is working without a remote, mark PR and merge as local-only or skipped with reason.

### mode: compile_okf

Goal: compile docs into a small OKF v0.1 Knowledge Bundle for AI Notes.

Inputs:

- all docs
- trace files
- existing `okf/` pages, if any

Outputs:

- `okf/index.md` with progressive-disclosure links
- focused OKF concept pages
- optional `okf/log.md` update history
- optional `okf/references/` pages for mirrored external references
- issue pages for conflicts

Rules:

- Do not copy full docs into OKF.
- Make each page short and task-oriented.
- Generate one Markdown file per concept.
- Use the bundle-relative file path without `.md` as the Concept ID; keep paths stable after linking.
- Start each concept with YAML frontmatter and include a non-empty `type` field.
- Add `title`, `description`, `resource`, `tags`, and `timestamp` when they improve retrieval or provenance.
- Include source doc IDs in the extension field `source_ids`, not long source text.
- Use normal Markdown links to connect concepts. Prefer absolute bundle-relative links like `/requirements/login.md`.
- Use `# Schema`, `# Examples`, and `# Citations` only when they fit the concept.
- Put external supporting links under `# Citations`, or mirror durable source summaries under `okf/references/`.
- Use `okf/index.md` and subdirectory `index.md` files for navigation and short descriptions.
- Use `okf/log.md` only for chronological bundle updates, with `## YYYY-MM-DD` headings.
- If docs conflict, write an issue page and route the fix back to docs.
- Do not reject an OKF page only because it has unknown frontmatter keys, unknown `type`, missing optional fields, or a broken internal link. Warn and continue.

### mode: update_docs

Goal: update docs and OKF when requirements, decisions, tests, or planned paths change.

Inputs:

- user's requested change
- current docs and trace files

Outputs:

- changed docs
- updated trace
- updated OKF
- updated changelog

Update process:

```text
1. Identify the source item changed by the user.
2. Locate downstream items through TRACE.
3. Update the smallest necessary set of documents.
4. Update trace records.
5. Update or regenerate affected OKF concept pages.
6. Add an entry to CHANGELOG.
7. Run Ockham check and trace lint.
```

Never regenerate the whole document set when a local update is enough.

## Shipping gates

Before handing off project docs, verify:

- Idea Brief has confirmed requirements, assumptions, and open questions separated.
- Design Split has FRs, DPs, matrix, coupling status, and retry record if needed.
- Building Blocks does not repeat the Idea Brief; it references IDs.
- Check Plan has test oracles and requirement links.
- Build Path has task order, stop conditions, and rollback points.
- Project Map connects requirements to design, modules, tests, tasks, and OKF.
- OKF concept pages are short, linked, and derived from docs.
- OKF reserved files follow their roles: `index.md` for navigation, `log.md` for dated update history.
- PARKING_LOT contains future ideas that should not pollute current scope.
- CHANGELOG records document changes.
- Ockham check has removed repetition and speculative content.

Recommended check command:

```bash
python scripts/check_project_docs.py --root .
```

## Response style when using this skill

When interacting with the user:

- Be direct and concrete.
- Prefer friendly names with beginners.
- Ask a few high-value questions, not a questionnaire dump.
- When making assumptions, label them.
- When rejecting document bloat, explain which current decision or test it fails to support.
- Prefer small updates over large rewrites.
- Avoid abstract slogans. Show the exact document or trace change.

---
> Source: [goldengrape/vibe-coding-skill](https://github.com/goldengrape/vibe-coding-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
