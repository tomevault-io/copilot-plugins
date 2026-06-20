## design-and-ship

> Author a rich, mobile-responsive HTML design doc for a code change, get the user's sign-off, merge the doc's predicted ask-gates (commands risky enough to confirm) into the permission config, isolate the work in a git worktree, execute the change step by step against the doc, then append a "Roadblocks & deviations" recap to the same HTML file. Trigger on requests for a design doc, technical spec, RFC, implementation plan, architecture sketch, or runbook before shipping a non-trivial change — even when the user doesn't say "design doc" but says things like "plan this", "spec it out", "write up how we'd do X", "design a doc for X then ship it", "make a plan with diagrams", or asks for a doc with architecture diagrams, an agent runbook, or a Cypress test plan. Also use whenever a task is large enough that the user would benefit from sign-off before code starts and a recap after it lands.


# Design and Ship

A three-phase workflow for non-trivial changes:

1. **Design** — author a self-contained HTML design doc with diagrams, pseudocode (not full source), the ask-gates (commands risky enough to confirm before running), and an agent runbook
2. **Ship** — after user sign-off, merge the ask-gates into `~/.claude/settings.json → permissions.ask`, enter a git worktree, execute the doc
3. **Recap** — append a "Roadblocks & deviations" section to the same doc, recording what actually happened

The reason for this shape: design docs that go stale during implementation are worse than no doc. By writing the recap into the same file, the doc stays accurate, the user gets a paper trail of decisions, and future work has a starting reference that matches reality.

---

## Phase 1 — Design

### Where the file lives

- Output path: `docs/<topic>-design-doc.html` inside the project repo (not the home folder, not `/tmp`)
- If the repo has no `docs/` directory, create one. Don't dump the file at the repo root
- Filename uses kebab-case and ends with `-design-doc.html` so it's grep-able

### Starting from the template

Read `assets/template.html` from this skill. It already contains the full CSS, the sticky-header, the layout grid, the sidebar `<aside class="outline">`, the inline `<nav class="toc">`, the Mermaid loader, the theme toggle, and the scrollspy. **Do not rewrite the CSS or scripts.** Replace only the marked placeholders:

- `__TITLE__` — page title (e.g., `Acme — Auth Design Doc`)
- `__SUBTITLE__` — short tagline (e.g., "Google OAuth + Email/Password")
- `__BRAND_LABEL__` — short label after the brand dot (e.g., "Auth Design")
- `__LEDE__` — one paragraph for the hero
- `__PILLS__` — `<span class="pill ...">…</span>` chips
- `__TOC_LIST__` — `<li>` entries for the inline TOC (mobile)
- `__OUTLINE_LINKS__` — `<a data-href="…">` entries for the sidebar (desktop)
- `__SECTIONS__` — the actual numbered sections
- `__THEME_STORAGE_KEY__` — distinct localStorage key (`dq-<topic>-theme`)
- `__FOOTER_NOTE__` — one-line footer

### Required sections (in this exact order)

Use the numbered `<span class="num">XX</span>` style for every section heading. Numbers run consecutively starting at 01.

| # | Heading | Must contain |
|---|---|---|
| 01 | Problem & root cause | Exact error string if there is one; one-paragraph root cause |
| 02 | Scope & non-goals | Two-column `.grid.cols-2` cards |
| 03 | Current architecture | One Mermaid `flowchart` of how things work today |
| 04 | Target architecture | One Mermaid `flowchart` of the desired state |
| 05+ | Affected flows | One `sequenceDiagram` per flow that materially changes |
| — | Files to edit | A table with columns: path / action tag / why |
| — | Code changes (shape, not source) | Before/after Mermaid + a per-file pseudocode table |
| — | Third-party dashboard steps | Only if external services (Google Cloud, Stripe, Supabase, etc.) need configuration |
| — | Environment variables | Table; only the vars that change |
| — | Test plan | **End-to-end only by default** (Cypress / Playwright). Strategy Mermaid + pseudocode of cases. See "Tests: e2e by default" below. |
| — | Commands that will ask first | Table + paste-ready JSON snippet for `permissions.ask` |
| — | Rollout & verification | Phased; acceptance checklist as `<ul class="checklist">` |
| — | Risk & rollback | Two-column `.grid.cols-2` cards |
| Last | Agent runbook | Preflight commands, ordered execution Mermaid, commands cheat-sheet table, definition-of-done `<ul class="checklist">`, must-nots list |

