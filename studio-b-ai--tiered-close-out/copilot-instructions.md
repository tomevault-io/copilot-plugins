## tiered-close-out

> Three-tier session close-out for Claude Code — routes behavioral rules, operational knowledge, and unfinished work to three stores with three independent approval gates, plus pluggable blocking domain guards. Generic scaffold — fill in your own stores and guards. Trigger on phrases like "close out", "wrap up", "session sweep", or end-of-session signals.


# Tiered Close-Out

Parse the current conversation, route outputs to the right tier, and present a summary for approval before committing.

## When to Trigger

User says "close out", "wrap up", "session sweep", "tier close", or signals end of session.

## Approval Gates

This skill uses **per-tier approval** rather than a single trailing yes/no. After the summary is presented, ask the user one `AskUserQuestion` per independent decision so each tier can be accepted, narrowed, or rejected without blocking the others:

1. **Q1 — Rules-store updates** (Tier 1).
2. **Q2 — Knowledge-store ingestion** (Tier 2).
3. **Q3 — Domain guard results** (only when at least one registered guard returned `FAIL`). PASS prints a check silently; FAIL asks whether to block close-out or override with a documented reason.

Branch disposition (§5) is delegated to `superpowers:finishing-a-development-branch` and owns its own gating — do not wrap it in `AskUserQuestion`. Each tier is approved independently; never collapse them into a batch yes/no.

## Process

Scan the entire conversation and identify:

### 0. Cross-Session Context (run first)

Before analyzing the current session, look for related past sessions so the close-out can flag recurring patterns ("3rd time this thing has broken") and dedupe against existing knowledge.

```
# TODO(0): cross-session context lookup.
#
# Identify the primary topic/project of this session from the conversation,
# then query your session/knowledge backend for the top ~5 related past
# sessions. Use the results to:
#   - flag recurring patterns
#   - reference prior decisions/fixes in your knowledge entries
#   - avoid duplicate ingestion in §2
#
# Common backends:
#   - claude-mem (vector recall over prior sessions)
#   - qmd-sessions (dated markdown files, search via ripgrep)
#   - SQLite-FTS over a notes folder
#   - ripgrep over ~/notes/ or similar
#
# Implement search_prior_sessions(query: str) -> list[{date, title, snippet}].
# If the backend is unavailable, skip this step and proceed.
```

### 1. Rules — Tier 1

New behavioral instructions the user stated during the session ("always do X", "never do Y", "I want Z from now on").

```
# TODO(1): rules-store path.
#
# Default placeholder: ~/.claude/CLAUDE.md
# Replace if your project uses AGENTS.md, .cursor/rules, or a dotfile under
# the repo. The store should be a plain-text file the harness auto-loads
# every session.
RULES_STORE = "~/.claude/CLAUDE.md"
```

**Action:** Propose the edit, show the diff, gate via Q1.

### 2. Operational Knowledge — Tier 2

New facts learned during the session:

- Incidents (what broke, why, how it was fixed)
- Architectural decisions (what was decided and why)
- Patterns and anti-patterns discovered
- Business rules stated
- Infrastructure changes

```
# TODO(2): knowledge-store search + ingest.
#
# Provide two functions:
#
#   def search_existing(query: str) -> list[dict]:
#       """Return top matches for dedup. Each dict: {id, title, snippet, score}."""
#
#   def ingest_entry(payload: dict) -> None:
#       """Persist one entry. Payload: {title, body, tags: dict, source}."""
#
# Common backends:
#   - tagged markdown folder (~/notes/{domain}/{slug}.md with frontmatter,
#     search via `rg --type md`)
#   - SQLite-FTS database
#   - vector DB (Qdrant / Chroma / pgvector) with an embedding model
#   - claude-mem ingest CLI
#
# Suggested tag schema (adapt freely):
#   - domain   (which area of work)
#   - project  (which repo/product)
#   - source   (incident | architecture | policy | runbook)
```

**Action:** Build the documents, search for duplicates, show them for approval, gate via Q2.

### 3. Unfinished Work

Tasks that were started but not completed, follow-ups needed in other repos, pending decisions.

**Route to:** Summary in the close-out message. Flag cross-repo dependencies prominently. No persistent store.

### 4. Session Artifacts

Commits, PRs, design docs, scripts, branches, worktrees created during the session.

**Route to:** Summary list with full paths and links.

### 5. Branch Disposition

If the session is in a git worktree or a feature branch with commits:

**Invoke `superpowers:finishing-a-development-branch`** to present structured options:

- Merge to main (if clean, reviewed, passing)
- Create PR (if needs review or CI)
- Keep worktree (if work is ongoing)
- Remove worktree (if abandoned or merged)

This replaces ad-hoc worktree cleanup — the skill handles the full decision tree. Do **not** wrap this in `AskUserQuestion`; the downstream skill owns its own approval surface.

### 6. Cleanup

- CLI/project links that were changed during the session
- Temporary files or state

### 7. Domain Guards — pluggable blocking checks

If the session touched code/files governed by a registered guard, run the guard **before** the summary. Failures drive Q3 below.

This is the slot for "irreversible-cost" gates — checks where missing a step now means a production outage, data loss, or a fix that cannot be applied retroactively. Generic close-outs ship with one example guard and an empty `GUARDS` list; you extend it for your stack.

