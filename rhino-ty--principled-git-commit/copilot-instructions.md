## principled-git-commit

> >


# Commit Conventions

> Universal commit conventions derived from Conventional Commits 1.0.0 + Tim Pope + `awesome-copilot/git-commit` + a 200-commit empirical study. Project-specific dialect (domain proper nouns, custom trailers, workflow integrations like PDCA) lives in the project's `docs/references/COMMIT.md` — see §15 Dialect Loading.
>
> **TL;DR**: `type(scope): summary` (≤100 chars, lowercase, imperative) + 1-2 line context + `- ` bullets (avg 16 lines). No `##` headers. English body by default. Atomic + leaves-repo-green + why-over-what.

---

## 0. Principles

A commit must satisfy four readers — humans **and** AI:

1. **`git log --oneline` scanner** — wants context from a single line (most common reader)
2. **`git blame <file>` tracer** — wants to know "why is this line here?" while debugging
3. **`git bisect` hunter** — wants to isolate the exact commit that broke production
4. **AI agent** — rebuilds context after `/clear`, reviews PRs, generates changelogs, answers natural-language history queries. Depends on `grep` + embedding search + consistent structure. Atomic units, domain keywords, English body, and explicit trailers are decisive.

The five principles below are the minimum that satisfies all four readers simultaneously. Return here when in doubt.

### 0.1 Atomic — one commit, one intent

Bundle exactly one logical change. Never mix "fix + style + docs" in one commit.

→ **Bisect, revert, and cherry-pick all assume atomic units.** Mixed commits also confuse AI summarization — the LLM grabs one intent and hallucinates the rest as a single coherent change.

> Test: "If I had to revert this commit, would I drag along unrelated changes?" If yes, split.

### 0.2 Leaves repo green — every commit builds clean