See `references/sections.md` for concrete examples of each.

### Tests: e2e by default

At scale, the test that actually catches regressions is the one that drives the app like a user does. Mocked unit tests pass in isolation and lie about integration. The design doc reflects this priority:

- **Always include an end-to-end test plan.** Cypress or Playwright, whatever the repo already uses. Pseudocode the cases; the strategy diagram explains how they assert (intercept, network probe, DOM assertion).
- **Do NOT add a unit-test plan, integration-test plan, or "expand existing mocks" task unless the user explicitly asks for one.** If existing unit tests would be broken by the change, the doc says "update broken unit tests" — not "add new ones".
- **Definition of done references only the e2e suite.** Don't make `npm run test` (unit) a gate unless the user asked for it.
- **Skip lint/build gates by default too** unless the project's pre-existing CI runs them — those are scaffolding, not signal.

If the user asks for unit tests on top of e2e ("also add unit tests for the new helper", "extend the vitest suite"), add the unit plan as a separate subsection (e.g., 13.x) under the existing Test plan section. Don't replace the e2e content.

The runbook's Definition-of-Done checklist should only reference the e2e spec passing. Commands cheat-sheet should list the e2e runner, plus `npm run lint`/`build` only if the user opted into them.

### The pseudocode rule

Show the **shape** of the change, never paste full files. The doc is for orientation, not source-of-truth code. Use a sketch syntax that conveys intent:

```
- remove AppleIcon, MetaIcon, XIcon
- socialButtons = [{ google }]   (was 4 entries)
- handleOAuth(provider: "google")
+ <button data-testid="google-signin">
```

If you find yourself pasting more than ~10 lines of real source into a `<pre>`, stop and refactor it into pseudocode. Full source goes into the actual files during Phase 2, not the doc.

### Diagram conventions

- Mermaid 11. Always wrap in `<div class="diagram"><pre class="mermaid">…</pre></div>` and follow with `<p class="cap">caption</p>`
- Pick the right shape: `flowchart TB|TD|LR` for systems, `sequenceDiagram` for actor flows, `erDiagram` for data
- Mermaid is initialized once with `useMaxWidth: true` already — don't override
- Diagrams must render in both light and dark themes — the script in the template re-renders on toggle. Don't hard-code colors inside diagrams
- Use plain ASCII in diagram labels. Smart quotes and other Unicode quirks can break the Mermaid parser

### Pre-flight checks before saving

Before you tell the user the doc is ready:

1. **Tag balance.** Run a quick parser pass — every `<section>`, `<pre>`, `<table>`, `<div>`, `<aside>`, `<main>`, `<nav>`, `<script>`, `<style>` must match. Use the snippet in §"Tag balance check" below.
2. **Sidebar ↔ sections 1:1.** Count `<a data-href="…">` entries in the sidebar and `<section id="…">` in main. Same count. Same IDs.
3. **Every claim is verified.** Any file path, line number, API endpoint, env var name, error string, or version cited in the doc must be confirmed against the actual repo / live service before the doc lands — not pulled from memory. Use grep, read, curl. When something can't be verified, mark it as `(verify before relying)` rather than asserting it.
4. **No long source blocks.** If a `<pre>` is over 20 lines and isn't a permissions JSON snippet or a shell preflight command block, it's probably real source — re-cast as pseudocode.

#### Tag balance check

```python
import re
src = open(path).read()
for tag in ['section','aside','main','div','nav','pre','table','script','style']:
    o = len(re.findall(r'<'+tag+r'\b', src))
    c = len(re.findall(r'</'+tag+r'>', src))
    if o != c: print(f'WARN {tag} open={o} close={c}')
```

### Ask-gates section format

This section is doing real work — it's both the user's audit trail and the literal payload Phase 2 will merge into `~/.claude/settings.json → permissions.ask`. The session runs in auto mode: commands execute without confirmation by default. So the job is inverted from a classic allowlist — instead of enumerating everything the runbook will fire, **predict the small set of commands consequential enough that the user should still be asked**, and gate exactly those. Get this right and execution flows freely while the user is pulled in only at the moments that genuinely deserve a human pause; get it wrong and either something irreversible runs silently, or the user gets nagged for routine work.

#### The principle