```python
def example_guard() -> dict:
    """
    Example guard. Replace with a real check.

    Candidate guards (pick what matters for your stack):
      - migration_rollback_present:  every new SQL migration ships with a
        matching .down.sql / rollback step.
      - public_api_changelog:        every diff touching public API surfaces
        adds an entry to CHANGELOG.md or equivalent.
      - billing_change_feature_flag: every diff touching pricing/billing
        code is gated behind an off-by-default feature flag.
      - secret_rotation_bump:        every change to an env var name bumps
        the rotation log + paper-trail file.
      - customer_facing_copy_review: every diff to user-visible copy is
        reviewed/approved before merge.

    Each guard returns:
      {
        "status": "PASS" | "FAIL",
        "message": str,             # human-readable detail; required on FAIL
        "override_allowed": bool,   # if False, FAIL hard-blocks (no override)
      }
    """
    return {
        "status": "PASS",
        "message": "no checks registered",
        "override_allowed": True,
    }


# Register additional guards by appending to GUARDS. Each guard is a
# callable with no args returning the dict shape above.
#
# Reference instance: Kevin Bibelhausen's `rigby` skill — the original
# this scaffold was extracted from — has a 16-line guard validating that
# every Acumatica `<GenericInquiryScreen ExposeViaOData=1>` block in a
# changed project.xml has a matching entry in GI_SCREENS_REQUIRING_GRANT[]
# in the same PR. Missing that pairing means a 24-hour production outage
# that SOAP cannot recover from. That's the kind of irreversible-cost
# gate this hook is for — not lint, not style, not "would be nice".
GUARDS: list = [example_guard]
```

**Output format in the close-out summary:**

```
DOMAIN GUARDS:
  PASS  example_guard — no checks registered
  ── or ──
  FAIL  migration_rollback_present — migration 042 ships without down.sql
        Q3 will gate close-out.
```

## Output Format

Present everything in one structured summary, then drive Approval Flow.

```
TIERED CLOSE-OUT — {date}

PRIOR SESSIONS:
  {related past sessions found, or "No backend configured"}

RULES UPDATES (Tier 1):
  {proposed additions/changes, or "None"}

KNOWLEDGE INGESTION (Tier 2):
  {list of new entries with tags, or "None"}

UNFINISHED:
  {list of incomplete work with clear next steps}

ARTIFACTS:
  {commits, files created/modified, PRs, branches}

BRANCH:
  {merge/PR/keep/remove recommendation, or "Not in a feature branch"}

CLEANUP:
  {linked projects, temp state}

DOMAIN GUARDS:
  {PASS / FAIL block per registered guard, or "None registered"}
```

## Approval Flow

After printing the summary, ask each tier independently via `AskUserQuestion`. Skip a question entirely if its tier had nothing to propose — don't ask `AskUserQuestion` with no real choice.

### Q1 — Rules updates

Skip if `RULES UPDATES` is "None".

Otherwise call `AskUserQuestion`:

- **Question:** "Apply the proposed rule updates?"
- **Options:**
  - `Apply all` — write all proposed edits to the rules store.
  - `Apply selected` — user picks which by number/label; show the list, accept the subset, write only those. **Omit this option when only one rule edit is proposed** — it collapses to `Apply all`.
  - `Skip` — make no changes.

### Q2 — Knowledge ingestion

Skip if `KNOWLEDGE INGESTION` is "None".

Otherwise call `AskUserQuestion`:

- **Question:** "Ingest the proposed knowledge entries?"
- **Options:**
  - `Ingest all` — persist every entry.
  - `Ingest selected` — user picks which; ingest only the chosen subset. **Omit this option when only one entry is proposed** — it collapses to `Ingest all`.
  - `Skip` — no ingestion.

### Q3 — Domain guard failures

Skip if every registered guard returned `PASS`, or if no guards are registered.

For each `FAIL` guard with `override_allowed=False`: print the failure detail and stop. No question — that gate is hard-block by design.

For one or more `FAIL` guards with `override_allowed=True`, call `AskUserQuestion`:

- **Question:** "Domain guard(s) failed. How do you want to proceed?"
- **Options:**
  - `Block close-out (correct first)` — DEFAULT. Stop here. Print the failure list and the file(s) that need editing. Do not apply any other tier's actions until rerun.
  - `Override (document why)` — proceed, but require a non-empty reason string from the user. Capture the reason in the close-out summary under a `GUARD OVERRIDE:` block alongside operator + date as a paper trail. Empty/whitespace reason = treat as `Block`.

### Execution after answers

Apply the approved actions in order:

1. Tier 1 edits (if Q1 approved).
2. Tier 2 ingestion (if Q2 approved).
3. Hand off branch disposition to `superpowers:finishing-a-development-branch`.
4. Report cleanup actions taken.

If Q3 was `Block`, stop after printing the failure detail — skip steps 1–4.

## Rules

- **Never skip the summary.** Even if the session was trivial, present the summary.
- **Each tier is approved independently — no batch approval.** Q1, Q2, Q3 are separate `AskUserQuestion` calls; the user can accept one and reject another.
- **Deduplicate against existing knowledge.** Search before ingesting to avoid duplicates.
- **Don't ingest conversation-specific context** (debugging steps, intermediate attempts). Only ingest the conclusions — what was learned, decided, or built.
- **Incident entries must include:** what broke, root cause, fix applied, prevention rule.
- **Guard override requires a documented reason.** Empty reason falls back to Block.

---
> Source: [studio-b-ai/tiered-close-out](https://github.com/studio-b-ai/tiered-close-out) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
