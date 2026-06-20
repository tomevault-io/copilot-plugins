## handoff

> Write a structured session handoff (Goal / Done / Open loops / Gotchas) and append it to a ledger so the user can `/clear` and pick up cleanly in the next session — same project, or a sibling session on the side. Use whenever the user says `/handoff`, "write a handoff", "wrap this up for the next session", "hand this off", or wants to capture current work state before clearing context. **Steering**: any free text after `/handoff` (that isn't `pickup` or `list`) is a directive that shapes composition — `/handoff pay special attention to the auth migration`, `/handoff for a designer, skip code details`, `/handoff this is paused, emphasize the blocker`, `/handoff call this auth-migration` (slug override). ALSO use in **pickup mode** whenever the user says "pick up the handoff and continue", "carry over from handoff", "resume from handoff", "/handoff pickup", or "/handoff pickup <description>" — read the ledger, resolve the right handoff (cwd-default or fuzzy match), prime context, mark resumed, continue work. **List mode**: "/handoff list" prints recent entries. The skill is the durable counterpart to `/compact` and the explicit alternative to `/clear` — preserves intent and open loops, not just history.


# Handoff — Session Continuity Skill

`/handoff` is the bridge between sessions. It produces a structured, durable artifact (4 sections: Goal, Done, Open loops, Gotchas) and registers it in a ledger so any later session — same project or a sibling — can resume without remembering file paths.

Pipeline at a glance:

```
session A:  /handoff                              → writes file + appends ledger
session A:  /clear
session A:  "pick up the handoff and continue"    → pickup auto-fires (cwd match)

session B:  /handoff pickup design-system tokens  → fuzzy match across ledger
```

Handoffs are **ephemeral, per-session state**: what was done, what's still to do, why. Durable facts (project conventions, credentials, long-lived references) belong elsewhere — in a README, an ADR, or Claude Code memory.

---

## Mode dispatch

Parse the invocation. Default is **write**. The first token after `/handoff` determines routing: reserved keywords `pickup` and `list` route to those modes; anything else is captured as a **steering directive** for write mode (see "Interpreting steering" below).

| Invocation                                                                                       | Mode              |
| ------------------------------------------------------------------------------------------------ | ----------------- |
| `/handoff`                                                                                       | write             |
| `/handoff <free text>` — e.g. "pay special attention to X", "for a designer"                     | write (steered)   |
| "write a handoff", "wrap this up for the next session", "hand this off"                          | write             |
| `/handoff pickup`, `/handoff pickup <desc>`                                                      | pickup            |
| "pick up the handoff and continue", "carry over from handoff", "resume from handoff"             | pickup            |
| `/handoff list`, "show recent handoffs"                                                          | list              |

If the invocation is ambiguous, ask one short question and proceed.

---

## Paths

The skill writes under a single base directory:

```
$HANDOFFS_DIR  (env override, optional)
   ↓ default
~/.claude/handoffs/
   ├── .ledger.json                         ← single source of truth
   ├── {project}/{date}-{slug}.md           ← one file per handoff
   └── general/                             ← fallback when cwd has no project
```

If the `HANDOFFS_DIR` environment variable is set, honor it. Otherwise default to `~/.claude/handoffs/`. The skill creates the base dir and ledger on first run if missing.

---

## Write mode

### Step 0 — Capture the steering directive (if any)

Parse the invocation. If the first token after `/handoff` is **not** a reserved keyword (`pickup`, `list`), treat the entire trimmed argument string as **steering text** (the `directive`). Empty → no steering, behave as before.

Steering will be:
1. **Applied during composition** (see "Interpreting steering directives" below)
2. **Recorded** as a `Steering:` metadata line in the document header (when non-empty)
3. **Stored** as `directive` in the ledger entry (when non-empty)
4. **Echoed** in the final report-back so the user can verify it was understood

Examples:
- `/handoff pay special attention to the auth migration steps`
- `/handoff for a designer — skip code details, focus on visual decisions`
- `/handoff this is paused, not done; emphasize the blocker`
- `/handoff call this auth-migration` (overrides slug inference)
- `/handoff short` (caps each section at 2–3 bullets)

### Step 1 — Infer project from cwd

Apply these rules **in order** and stop at the first match:

1. If cwd is inside a git repository, project = basename of `git rev-parse --show-toplevel` (lowercased).
2. Otherwise, project = basename of cwd, lowercased.
3. If cwd is `$HOME`, `/`, or otherwise generic, project = `general`.

Print the inferred project to the user in one line. Don't ask for confirmation unless rule 3 fires; then offer the inference and let the user override.

### Step 2 — Pick a slug

From the **current conversation**, choose a short kebab-case slug (2–4 tokens) that describes the work. Examples:

- Dev: `auth-flow-bug`, `db-migration-rollback`, `login-rate-limit`
- Marketing: `q4-newsletter`, `pricing-page-copy`, `launch-announcement`
- Design: `dark-mode-tokens`, `nav-redesign`, `onboarding-illustrations`

If the slug is genuinely unclear (very mixed session), pick something generic like `mixed` or `multi-thread`.

**Steering override:** if the directive contains "call this `<slug>`" or "name it `<slug>`", use that slug verbatim (kebab-case it if needed).

### Step 2.5 — Audit the session

Before drafting, sweep the session and **classify each thing it touched**. This forces enumeration over stream-of-consciousness drafting and is the single biggest lever against both bloat and omissions.

**Source taxonomy** — sweep in this order:

| Source                                                          | Default treatment                                                  |
| --------------------------------------------------------------- | ------------------------------------------------------------------ |
| User-stated intent (original ask + mid-session edits)           | **Load-bearing** → Goal                                            |
| Decisions made (technical / UX / scope)                         | **Load-bearing** → Done if shipped, Open loops if pending          |
| Files actually changed (Edit / Write)                           | **Mention by path** — one bullet, not a narrative                  |
| Files only investigated (Read / Grep)                           | **Omit by default** — include only if a finding mattered           |
| Open questions / blockers                                       | **Load-bearing** → Open loops                                      |
| Dead ends ruled out                                             | **Load-bearing** → Gotchas, one line each                          |
| User asides ("by the way…", "actually…", "don't worry about X") | **High-value, easy to miss** — sweep the transcript explicitly     |
| External resources (URLs, tickets, dashboards)                  | **Link, don't restate** — one inline ref                           |

**Calibrate to session size** — don't pad small work; don't strip large work:

| Session shape                                       | Handoff shape                                                                                |
| --------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Trivial (≤ 5 substantive changes, single decision)  | **Mini** — Goal + Done only, ≤ 5 bullets total. Skip empty sections.                          |
| Standard (focused session, one workstream)          | **Full** — all 4 sections, modest length                                                      |
| Sprawling (multi-thread, exploratory, long)         | **Full + dense Gotchas** — gotchas usually carry the most signal here                         |
| Paused / blocked mid-work                           | **Full + foreground the blocker** — Goal mentions paused state; Open loop #1 is the blocker  |

**Identify where value concentrates by session type** — different session shapes load different sections. Spend draft-time proportionally:

| Session type                  | Where value concentrates                                                          | Where to under-invest                  |
| ----------------------------- | --------------------------------------------------------------------------------- | -------------------------------------- |
| Bug fix                       | **Gotchas** — the *why* and the dead ends                                         | Done (one line + line ref is usually enough) |
| Refactor                      | **Done** (shape of changes) + **Open loops** (callers not yet migrated)           | Gotchas (often genuinely sparse)       |
| Exploration / spike           | **Gotchas** (what didn't work and why) + **Open loops** (recommended next path)   | Done (often "(none — exploration)")    |
| New feature build             | **Done** (scope shipped) + **Open loops** (cut scope, follow-ups)                 | Gotchas (light unless something surprised) |
| Design / copy / content       | **Done** (decisions taken, with rationale) + **Gotchas** (alternatives rejected)  | Open loops (often just review/sign-off) |
| Skill / tool / infra build    | **Done** + **Gotchas** (design choices that aren't in the code)                   | balanced                                |
| Paused / blocked              | **Goal** foregrounds the blocker; **Open loop #1** is the unblock action          | Don't pad Done to fake completeness    |

Use this to weight emphasis — *not* to skip sections. Every handoff still has all four headers; the type just tells you where to lean.

**Route durable facts before composing** — durable facts are things that survive 5+ sessions: project conventions, credentials, long-lived references, architectural decisions. These do **not** belong in a handoff (which is ephemeral and gets buried after pickup). Route them to their permanent home *before* drafting:

| Durable fact                              | Permanent home                                  |
| ----------------------------------------- | ----------------------------------------------- |
| Project structure, conventions, "always X" | CLAUDE.md or memory                            |
| Credentials, URLs, dashboard locations    | memory (`reference-*.md`)                       |
| Architectural decisions ("we chose X over Y because…") | ADR or project README                |
| "User prefers X" / "always do Y"          | memory (`feedback-*.md`)                        |
| Long-lived project goals                  | project doc or memory (`project-*.md`)          |

If during the source sweep you notice a durable fact, write it to its permanent home *now*, then either omit from the handoff or leave a single pointer line (`per [[memory-slug]]…`). A handoff that restates durable facts is a handoff that wastes the reader's attention and rots.

### Step 3 — Compose the document

**Inclusion / exclusion tests** — apply these to every candidate bullet:

A bullet **earns its place** if at least one is true:
- The next session wastes time without it (load-bearing)
- It captures intent or a constraint not visible in the code (insight)
- It's a surprise that would catch the next session off-guard (gotcha)
- It's a dead end ruled out (saves future repetition)

A bullet **gets cut** if any are true:
- The next session could derive it from `git status` / `git log` / a 30-second skim of changed files
- It describes Claude's *process* (keep only the *settled* fact)
- It's about a file the session touched but didn't change meaningfully
- It restates the conversation transcript

The bar isn't "is this true?" — it's "does the next session **need to know** this?"

Use this **exact template**. Every section is required, even if a section is `(none this session)`.

```markdown
# Handoff: {Project} — {Slug}

- **Project:** {project}
- **Date:** {YYYY-MM-DD}
- **Cwd:** {absolute path at write time}
- **Status:** open
- **Steering:** {directive}    ← include this line ONLY when steering was provided

## Goal
*One paragraph, ≤ 3 sentences.* What we set out to do, and why. Frame *intent* before *state* so the next session understands the aim, not just the breadcrumbs. Mention the original ask verbatim if useful.

## Done
*5–10 bullets, each ≤ 25 words. If you have 15+, you're listing process, not outcomes.*
- Bulleted artifacts shipped, decisions taken, files changed (with paths + line refs where it helps).
- Each item is a complete sentence — the reader picks up cold and may never read the diff.
- Reference commits by SHA when relevant.

## Open loops
*3–7 items, numbered, priority order.*
1. Next action first.
2. Each with enough context to start without re-reading the full diff.
3. Note blockers, dependencies, or "needs design review" / "needs PM input" explicitly.
4. If nothing is open, write `(none — work is complete)`.

## Gotchas & context
*2–6 items. Quality > volume; one sharp gotcha > four mild ones.*
- Surprises that cost time; dead ends already ruled out; constraints discovered mid-session.
- Anything future-you would want to know but isn't obvious from the code or files.
- External resources (issue links, doc URLs, dashboards) that became load-bearing.
```

**Drafting rules:**
- **Goal** is one paragraph, not bullets. It carries intent.
- **Done** is past-tense bullets with file paths. Be specific: `src/auth/login.ts:42` beats "fixed login".
- **Open loops** is numbered and ordered — the top item is what the next session should pick up first.
- **Gotchas** is the most valuable section. Things that aren't in the diff: dead ends, surprises, hidden constraints, preferences expressed mid-session.
- Don't pad. If a section is genuinely empty, say so in one line.

### Step 3.25 — Antipattern catalog

Before self-critique, scan your draft for these named failure shapes. Each has a recognizable signature and a one-line fix. This is the dual to the inclusion tests — concrete shapes the model can pattern-match, not abstract principles.

| Antipattern               | Signature                                                                 | Fix                                                                                       |
| ------------------------- | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| **Fat bullet**            | One bullet contains 3+ distinct facts, often ≥30 words, often a paragraph | Split into 2–3 separate bullets, or use a header + sub-bullets if the items are related   |
| **Process narrative**     | Bullets describe what Claude *did* (read X, tried Y, found Z)             | Keep only the settled fact: the change, the decision, the dead end                        |
| **Transcript restatement**| Bullet repeats something visible in the conversation history              | Cut. The reader doesn't need to relive the session                                        |
| **Durable-fact leakage**  | Bullet states a fact that lives (or should live) in memory/CLAUDE.md/ADR  | Route to permanent home (Step 2.5), then drop or pointer-link with `per [[memory-slug]]…` |
| **Implementation trivia** | Code-internal detail (selector tricks, keyframe %ages, regex patterns)    | Belongs in code comments. Keep only if it would catch the next session off guard          |
| **Stretch Open loop**     | Loop item is a maintenance/polish task padding to hit the 3–7 budget      | Cut. 2 sharp loops beat 5 mushy ones. The budget is a ceiling, not a floor                |
| **Vague Gotcha**          | "Be careful with X" / "watch out for Y" without specifics                 | Name the failure mode and what it cost (`~20m on this`); otherwise cut                    |
| **Padded Goal**           | Goal paragraph restates the Done section in narrative form                | Goal = *intent*, not summary. If you can derive Goal from Done, the Goal is wrong         |

A good self-check: read each bullet and name the antipattern it might fit. If you can't name one, the bullet is probably load-bearing. If you can, apply the fix.

### Step 3.5 — Self-critique before writing

After drafting, re-read your draft with **two adversarial lenses, in order**. This catches both omissions and bloat — and is the second-biggest lever (after Step 2.5) for quality.

**Lens 1 — completeness** ("what's missing?"):
- Did the user say anything mid-session that isn't reflected anywhere?
- Are there decisions in the diff whose *why* I haven't captured?
- Did I rule out any approaches that aren't documented as dead ends?
- Is there a constraint discovered late that the next session would re-discover?

**Lens 2 — bloat** ("what's redundant?"):
- For each Done bullet: could the next session derive this from `git diff` in under 60s? If yes → cut or compress.
- For each section: am I padding to hit a budget? Cut. Budgets are ceilings, not floors.
- Any sentence that describes process rather than outcome → cut.
- Scan against the **antipattern catalog** (Step 3.25): can you name a pattern for any bullet? If yes, apply the fix.

Concrete bloat shapes to look for:
- A Done bullet >30 words = probably a **fat bullet**; split it
- A Gotcha that starts "Be careful with…" = probably **vague gotcha**; name the failure or cut
- A bullet that names a memory slug, repo, or credential = probably **durable-fact leakage**; pointer-link or drop
- A Goal paragraph that lists what was done = probably **padded Goal**; rewrite to capture *intent*

Apply edits in place. Only then proceed to Step 4.

### Step 4 — Write the file

Compute `date=$(date +%Y-%m-%d)`. Ensure the project dir exists:

```bash
mkdir -p "${HANDOFFS_DIR:-$HOME/.claude/handoffs}/{project}/"
```

Write the file at `{base}/{project}/{date}-{slug}.md`.

If a file with the same name already exists today (rare: two handoffs same day, same slug), append a `-2`, `-3` suffix to the slug.

### Step 5 — Append to ledger

The ledger lives at `{base}/.ledger.json`. If it doesn't exist, initialize as `{"version": 1, "handoffs": []}`.

Append a new entry to the `handoffs` array:

```json
{
  "id": "{project}-{date}-{slug}",
  "path": "{project}/{date}-{slug}.md",
  "project": "{project}",
  "slug": "{slug}",
  "summary": "{one-line summary — half about Done, half about Open loops}",
  "tags": ["{tag1}", "{tag2}"],
  "directive": "{steering text, or null if none}",
  "cwd": "{absolute cwd}",
  "created": "{ISO 8601 with TZ offset, e.g. 2026-05-18T15:32:00+0300}",
  "status": "open",
  "picked_up_at": null
}
```

- `summary` is ~120 chars max — must convey both what was done and what remains so fuzzy match and `/handoff list` are useful.
- `tags` are 2–5 tokens that help fuzzy match (project subsystem, framework, file area, role).
- For the timestamp, prefer `date +"%Y-%m-%dT%H:%M:%S%z"` (portable across macOS BSD `date` and GNU `date`).

**Atomic write** (avoids corruption if two sessions race):

1. Read current ledger JSON.
2. Compute new ledger object with appended entry.
3. Write to `{base}/.ledger.json.tmp`.
4. `mv` the tmp file over `.ledger.json`.

Use Bash for the rename so the OS-level atomic guarantee applies. Do not edit the ledger in place.

### Step 6 — Report back

Print to the user, no fluff:

```
Handoff written:
  path:    {base}/{project}/{date}-{slug}.md
  id:      {id}
  steered: {directive}        ← print this line ONLY when steering was provided

Next: /clear, then "pick up the handoff and continue".
```

The `steered:` echo lets the user verify Claude understood the directive before they `/clear`.

### Worked example: thin and decision-rich, not bloated

**Bad** (process narrative, restates the diff):

```markdown
## Done
- Looked at auth/login.ts to understand the flow
- Read auth/session.ts and auth/cookie.ts
- Tried clearing the token cookie before re-auth — didn't work first time
- Then tried setting Max-Age=0, also didn't work
- Finally found that the cookie needs httpOnly + sameSite reset
- Made changes to auth/login.ts:84
```

*Why it's bad:* six bullets, all derivable from `git diff` + the conversation transcript. No insight. The next session learns nothing it couldn't get in 30 seconds.

**Good** (thin Done, decisions visible, gotchas earn their keep):

```markdown
## Done
- Fix shipped at auth/login.ts:84 — stale token cookie now cleared before re-auth.

## Gotchas
- httpOnly + sameSite must both reset; clearing Max-Age alone leaves the cookie. Spent ~20m on this.
- Loop only fires on stale-token + SSO; password login never sees it. Easy to miss in testing.
```

*Why it's good:* Done is a single load-bearing line pointing at the change. Gotchas carry the time-saving knowledge — the next session avoids the 20-minute dead end *and* knows where to test.

### Worked example: sprawling session, multiple threads

A long iterative session (methodology + visual artifact + memory updates) is where bloat compounds. Most failure modes show up here.

**Bad** (fat bullets, durable-fact leakage, implementation trivia):

```markdown
## Done
- v2 methodology (commit `73a1035`): Step 2.5 (source taxonomy + adapt-to-size table), Step 3 inclusion/exclusion tests, soft section budgets in template, Step 3.5 self-critique pass (completeness + bloat lenses), one worked-example pair inline.
- Personal HTML explainer at `~/.claude/handoffs/howitworks.html` — iterated extensively: A/B lanes, 3-pill comparison row with `/handoff` accented, animated 8s context-fill loop with smooth color shifts (green→amber→red via animated `background-color` in keyframes, flat-right-edge via `overflow: hidden` parent + `border-radius: 0` fill), Structure section showing global ledger + per-project tree.

## Gotchas
- Repo is web-app (NOT design-system).
- Gauge CSS: `.gauge-bar { overflow: hidden }` + `.gauge-fill { border-radius: 0 }` produces flat-right-edge.
- Animation timing tightly coupled: contextLoop drives bar, pickupFlash/handoffFlash/clearFlash swap chips at 56–62% and 86–90%.
```

*Why it's bad:*
- **Fat bullet** ×2: bullet 1 jams 5 sub-features in one line; bullet 2 is a paragraph.
- **Durable-fact leakage**: "Repo is web-app" lives in memory already.
- **Implementation trivia**: gauge CSS selectors and keyframe percentages belong in the code, not the handoff.

**Good** (split, routed, compressed):

```markdown
## Done
- Composition methodology shipped (commit `73a1035`):
  - Step 2.5: source taxonomy + adapt-to-size table
  - Step 3: inclusion/exclusion tests
  - Step 3.5: self-critique pass
  - One worked-example pair
- Visual explainer at `docs/explainer.html` — local working copy, not committed.

## Gotchas
- v2 methodology is **unproven against real handoffs** — added on first-principles reasoning. Test on a real session before further tuning.
- Viz animation uses one 8s shared keyframe cycle; chip swaps depend on bar timing. Changing one timing requires updating all three (see code comments at the keyframes).
```

*Why it's good:*
- Fat bullet split into header + sub-bullets — same information, scannable.
- Long viz paragraph cut to two load-bearing facts (location + scope).
- Durable fact ("repo is web-app") routed out entirely — memory already has it.
- CSS trivia dropped; timing-coupling gotcha kept because it would actually catch the next session off-guard, but pointer-linked to code instead of inlined.

---

## Interpreting steering directives

Steering is open-ended free text. Read it once, classify intent, then apply during composition. Common shapes:

| Directive shape                                  | Effect on draft                                                          |
| ------------------------------------------------ | ------------------------------------------------------------------------ |
| "focus on X" / "pay attention to X"              | Expand Done/Open/Gotchas with X-related items; demote unrelated threads to brief mentions or a single line |
| "for `<audience>`" / "write it for `<role>`"     | Match register (designer = plain language + visual choices; junior dev = explain assumptions; PM = outcomes over implementation) |
| "skip X" / "leave out X" / "no X"                | Omit X entirely; note the omission in one Gotchas line so the next session knows X exists but was filtered |
| "short" / "tight" / "brief" / "TL;DR"            | Cap each section at 2–3 bullets; Goal one sentence                       |
| "this is paused" / "this is blocked"             | Foreground the blocker in Goal AND as Open loop #1; mark Status implicitly |
| "call this `<slug>`" / "name it `<slug>`"        | Override Claude's inferred slug                                          |
| "more gotchas" / "emphasize the surprises"       | Expand Gotchas section; compress Done                                    |

If the directive doesn't match a known shape, **apply best-effort** and surface what you did in the report-back (`steered:` line stays verbatim; nothing to translate).

Steering doesn't *replace* Claude's judgment — it weights it. Sections still appear in the standard order with the standard headers; only the *content* shifts.

---

## Pickup mode

### Step 1 — Parse the description

- `/handoff pickup` (no args) → description is empty.
- `/handoff pickup <words>` → description = `<words>`.
- Natural phrasing → description is whatever extra words the user added. "pick up the handoff and continue" → empty. "pick up the design tokens handoff" → `design tokens`.

### Step 2 — Resolve the target

Read `{base}/.ledger.json`. Apply in order:

1. **Empty description + cwd has a real project** (not `general`):
   - Filter to entries with `project == inferred_project`.
   - Prefer `status == "open"`. If none open, fall back to most recent regardless of status.
   - Pick the most recent by `created`.
   - **Confirm in one line** before reading: `"Picking up: {id} — {summary}. Continue? [Y/n]"`. If the user says no, list candidates.

2. **Empty description + generic cwd**:
   - List the 5 most recent open entries with `id`, `project`, and `summary`.
   - Ask the user to pick one.

3. **Non-empty description**:
   - Score each ledger entry by token overlap with `project + slug + tags + summary + directive`. Lowercase, split on whitespace and hyphens, strip stopwords. The `directive` field carries the user's original steering text — including it lets pickup queries find handoffs by the *focus* the user named at write time, not just the slug.
   - If the top score is at least 1.5× the second score, pick it.
   - Otherwise list the top 3 and ask the user to pick.

### Step 3 — Load and summarize

Read the handoff file. Print to the user a tight 4–5 line summary:

```
Resumed: {id}
Goal:    {one-line condensed from Goal section}
Open:    {first 2 open loops, semicolon-separated}
Gotcha:  {most relevant gotcha if any}
```

Then continue the work — pick up at the top Open loop unless the user redirects.

### Step 4 — Mark resumed in the ledger

Update the entry in place:

- `status = "resumed"`
- `picked_up_at = <ISO 8601 timestamp>`

Atomic write the same way as in Write mode (write to `.tmp`, `mv` over).

A `resumed` entry can be picked up again later — pickup does not lock or hide it.

---

## List mode

`/handoff list` → print recent ledger entries. Default: 10 most recent.

Args (optional, parse loosely):
- `--all` → show everything
- `--project <X>` → filter to one project
- `--open` → only `status == "open"`

Format as a fixed-width code block. Prefix the summary with `▸ ` when the entry has a non-null `directive` — a visual hint that the handoff was steered:

```
ID                                       PROJECT          STATUS    SUMMARY
web-app-2026-05-18-auth-flow-bug         web-app          open      ▸ Fixed redirect loop; open: SSO test coverage
design-system-2026-05-17-dark-tokens     design-system    resumed     Added dark-mode tokens; open: component audit
marketing-site-2026-05-15-pricing-copy   marketing-site   open      ▸ Drafted pricing variants; open: legal review
```

Truncate `SUMMARY` (after the optional `▸ ` prefix) to ~50 chars in the table; the full text and directive live in the ledger entry and the file.

---

## Implementation rules

- **Always use atomic write** for the ledger. In-place edits corrupt under concurrent access.
- **Timestamps** are ISO 8601 with timezone offset. `date +"%Y-%m-%dT%H:%M:%S%z"` works on macOS and Linux without extra flags.
- **Paths in the ledger** are relative to the base directory. Keeps the ledger portable across machines.
- **Don't reread the file you just wrote** — the harness tracks file state; reading wastes context.
- **Don't ask permission for read-only actions** (project inference, ledger read, fuzzy match). Confirm only before *writing* the file or marking resumed.
- **One handoff per slug per day.** If you'd collide, suffix `-2`, `-3`.
- **Empty sections** get a one-line `(none this session)` placeholder. Never silently omit a section — the template's shape is the value.
- **Durable facts route at Step 2.5**, not at compose time. If you catch one mid-draft, fix it: write to its permanent home and either drop or pointer-link in the handoff.

---

## Verification

After install, run the full loop once:

1. `cd` into any git repo → `/handoff`. File should appear at `~/.claude/handoffs/{repo-name}/{date}-{slug}.md`; ledger gains an entry with matching id, project, `status: "open"`.
2. `/clear` (manual), then in the same cwd say "pick up the handoff and continue" → skill auto-fires, resolves the just-written entry, prints summary, sets `status: "resumed"`.
3. From an unrelated cwd: `/handoff pickup <a few words from the slug>` → fuzzy match returns the same entry.
4. `/handoff list` → shows recent entries with id, project, status, summary.

If any step asks for a file path explicitly, the skill is failing its core promise — fix the inference or fuzzy logic, don't paper over with a prompt.

---
> Source: [GitJarno/handoff](https://github.com/GitJarno/handoff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