**Gate consequence, not activity.** Walk the runbook (§17) and any embedded commands in §10/§13 and ask of each command: "if this ran while the user was looking away, could it cost something that can't be cheaply undone?" If yes, it gets an `ask` pattern. If no — builds, tests, greps, local file ops, read-only probes — it does NOT appear in this section at all. A short, sharp ask list is the goal; a long one means you're gating noise.

#### What MUST be gated — predict from the runbook

Categories that warrant an `ask` entry whenever the runbook touches them:

- **Irreversible / destructive:** `rm -rf` outside the worktree, `git push --force*`, `git reset --hard` on shared branches, dropping or truncating database tables, deleting cloud resources.
- **Outward-facing / publishing:** `git push` to a shared remote, production deploys (`vercel deploy --prod`, `vercel --prod`), `npm publish`, sending email/notifications, creating public URLs.
- **Mutating third-party state:** writes via cloud CLIs (`gcloud`, `supabase db push`, `stripe`, `gh pr merge`), MCP tools that create/update/delete remote objects (`mcp__<server>__<tool>` form), applying migrations against a live database.
- **Money / quota:** provisioning paid resources, plan changes, anything that bills.
- **Secrets:** writing env vars to a remote (`vercel env add`), rotating keys, commands that would echo credentials.

Be specific with patterns, same as ever: `Bash(git push:*)` over `Bash(git *)`; `Bash(vercel --prod:*)` over `Bash(vercel:*)`. The `*` is a glob — pick the smallest pattern that covers the risky command without catching its harmless siblings (gating `Bash(supabase db push:*)` must not also catch `supabase db diff`).

#### What must NOT be gated

Everything routine the runbook fires — test runners, linters, builds, `curl` read probes, `git status/diff/log/commit` inside the worktree, file-system tools (`Edit`, `Read`, `Write`, `Glob`, `Grep`), task tools. Listing these defeats the point of auto mode and buries the real gates in noise. If the section has more than ~8 entries, re-read it: you're probably gating activity instead of consequence.

#### Auditing comprehensively (do this before signing off)

Re-read §17 with a red-team mindset. For every backticked command, ask: "what's the worst plausible outcome if this runs unattended?" Anything whose answer involves prod, money, a shared remote, or data loss gets a pattern. Then do the inverse pass: for every entry in the list, name the runbook step that fires it ("Fires at") — an ask entry no step triggers is dead weight; cut it. Sort, dedupe, that's your `permissions.ask` array.

#### HTML format

```html
<section id="permissions">
  <h2><span class="num">14</span> Commands that will ask first</h2>
  <p>The session runs in auto mode — everything else in the runbook executes without prompts. These commands are consequential enough that Claude will pause and ask before running them. Paste-ready snippet for <code>~/.claude/settings.json → permissions.ask</code>:</p>
  <div class="scroll">
    <table>
      <thead><tr><th>Ask pattern</th><th>Risk if unattended</th><th>Fires at</th></tr></thead>
      <tbody>
        <tr>
          <td><code>Bash(git push:*)</code></td>
          <td>Publishes the branch to the shared remote</td>
          <td>§17.3 step 6</td>
        </tr>
        <tr>
          <td><code>Bash(vercel --prod:*)</code></td>
          <td>Production deploy — user-visible immediately</td>
          <td>§17.4</td>
        </tr>
        ...
      </tbody>
    </table>
  </div>
  <pre><code>[
  "Bash(git push:*)",
  "Bash(vercel --prod:*)",
  "Bash(supabase db push:*)"
]</code></pre>
</section>
```

The second column states the concrete risk — that's what the user is actually auditing. The third column ("Fires at") maps each gate to the runbook step that triggers it, proving nothing in the list is dead weight. The JSON inside `<pre><code>` is what Phase 2 parses; keep it as a valid JSON array with no comments.

### Agent runbook section format

This is the most important section. It's the part that lets a fresh agent execute the doc with no other context. Include all five subsections:

- **17.1 Preflight** — exact shell commands that verify state-of-the-world (read env, probe live services, confirm file paths exist, grep for hidden dependencies). Each command should print the decision-relevant output.
- **17.2 Execution order** — a Mermaid `flowchart TD` showing the phases, with explicit hand-off points where the user has to act in a third-party UI.
- **17.3 Commands cheat-sheet** — a table mapping intent → exact command (use the repo's real `npm run …` scripts, not invented ones).
- **17.4 Definition of done** — a `<ul class="checklist">` of objective checks (lint pass, test pass, grep returns empty, etc.). Each item must be verifiable, not opinion-based.
- **17.5 Must-nots** — five-ish concrete things the agent should not do even if they'd be expedient (e.g., "don't commit `.env.local`", "don't disable email confirmation to make tests easier"). These come from anticipating shortcuts that would compromise the change.

---

## Phase 2 — Approve & apply ask-gates

When the user approves the doc, apply the ask-gates in a single deterministic step. Don't ask the user to paste anything — the script below reads the JSON array directly out of the design doc you just wrote, so there's no copy-paste error path.

### The merge

`jq` cannot dedupe arrays without `--slurp` gymnastics, so use Python. This one command reads the doc, finds the permissions section, parses the JSON, and merges into `~/.claude/settings.json → permissions.ask` — preserving all other keys (hooks, env, theme, enabledPlugins, etc.). Substitute `DOC_PATH` for the actual design doc path you wrote in Phase 1:

```bash
python3 - <<'PY'
import json, pathlib, re, sys

DOC_PATH = "docs/<topic>-design-doc.html"   # <-- replace with the actual path
SETTINGS = pathlib.Path.home() / ".claude/settings.json"

doc = pathlib.Path(DOC_PATH).read_text()
# Find the permissions section and pull the first JSON array out of its <pre><code>...</code></pre>
m = re.search(
    r'<section id="permissions">[\s\S]*?<pre><code>(\[[\s\S]*?\])</code></pre>',
    doc,
)
if not m:
    sys.exit("ERROR: no <pre><code>[...]</code></pre> JSON array found inside <section id=\"permissions\">")
try:
    new = json.loads(m.group(1))
except json.JSONDecodeError as e:
    sys.exit(f"ERROR: permissions block is not valid JSON: {e}")
if not isinstance(new, list) or not all(isinstance(x, str) for x in new):
    sys.exit("ERROR: permissions block must be a flat array of strings")

cfg = json.loads(SETTINGS.read_text()) if SETTINGS.exists() else {}
perms = cfg.setdefault("permissions", {})
ask = perms.setdefault("ask", [])
before = set(ask)
added = [x for x in new if x not in before]
ask.extend(added)
SETTINGS.write_text(json.dumps(cfg, indent=2) + "\n")

print(f"Settings: {SETTINGS}")
print(f"Added to ask ({len(added)}):")
for x in added: print("  +", x)
skipped = [x for x in new if x in before]
if skipped:
    print(f"Already present ({len(skipped)}):")
    for x in skipped: print("  =", x)
PY
```

### Verifying the merge took effect

After the merge, any tool call matching an `ask` pattern WILL prompt the user — that's the point. Rules evaluate deny → ask → allow, first match wins, so an ask entry holds even if the same pattern also appears in `permissions.allow`. Sanity-check by:

1. Reading `~/.claude/settings.json` back and confirming every entry from the doc is present in `permissions.ask`.
2. Do NOT smoke-test a gated command — that fires a real prompt (or worse, the real side effect) just to test plumbing. The settings read-back is the verification.

One known hole: if Bash sandboxing is enabled with `autoAllowBashIfSandboxed` (the default when sandboxing is on), sandboxed Bash commands skip ask prompts — the sandbox boundary substitutes for them. If the project uses sandboxing, say so to the user at sign-off: the gates then rely on the agent honoring them behaviorally (see Safety rules), not on the harness prompting.

### Safety rules

- Never edit `permissions.ask` outside of this script — manual edits drift from what the doc says, and the recap in Phase 4 won't be honest.
- Never apply the gates before the user approves the doc. The doc is the audit artifact; running this step is the consent signal.
- Never write to `permissions.allow` or `permissions.deny` from this skill — those are user-managed.
- Do not gate `Edit`, `Read`, `Write`, `Glob`, `Grep`, or bare `Bash` — gating generic file-system tools or all of Bash defeats auto mode entirely.
- When a gated command comes up during Phase 3, present it and wait. Never reword or restructure a command so it slips past its own ask pattern.

---

## Phase 3 — Worktree + execute

1. **EnterWorktree.** Unless the doc itself opts out (some changes legitimately belong on the main checkout — surface this to the user before deciding). The worktree isolates uncommitted state so a failed execution can be discarded by abandoning the worktree.
2. **Mirror the runbook into a task list.** Use TaskCreate so progress is visible in the UI; the runbook's "Execution order" diagram is the source. Mark each task in_progress when starting, completed when done.
3. **Execute step by step.** Run each preflight command; act on the output. Don't batch decisions ahead — the doc anticipates branches (e.g., "if external.google is true, skip §11") and you should follow them.
4. **Keep a deviation log.** Maintain a running scratch list as you go. Every time something diverges from the doc, append an entry. Examples of what counts:
   - A file you had to touch that wasn't in the §9 table
   - A command in the doc that didn't work as written and what you ran instead
   - A third-party dashboard step where the UI flow differed from the doc
   - An assumption in the doc that turned out to be wrong (file moved, env var renamed, version drift)
   - A blocker that pushed work to a follow-up

Keep the log in a scratch file at `$CLAUDE_JOB_DIR/deviations.md` (or in memory if no jobs dir) so it survives a compact. Format each entry as:

```
## <what>
- planned: <what the doc said>
- actual: <what happened>
- why: <root cause / surprise>
- impact: <kept going / fixed in flight / deferred to follow-up>
```

### When to stop and hand back

Stop and ask the user before continuing if:

- The deviation is bigger than the doc's scope (would require revisiting Phase 1)
- A third-party dashboard step requires their account / consent
- A command matches one of the doc's ask-gates — the prompt is the hand-back; never reword a command to dodge its gate
- You're about to run something consequential (prod, money, shared remote, data loss) that the doc failed to gate — treat it as gated anyway and ask; the missing pattern goes in the deviation log
- A "must-not" from §17.5 would be violated by the path forward

---

## Phase 4 — Append "Roadblocks & deviations"

When execution ends (success, partial, or stopped), append a new section to the same HTML doc. It's the source of truth for what *actually* happened.

### Section content

- **Numbered** as the next in sequence (e.g., if the runbook was §17, this is §18)
- **Title:** `Roadblocks & deviations`
- One opening paragraph: honest one-sentence summary of how it went (success / partial / blocked) and the headline reason
- A small Mermaid `flowchart LR` of **planned vs actual** if the path diverged meaningfully — left side is the doc's flow, right side is what actually happened, with arrows linking the diverge points
- A `<table>` with three columns: **Planned** / **Actual** / **Why** — one row per deviation log entry
- A `<ul class="clean">` of surprises worth recording for next time (things that didn't break the change but would have been useful in the doc up front)
- A short "doc patches" subsection listing fixes the doc itself needs — these are the corrections the next person reading the doc will benefit from

### Sidebar + TOC updates

Add the matching entry to **both** the inline `<nav class="toc">` and the sticky `<aside class="outline">`:

- TOC: `<li><span>18</span><a href="#roadblocks">Roadblocks & deviations</a></li>`
- Outline: `<a href="#roadblocks" data-href="roadblocks"><span class="n">18</span><span>Roadblocks &amp; deviations</span></a>`

### Re-balance before saving

Run the tag balance check again. Append-only edits to an HTML file have a way of breaking sidebar ↔ section parity. Confirm:

- Section count == sidebar entry count == TOC entry count
- The new section ID is unique
- No `<hr class="divide" />` was dropped between sections
- Mermaid block count increased by exactly 0 or 1

---

## Naming and tone

- The doc speaks plainly. No marketing voice. State what something is, why it matters, and what changes.
- Headings use sentence case (`Files to edit`, not `Files To Edit`).
- Tables prefer short cells. If a cell is a paragraph, lift it into prose under the table.
- Code/identifiers always in `<code>`. File paths too.
- Use Mermaid for any flow with more than two arrows. Use ASCII or prose for two-step things — diagrams have overhead.
- One Mermaid block per concept. Don't cram four ideas into one diagram.

## What this skill does NOT do

- Does not write the design doc autonomously without first scanning the codebase. Every claim is grounded.
- Does not apply ask-gates to `settings.json` before the user approves the doc.
- Does not skip the worktree.
- Does not silently expand scope mid-execution.
- Does not consider the work done until the recap section is in place.

## Pointer: section examples

See `references/sections.md` for one realistic example of each required section, copy-pasteable as a starting point.

---
> Source: [nihadern/design-and-ship](https://github.com/nihadern/design-and-ship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