Each commit, taken on its own, must pass `tsc --noEmit` + `lint` + `build` (or your project's equivalent green-build set). No "WIP, next commit will fix this."

→ **Bisect signal integrity.** A broken-state commit in the middle of history poisons `git bisect` results — the bisect ends up blaming the wrong commit. AI-driven regression analysis suffers the same way.

### 0.3 Why over what — body explains motivation, not changes

The diff already shows **what** changed. The body's job is **why** — the motivation, the trade-off, the alternative you rejected, the constraint that forced this approach.

→ **Answers the 6-month-later blame question directly.** AI can extract intent from the body without re-deriving it from the diff (diff inference is expensive and prone to misinterpretation).

```
❌ Body: "modify foo.ts to add bar method"        (diff already shows this)
✅ Body: "extract bar() out of inline closure —    (the WHY)
        prevents re-allocation on every render
        causing useMemo deps invalidation"
```

### 0.4 Imperative mood — "If applied, this commit will..."

Summary is in imperative mood. Matches git's own internal messages (`Merge`, `Revert`, `Initial commit`).

→ **Consistent verb pattern.** Auto-changelog tooling and AI classification (type inference, change summarization) both depend on stable verb forms.

```
✅ add idempotency-key support to /v1/payments     (If applied, this commit will add ...)
✅ fix double-render in <Modal> on focus return
✅ migrate auth hashing from bcrypt to argon2id
❌ added idempotency-key support                   (past tense)
❌ adding idempotency-key support                  (gerund)
❌ idempotency-key support added                   (passive)
```

### 0.5 Searchable — keyword-rich summary and body

Name domain nouns, function names, file paths, SoT names, and component names explicitly. Vague verbs (`improve`, `update`, `enhance`, `cleanup`) must be paired with a concrete noun.

→ **Targets `git log --grep` and AI embedding search.** Six months later, a query like "where did we add idempotency keys to the payments endpoint?" should hit exactly one commit. Vague keywords cause AI hallucination; concrete keywords get pinpoint accuracy.

```
❌ feat(api): improve checkout
✅ feat(api/payments): add Idempotency-Key header support with 24h replay window

❌ refactor: cleanup auth helpers
✅ refactor(auth): replace bcrypt with argon2id (memory-hard against GPU attackers)
```

Keyword checklist: scope path / function or component name / pattern name / SoT name / domain enum.

---

## 1. Format

```
type(scope): summary

[optional 1-2 line context — why]

- bullet: concept- or file-level change
- bullet: ...

[optional Verification: block]
[optional trailers — Refs / BREAKING CHANGE / project-dialect trailers]
```

### 1.1 Summary line

| Property | Rule | Note |
|---|---|---|
| Length | **≤100 chars recommended** (empirical avg 71, max 116) | Strict 50/72 not enforced — clarity beats brevity rule |
| Form | `type(scope): summary` | Scope is optional but almost always specified |
| Subject case | **lowercase** (`feat(...): add ...`) | 200/200 in study were lowercase. Proper nouns retain their case |
| Language | **English by default**. Project-specific proper nouns (e.g., Korean product names) keep their original script — see project dialect | |
| Trailing punctuation | none | |
| Mood | imperative — see §0.4 | |

### 1.2 Body

- **`- ` bullets dominate**. In the 200-commit study: 981 bullets vs 5 `##` headers — headers are an outlier and therefore noise.
- Section labels in plain text: `Verification:` / `Server:` / `Plan + Design:` (not `## Verification`).
- `path: change` pattern (`server/src/foo.service.ts: rewrite`).
- Average 16 lines. Anything past **80 lines** indicates a missed split — the commit probably bundles two intents (see §0.1).
- English-default; switch to native language only when nuance is irretrievable in English.

### 1.3 Three commit surfaces (commit / PR title / squash message)

A commit message can be authored at three different surfaces depending on workflow. They all follow the same §0 principles but differ in audience and constraints:

| Surface | Audience | Constraint |
|---|---|---|
| **Individual commit** (`git commit`) | future-you / blame / bisect | Universal §1 format. Atomic. Leaves repo green. |
| **PR title** (squash-merge teams) | reviewer / changelog / `git log` after squash | Same format as individual commit — **becomes the squashed commit summary** at merge time. Therefore must satisfy §1.1 from PR creation, not just at merge. |
| **PR squash body** (squash-merge teams) | future-you reading squashed `main` history | The PR description becomes the body. Therefore the description must satisfy §1.2 (no `##` headers, why-over-what bullets, average 16 lines) — most teams' default GitHub PR templates fail this. |

**Trunk-based teams** (no long-running branches): only the **individual commit** surface matters. The other two are absent.

**PDCA / Linear / Jira teams** with feature branches that pre-merge or fast-forward: the individual commit surface is canonical; PR title is informational.

> **Practical**: if your team squash-merges, lint the PR title against §1.1 at PR creation (GitHub Action), and lint the PR description against §1.2. CI rejection at PR-creation time costs less than rejection at merge time.

---

## 2. Types

| Type | Purpose | Example |
|---|---|---|
| `feat` | New feature, new endpoint, new domain | `feat(shared): add phone domain` |
| `fix` | Bug fix, regression repair | `fix(auth): clear refresh token cookie on logout` |
| `refactor` | Behavior unchanged, structure changed (extract / rename / SoT migration) | `refactor(routes): extract route definitions into a single registry` |
| `docs` | Documentation only — markdown, code comments, ref docs | `docs(api): document rate-limit headers for /v1/users` |
| `style` | Visual/formatting only — Tailwind class, badge tone, prettier sweep — no logic change | `style(navbar): align logo wordmark with new brand kit` |
| `chore` | Build, tooling, scaffold, dependency bumps | `chore(deps): bump react 18.3.1 → 19.0.0` |
| `test` | Test additions or regression baselines | `test(checkout): add e2e flow for guest user with split payment` |
| `perf` | Performance-only change (measurable benchmark) | `perf(query): cache user permission lookup (300ms p95 → 12ms)` |
| `ci` | CI / GitHub Actions / pipeline config | `ci(release): cache pnpm store across release matrix` |
| `build` | Build system / bundler / compiler config | `build(rollup): output ESM + CJS from a single externals declaration` |
| `revert` | The output of `git revert` — see §10 | |

> Project may not use every type. If `perf`/`ci`/`build` are unused, document the omission in the project dialect.

---

## 3. Scopes

### 3.1 Single scope

```
feat(client): add ...
fix(server): handle ...
docs(refs): add ...
```

### 3.2 Sub-scope — slash-separated

When a domain has internal modules, use `/` to drill down:

```
refactor(packages/ui/Table): split row-renderer out of Table component
refactor(apps/dashboard/billing): extract invoice form into a hook
refactor(server/auth): unify session cookie attribute setters
fix(api/search): handle empty filter array in query builder
```

### 3.3 Multi-scope — comma-separated

When one commit spans multiple domains (≤3 recommended):

```
docs(claude,ui): document RHF / error / sticky bar / 3-zone patterns
docs(claude,refs): document server phone SoT after unification cycle
```

### 3.4 Feature-scope

Long-running features with their own document chain (PDCA-style cycles, multi-week migrations) can use the feature name directly as scope:

```
feat(auth-rewrite-2026-q2): M1 — replace bcrypt with argon2id verifier
feat(acme-pay-launch): wire Stripe Issuing webhook handler
```

The **scope catalog** is project-specific — projects should record their actual high-frequency scopes in the dialect (see §15.2).

### 3.5 Non-Conventional formats are also valid

Conventional Commits is the dominant format in modern web/JS projects but **not the only valid format**. The §0 founding principles (atomic, leaves-repo-green, why-over-what, imperative, searchable) apply equally to projects using:

| Format | Example | Where it's used |
|---|---|---|
| **Kernel-style** (slash subsystem path, no parens) | `mm/oom_kill: avoid OOM-killer for processes with mm == NULL` | Linux kernel, Postgres, OpenBSD |
| **Bracket-prefix** (older React / jQuery) | `[Component] fix focus restoration on dialog close` | older OSS projects, occasionally still seen |
| **Tag-prefix** (Mozilla / Bug-numbered) | `Bug 1234567 - prevent leaked timer in async fetch` | Firefox, Mozilla projects |
| **No-prefix imperative** (Linus's own style) | `Avoid double free in error path` | small projects, hobby repos |
| **Issue-id prefix** | `ABC-1234: implement export endpoint` | Jira-mandated repos |

This skill defaults to Conventional Commits because it's the most common in the surrounding ecosystem (skills.sh users, Claude/Copilot integrations, npm package conventions). But your dialect file (§13) can declare a different format — the principles still apply, only the surface vocabulary changes.

If your project uses non-Conventional format, document it in dialect §1 alongside the regex/grammar your CI enforces. Consumers of `git interpret-trailers` and changelog generators may need wrapper scripts.

---

## 4. Body Length Sweet Spot

| Change scope | Body lines | Example |
|---|:--:|---|
| Single fix / style sweep | 0-2 | `fix(button): preserve focus ring on disabled state` (no body) |
| Single concept | 5-10 | A focused fix or extraction |
| Module-scale | 15-25 | Multi-file refactor or feature module |
| Architecture change | 30-40 | New domain scaffolding, page rewrite |
| Cross-cycle wrap | 50-80 (rare) | Plan + Design + first implementation in one commit |

> Past 80 lines, you almost certainly should have split (§0.1).

---

## 5. Markdown in Body

`git log` displays the body as raw text. GitHub UI renders markdown but raw output is the primary consumer.

| Element | Use | Reason |
|---|:--:|---|
| `- ` bullets | ✅ standard | natural visual grouping in raw |
| `**bold**` | ✅ sparingly | works in both raw and rendered |
| `` `path/foo.ts` `` (backticks) | ✅ liberally | quote file/symbol names |
| `Section:` plain label | ✅ | pairs with indented bullets |
| `## Header` | ❌ avoid | renders as literal `## Foo` in `git log` (5/200 outlier) |
| `### Sub-header` | ❌ | same |
| Long fenced code blocks | ❌ | short snippets only; longer goes in linked file |
| Tables (`|`) | △ occasional | small comparisons only; large tables belong in PR description |

---

## 6. Breaking Changes

API / SoT / response shape / DB column / env var changes that break callers. Use either or both notations.

### 6.1 `!` notation

```
feat(api)!: rename /v1/users to /v1/people
refactor(shared)!: move stripPhone to @namespace/shared/phone (server inline removed)
```

The `!` after type/scope is itself a breaking-change signal. Searchable via `git log --grep='!:'`.

### 6.2 `BREAKING CHANGE:` footer

```
refactor(shared): migrate phone SoT to @namespace/shared/phone

- 24 server inline calls swapped for stripPhone() import
- DTO @Matches replaced with @IsKoreanPhone() decorator

BREAKING CHANGE: server-side direct imports from @/lib/utils/phone
no longer exist — only @namespace/shared/phone exports the helpers.
Update all import paths.
```

### 6.3 When to flag

- ✅ DTO request/response shape change
- ✅ shared workspace export removal/rename
- ✅ DB schema column drop / NOT NULL added
- ✅ env var rename
- ✅ public API URL change
- ❌ internal implementation change (callers unaffected)
- ❌ optional field addition
- ❌ default value addition with backwards compatibility

---

## 7. Commit Workflow

The five-step flow that produces a green-build atomic commit.

### 7.1 Inspect the diff first

Stage and type decisions both flow from this. Don't guess — read the diff.

| Situation | Command |
|---|---|
| Unstaged changes | `git diff` |
| Already staged | `git diff --staged` |
| Changed-file list | `git status --porcelain` |
| File history | `git log -p <path>` |
| Volume at a glance | `git diff --stat` |

### 7.2 Staging strategy

| Pattern | Command | When |
|---|---|---|
| Explicit path | `git add server/src/foo.ts client/...` | **default** — guarantees atomic |
| Glob | `git add 'server/**/*.spec.ts'` | grouping homogeneous files |
| Interactive hunk | `git add -p` | one file carries two intents — split here |
| Single directory | `git add docs/03-analysis/` | bundled deliverable |

❌ `git add -A` / `git add .` — risks unrelated changes and secrets, violates §0.1.

> Tip: `git add -p` is the standard tool when one file accidentally accumulates two unrelated changes. Stage the hunks for intent A, commit, then stage hunks for intent B and commit again.

### 7.3 Type decision tree

After §7.1 diff inspection:

| Diff pattern | Type |
|---|---|
| New function / component / file with new behavior | `feat` |
| New endpoint / DB column / event / hook | `feat` |
| Same behavior, structure changed (extract / rename / SoT consolidation) | `refactor` |
| Wrong behavior → correct behavior (user-visible regression fix) | `fix` |
| Test files only (`*.spec.*` / `*.test.*` / `tests/`) | `test` |
| `*.md` / `docs/` / code comments only | `docs` |
| Tailwind class / CSS / badge tone (no logic) | `style` |
| `package.json` / lockfile / `tsconfig.json` / scaffold | `chore` |
| `git revert` output | `revert` |
| API request/response shape / SoT export removal/rename | `<type>!` + §6 footer |

When ambiguous, follow the **largest impact** (e.g., `refactor` plus an incidental `fix` is `refactor`).

### 7.4 Secrets blocklist

`.gitignore` is not enough — a freshly-created secret file may slip in. Verify before commit:

| Pattern | Risk |
|---|---|
| `.env` / `.env.local` / `.env.*` | env vars / DB / API keys |
| `credentials.json` / `service-account.json` | auth keys |
| `*.pem` / `*.key` / `*.p12` / `*.pfx` | cryptographic keys |
| `*.jar` / `*.properties` | vendor SDK assets that often carry credentials |
| `id_rsa*` / `.ssh/` | SSH keys |
| `*.dump` / `*.sql.bak` / `db_backup_*` | production data |
| `.npmrc` (with auth token) | registry token |

**Verification command**:

```bash
git diff --staged --name-only | grep -iE '\.(env|pem|key|p12|pfx|jar|properties)$|credentials\.|id_rsa'
# Must be 0 hits before commit.
```

### 7.5 Pre-commit mental checklist

After §7.1-§7.4 pass, do a final sanity sweep:

```
[ ] Atomic — one commit = one intent (no split required)              §0.1
[ ] Leaves repo green:                                                §0.2
    - tsc --noEmit (or your typechecker)
    - lint (zero errors on changed files)
    - build (when in doubt)
[ ] Why-over-what — body doesn't restate the diff                     §0.3
[ ] Imperative mood — summary in command form                         §0.4
[ ] Searchable — concrete nouns / paths / function names              §0.5
[ ] No secrets (§7.4 grep returns 0)
[ ] No console.log / debugger statements
[ ] Explicit path stage (not `git add -A`)                            §7.2
[ ] Subject ≤100 chars, body avg 16 lines, zero `##` headers          §1, §5
[ ] BREAKING CHANGE annotated if compatibility broken                 §6
[ ] Hook failure → fix and create NEW commit (do not amend)           §9
```

---

## 8. Trailers (Footers)

Last block of the body, separated by one blank line. Format `Token: value`. Parseable via `git interpret-trailers --parse`.

### 8.1 Standard trailers

| Token | Use | Example |
|---|---|---|
| `Co-authored-by:` | Pair / AI co-authoring | `Co-authored-by: Claude <noreply@anthropic.com>` |
| `Refs:` | Issue reference (no auto-close) | `Refs: #42` |
| `Closes:` / `Fixes:` | GitHub auto-close | `Closes: #42` |
| `Reviewed-by:` | Reviewer attribution | `Reviewed-by: Daniel <…>` |
| `Signed-off-by:` | DCO (Developer Certificate of Origin) | `Signed-off-by: Foo Bar <…>` |
| `BREAKING CHANGE:` | §6.2 | |

### 8.2 Project dialect trailers

Projects often define their own trailers — examples seen in real codebases:

- `Flag: billing-invoice-pdf-v2` — LaunchDarkly / GrowthBook feature flag named
- `Storybook: components-button--all-variants` — Storybook story link
- `Rollout: 5% → 50% → 100%` — phased rollout plan
- `Plan SC: ABC-001~004` — PDCA-style success-criteria reference
- `Design Ref: §2.1` — design-document section pointer
- `Match Rate: 98%` — gap-analysis match score
- `Reported-by: <name>` — kernel-style report attribution
- `Fixes: <hash>` — kernel-style fix-of-prior-commit pointer

These are not parsed by stock git tooling but are grep-friendly. Define yours in the project's `docs/references/COMMIT.md` and use them consistently.

### 8.3 Conventions

- One blank line between body and trailer block
- No blank lines inside the trailer block — tokens run consecutively
- `Signed-off-by:` only when the project enforces DCO

### 8.4 AI co-authoring (Claude / Copilot / Cursor / Gemini)

When an AI assistant materially shapes a commit, attribute via `Co-authored-by:` so credit and traceability survive in `git log` and `git shortlog`. This is increasingly common practice as agentic tools take a larger share of authorship.

**When to add the trailer**:

| Scenario | `Co-authored-by:`? |
|---|:--:|
| AI wrote ≥30% of the diff (substantial code generation, not just suggestions) | ✅ |
| AI proposed the architecture / approach that shaped the commit | ✅ |
| AI authored the commit message itself, with you accepting most of it | ✅ |
| AI auto-completed individual lines (Copilot inline suggestions) without architectural input | ❌ |
| AI explained an API or pointed at docs but you wrote the code | ❌ |
| AI ran a refactor command (e.g., rename) — purely mechanical | ❌ |

**Common identities** (use whichever your tool documents — the email is canonical for GitHub coauthor rendering):

```
Co-authored-by: Claude <noreply@anthropic.com>
Co-authored-by: GitHub Copilot <copilot@github.com>
Co-authored-by: Cursor <noreply@cursor.com>
Co-authored-by: Gemini Code Assist <noreply@google.com>
```

**Multiple AI tools or pair-with-AI**: list all coauthors, one per line:

```
Co-authored-by: Alex Kim <alex@example.com>
Co-authored-by: Claude <noreply@anthropic.com>
```

**Why this matters for AI agents reading history**: the `Co-authored-by:` trailer is one of the strongest signals for an AI agent rebuilding context after `/clear` to know that *another agent* shaped a commit. It informs review weight, debugging hypotheses ("hallucination might be the bug source"), and credit attribution. Skipping the trailer when AI authored substantial work is misleading to the next AI reader as much as to humans.

> **Org policy**: some companies require explicit AI attribution for compliance / IP review; some require it stay out of public history for IP-cleanroom reasons. Check your company policy and bake it into the project dialect.

---

## 9. Amend / Fixup Rules

`--amend` is dangerous. Stay strict:

| Situation | Allowed? | Alternative |
|---|:--:|---|
| Most recent commit, **before push** | ✅ | typo / forgotten file |
| Most recent commit, **after push** | ❌ | new commit (`fix(scope): ...` or `revert: ...`) |
| Older than HEAD | ❌ | new commit fix-up — never rewrite published history |
| Pre-commit hook failed | ❌ amend | hook failure means the commit didn't form — fix and create a fresh commit |

### 9.1 `fixup!` / `squash!` autosquash flow (optional)

Pre-merge interactive rebase:

```bash
git commit --fixup=<hash>           # message: "fixup! original summary"
git rebase -i --autosquash <base>
```

Use only on **unpublished** commits. Once pushed, treat history as immutable.

---

## 10. Reverts

Use `git revert <hash>` and keep the auto-generated message — but always add the **reason**.

### 10.1 Format

```
revert: feat(api): add idempotency keys to /v1/payments

This reverts commit 8a3b9c2d... .

Reason: idempotency-key TTL of 24h, combined with a Redis cluster
failover at 03:42 UTC, caused all replays in the 5-minute window
after failover to bypass deduplication and produce duplicate charges
(post-mortem #INC-2031). The fix is to persist replay records to
durable storage rather than Redis-only, which requires a schema
migration. Reverting until the durable-storage variant lands.

Refs: 8a3b9c2
Refs: #INC-2031
```

### 10.2 Rules

- Summary: `revert: <original summary>` (type=`revert`)
- First body line: `This reverts commit <full-hash>.`
- Then **mandatory reason** — what failed, why we backed out
- Original hash as `Refs:` trailer for cross-link

---

## 11. Anti-patterns

| Anti-pattern | Why bad | Replacement |
|---|---|---|
| `## ` headers in body | renders raw, 5/200 outlier in study | plain `Section:` label |
| Listing every file in body | noise — `git show --stat` covers it | concept-level grouping |
| 80+ line body for single-module change | reviewer fatigue, scope creep | split |
| `update files` / `wip` / `fix bug` / `improve form` (vague summary) | violates §0.5 — `git log --grep` blind, AI embedding noisy, blame meaningless | scope path + concrete noun + function/component name |
| Mixing concerns (auth fix + style + docs) | violates §0.1, breaks bisect | atomic commits |
| Native-language summary in monolingual prose | grep-unfriendly, AI cross-lingual cost | English + project-defined proper nouns only |
| **Partial native-language drift** — body/summary creeps into native prose beyond the dialect whitelist (e.g., `docs: lifecycle 책임 분리 + adapter parity` where only `친구톡`-style nouns are whitelisted) | LLM reading recent `git log` sees whitelisted native terms → over-generalizes to "native prose is OK" → drift compounds across cycles. Affects any locale where the dialect lists native proper nouns (ko / ja / zh / es / de / pt / ...) | Dialect MUST pair the whitelist with an explicit inverse rule ("default = English; native only for §1 whitelist"). Concrete drift examples per language help (책임 분리 → responsibility separation, 責任分離 → responsibility separation, separación de responsabilidades → responsibility separation). See §13.1 dialect guide + §13.6 verification |
| `--no-verify` / amending published commits | hook bypass / blame breakage | §9 — new commit fix-up |
| `git add -A` blanket staging | risks secrets and unrelated files | explicit path stage |
| Body restates the diff (WHAT only) | violates §0.3 | center the WHY |
| Past or progressive tense (`added`, `adding`) | violates §0.4 | imperative `add` |
| Missing BREAKING CHANGE annotation | callers unaware | §6 `!` or footer |
| WIP / "next commit will fix" | violates §0.2 | green build before commit |

---

## 12. Quick Reference

```
✅ § Principles (4 readers: log-scanner / blame / bisect / AI agent)
   atomic / leaves-repo-green / why-over-what / imperative / searchable

✅ § Format
   type(scope): summary           ← lowercase, ≤100 chars, imperative
   <blank>
   1-2 line context (why)
   <blank>
   - bullet (concept-level)
   - bullet
   <blank>
   Verification: ...              ← optional
   <blank>
   Token: value                   ← trailers (Refs / BREAKING CHANGE / dialect tokens)

✅ § Workflow (5-step)
   1. Inspect diff (git diff / --staged / status --porcelain)
   2. Stage (explicit path / glob / -p hunks / directory)
   3. Decide type (§7.3 tree)
   4. Secrets grep (§7.4)
   5. Pre-commit checklist (tsc + lint + build + console.log + atomic)

✅ § Auto-commit boundary signals (project-defined)
   Project dialect declares phase boundaries (e.g., PDCA Plan/Design/Do/Check/Act).
   Push / force / rebase / branch deletion remain user-triggered.

❌ ## headers / 80+ line single-module / "update"·"wip" / mixed concerns
❌ amending published commits / --no-verify / git add -A
❌ WHAT-only body / past tense summary / missing BREAKING annotation
```

---

## 13. Project Dialect

This skill provides the universal layer. Project-specific extensions live in `<project>/docs/references/COMMIT.md` (the **dialect** file).

### 13.1 What goes in the dialect

- **Domain proper nouns** — terms that should not be translated (product names, native-language identifiers, regulatory references)
- **Custom scope catalog** — actual high-frequency scopes derived from the project's own `git log` (use `scripts/analyze-history.sh`)
- **Custom trailers** — e.g., `Refs:`, `Closes:`, `Flag:`, `Storybook:`, `Rollout:`, `Plan SC:`, `Design Ref:`, `Match Rate:`, kernel-style `Fixes:` / `Reported-by:`. Whatever your team uses consistently.
- **Workflow integrations** — auto-commit boundary signals (PDCA phase completion, Linear ticket close, Jira transition, etc.)
- **Project-specific examples** — annotated commits from this repo's actual log
- **Type usage policy** — if `perf`/`ci`/`build` are unused, document it
- **Default language + explicit inverse rule** — if the dialect whitelists any native-language terms, state the default in the same section: "everything outside §1 is English." Whitelist-alone invites LLM over-generalization (§11 anti-pattern: *partial native-language drift*). Applies symmetrically to Korean / Japanese / Chinese / Spanish / German / Portuguese / any non-English domain vocabulary. The `templates/DIALECT.template.md` from v0.1.5 onward models this pairing as the §1 / §1.1 split — adopt that structure when authoring a new dialect.

### 13.2 What does NOT go in the dialect

- Anything in this skill's §0-§12 — the universal layer is the source of truth
- New principles — if you find a generalizable principle, contribute it back to the skill instead

### 13.3 Generating a dialect

```bash
~/.claude/skills/principled-git-commit/scripts/scaffold-dialect.sh
```

Interactive prompt:
1. Project name
2. Domain proper nouns (comma-separated)
3. Run `analyze-history.sh` to extract scope catalog from `git log`?
4. Workflow integration (PDCA / Linear / Jira / none)
5. Custom trailers

Drops `docs/references/COMMIT.md` populated from `templates/DIALECT.template.md`.

### 13.4 Loading order at runtime

When this skill triggers:

1. Load `SKILL.md` (this file) — universal layer
2. Detect `<project>/docs/references/COMMIT.md` — if present, load and apply on top
3. If no dialect exists and the user is about to commit substantial work, offer to scaffold

### 13.5 Org-level dialects (multi-repo organizations)

Companies with N repos that share trailers / scopes / workflow conventions can layer **three** levels instead of two:

```
~/.claude/skills/principled-git-commit/SKILL.md   ← universal (this skill)
$ORG_HOME/COMMIT.md                               ← org-level dialect (intranet / template repo)
<project>/docs/references/COMMIT.md               ← project-level dialect
```

**How to set up**:

1. Maintain an `acme-eng/commit-dialect` GitHub repo (or similar) with the org-wide `COMMIT.md`.
2. New repos clone that file into `docs/references/COMMIT.md` as a starting template, then add project-specific extensions on top.
3. Org changes (e.g., "we now require `Refs:` for all features") propagate via PRs that copy the updated `COMMIT.md` into each repo.

**What goes in org-level dialect**:

- Common trailers all repos use (e.g., Linear ticket prefix, security-review trailer, AI co-author policy)
- Universal scope catalog (e.g., `apps/`, `packages/`, `infra/` shape if monorepo standard)
- AI attribution policy (when `Co-authored-by:` is required org-wide)
- Squash-merge policy if uniformly adopted

**What stays project-level**:

- Domain proper nouns specific to this product
- Scope catalog from this repo's actual `git log`
- Project-specific examples and anti-patterns

**Override rules**: project-level beats org-level beats universal. Each layer adds, never silently removes — if a project disables an org rule, it documents the disable explicitly in §5 Type Usage Policy or the relevant section.

### 13.6 Verifying dialect enforcement

Even with the structure in §13.1 (whitelist + inverse rule paired), the rules only matter if (a) commits actually follow them and (b) the LLM doesn't silently drift between sessions. Three-layer verification:

1. **Static check — does the dialect have the inverse rule at all?**
   ```bash
   # From repo root. Exits 0 if pattern found.
   grep -Ei "(default.*english|english.*default|whitelist.*only|inverse rule)" docs/references/COMMIT.md
   ```
   Missing → add the §1 / §1.1 split per `templates/DIALECT.template.md`.

2. **Runtime check — recent commits for drift symptoms.**
   ```bash
   # Show last 30 commit subjects; flag rows with non-ASCII (likely native-language drift).
   git log --format='%h %s' -30 | LC_ALL=C grep -P '[^\x00-\x7f]' || echo "(all ASCII — no drift)"
   ```
   Hits are not all violations (whitelisted proper nouns are fine) — review each to confirm it's `친구톡`-style and not `책임 분리`-style prose. Patterns recurring without whitelist hits indicate drift.

3. **Hook layer (project-local, optional)** — a `.claude/hooks/` UserPromptSubmit hook that injects a reminder when "commit"-class keywords appear in user prompts. Reminder-only (`exit 0`), no blocking. Pairs the SKILL invocation requirement with the dialect language rule so the LLM cannot bypass either by reading recent `git log` alone. See `references/` for an example template.

The static check belongs in CI when team size > 1; the runtime check is a periodic audit (monthly or per release); the hook is per-developer.

---

## 14. Source Attribution

- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) — type/scope/body format, `!` and `BREAKING CHANGE:` notation
- Tim Pope, ["A Note About Git Commit Messages"](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html) — imperative mood, summary/body structure
- `github/awesome-copilot@git-commit` (29.6K installs) — workflow steps, type detection, secrets blocklist
- Empirical 200-commit study (private codebase, 2026-05) — length sweet spots (avg 71-char summary, 16-line body), `##`-header outlier observation (5/200 = 2.5% of corpus), sub-scope (`/`) and multi-scope (`,`) conventions, module-tag pattern in long-running cycles

---

## Version

| Version | Date | Notes |
|---------|------|-------|
| 0.1.0 | 2026-05-10 | Initial release. Universal extraction from a private project's `docs/references/COMMIT.md` v0.4 (200-commit empirical study). Project-specific facts (PDCA workflow integration, native-language proper nouns, scope catalog) moved out to project dialect (see §13). |
| 0.1.1 | 2026-05-11 | Rename `commit-skill` → `principled-git-commit`. Frontmatter `name:`, install paths, scaffold script, DIALECT template, README all updated. Skill content (§0-§14) unchanged. |
| 0.1.2 | 2026-05-11 | Genericize all examples — replace project-specific commit examples with patterns drawn from real-world open-source operating models (Stripe-style idempotency keys, bcrypt→argon2id migration, idempotent payments revert, kernel-style mm/oom_kill long-form, monorepo `packages/ui` scopes, feature-flag rollout). `examples/good-commits.md` reorganized into 12 categories (A: single-line / B: why-driven / C: features / D: refactors / E: perf with metrics / F: breaking / G: reverts / H: multi-author + AI co-author / I: test/chore/docs/build/ci / J: kernel-style long-form / K: monorepo + feature-flag / L: anti-patterns). `templates/DIALECT.example.md` swapped from a single project to a fictional "Acme Cloud" pnpm monorepo + Linear + LaunchDarkly + squash-merge example. `templates/DIALECT.template.md` placeholder examples diversified (PDCA / squash-merge / trunk-based-with-flags workflows; brand names + native-language regulatory terms + service names). `scaffold-dialect.sh` prompt updated. SKILL.md inline examples (§0.4 / §0.5 / §2 type table / §3.4 feature-scope / §4 length / §8 trailer examples / §10 revert) replaced with generic real-world patterns. References to one specific source repo softened to "private-project 200-commit study" while preserving the empirical attribution. |
| 0.1.3 | 2026-05-11 | Coverage expansion: §1.3 Three commit surfaces (commit / PR title / squash body — different audiences, same principles), §3.5 Non-Conventional formats acknowledgment (kernel-style / bracket-prefix / Mozilla bug-numbered / no-prefix imperative / Jira-id all valid — §0 principles still apply), §8.4 AI co-authoring policy (when to add `Co-authored-by:` for Claude / Copilot / Cursor / Gemini, with substantial-vs-trivial decision matrix), §13.5 Org-level dialects (3-layer model: universal skill + org-level COMMIT.md + project-level COMMIT.md, with override rules). |
| 0.1.4 | 2026-05-11 | Drop premature CI + CONTRIBUTING.md from v0.1.3 — single-contributor repo with no branch protection got ~10KB of pollution per install for symbolic "eat-our-own-dogfood" value with effectively zero ROI. `scripts/validate-commit-msg.sh` stays (usable as a local commit-msg hook regardless of repo CI presence). |
| 0.1.5 | 2026-05-26 | Add §11 anti-pattern "Partial native-language drift" + §13.1 "Default language + explicit inverse rule" sub-bullet + §13.6 "Verifying dialect enforcement" (3-layer: static grep / runtime non-ASCII subject scan / project-local UserPromptSubmit hook). `templates/DIALECT.template.md` §1 restructured into §1 (Language Default & Domain Proper Nouns — explicit English-default rule + ko/ja/es drift examples) + §1.1 (Whitelist table). `lang/ko/SKILL.md` v0.1.5 note added for Korean dialect users (highest whitelist surface area among locales). Motivated by a Korean-project real case where 11 whitelisted proper nouns + missing inverse rule produced summaries like `docs(refs,adr): post-hotfix lifecycle 책임 분리 + adapter parity`. Language-agnostic — applies to ko/ja/zh/es/de/pt/etc. projects equally. No script change (scaffold-dialect.sh `{{DOMAIN_NOUNS}}` placeholder carries into new §1.1 location automatically). |

---
> Source: [rhino-ty/principled-git-commit](https://github.com/rhino-ty/principled-git-commit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
