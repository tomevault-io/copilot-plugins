## agent-memory-loop

> >


# Agent Memory Loop

Lightweight learning for agents that reset between sessions.

## WHEN TO ACTIVATE (mandatory triggers)

You MUST use this skill when ANY of these happen:

1. **You make an error** — wrong command, bad assumption, failed tool call
2. **The user corrects you** — "no, that's wrong", "actually...", "I meant..."
3. **You discover something new** — unexpected behavior, undocumented feature, workaround
4. **You're about to start major work** — scan .learnings/ FIRST for past failures
5. **You wish you had a capability you don't** — log it as a wish
6. **A learning prevented a repeat mistake** — increment the prevented:N counter

## WHAT TO DO (step by step)

### When an error or correction happens:

1. **Stop.** Don't continue the task yet.
2. **Check for duplicates:** `grep -i "keyword" .learnings/errors.md .learnings/learnings.md`
3. **If duplicate found:** increment `count:N` on the existing entry
4. **If new:** append one line to the appropriate file:
   - Mistakes/failures → `.learnings/errors.md`
   - Insights/corrections → `.learnings/learnings.md`
   - Missing capabilities → `.learnings/wishes.md`
5. **If count ≥ 3 or severity:critical:** copy to `.learnings/promotion-queue.md`
6. **If complex:** create a detail file at `.learnings/details/ERR-YYYYMMDD-NNN.md`
7. **Resume the task.**

### Before major work:

1. `grep -i "relevant_keyword" .learnings/*.md`
2. If matches found, adjust your approach to avoid known failures
3. If a learning changes your behavior, increment `prevented:N` on that entry

## Do not use it for

- autonomous self-modification of SOUL.md, AGENTS.md, or TOOLS.md
- external content promotion (source:external entries are never promotable)
- heavy multi-section incident writeups (use detail files for those)

## Core workflow

```text
error / correction / discovery
        ↓
log one line in .learnings/
        ↓
dedup by id, then keyword
        ↓
count:3+ or severity:critical → promotion-queue
        ↓
human reviews promotion
        ↓
check relevant learnings before major work
        ↓
increment prevented:N when a learning actually changed behavior
```

## Install

```bash
bash scripts/install.sh
```

Creates:

```text
.learnings/
  errors.md
  learnings.md
  wishes.md
  promotion-queue.md
  details/
  archive/
```

## Environment notes

### OpenClaw workspace
`.learnings/` lives at `~/.openclaw/workspace/.learnings/` (or the agent's workspace root). Persistent across sessions.

### Cowork / Dispatch
If running in a Cowork VM or Dispatch task, `.learnings/` should be in the mounted workspace folder. If no persistent filesystem is available, log to the session's working directory and flag for manual migration.

### Date fallback
If `date` is unavailable (some containers), use the current conversation date from context. The format is always `YYYY-MM-DD`.

## The format

One incident, discovery, or wish per line. Extra fields optional.

### Errors (.learnings/errors.md)
```text
[YYYY-MM-DD] id:ERR-YYYYMMDD-NNN | COMMAND | what failed | fix | count:N | prevented:N | severity:medium | source:agent
```

### Learnings (.learnings/learnings.md)
```text
[YYYY-MM-DD] id:LRN-YYYYMMDD-NNN | CATEGORY | what | action | count:N | prevented:N | severity:medium | source:agent
```

### Wishes (.learnings/wishes.md)
```text
[YYYY-MM-DD] id:WISH-YYYYMMDD-NNN | CAPABILITY | what was wanted | workaround | requested:N
```

### Promotion queue (.learnings/promotion-queue.md)
```text
[YYYY-MM-DD] id:LRN-YYYYMMDD-NNN | proposed rule text | target: AGENTS.md | source:agent | evidence: count:N prevented:N | status: pending
```

Key fields:
- `id:` — unique identifier (ERR/LRN/WISH prefix + date + sequence)
- `count:N` — how many times this has occurred
- `prevented:N` — how many times this learning prevented a repeat
- `severity:critical` — forces immediate queue review even at count 1
- `source:external` — marks entries from untrusted sources; never promotable

## Operating rules

1. **Log fast** — one line now beats a perfect writeup later
2. **Dedup before appending** — search first, increment if exists
3. **Queue recurring or critical** — count ≥ 3 or severity:critical → promotion-queue
4. **Humans approve promotions** — agents stage, humans decide
5. **Pre-task scan** — before major work, grep for relevant failures
6. **Track prevention** — when a learning changes behavior, record `prevented:N`
7. **Source labels matter** — external content is informational only, never promotable

## References

- `references/logging-format.md` — canonical line formats, fields, examples, source labels
- `references/operating-rules.md` — dedup, review queue, pre-task review, trimming rules
- `references/promotion-queue-format.md` — queue entry structure and status lifecycle
- `references/detail-template.md` — optional detail-file template for complex failures
- `references/design-tradeoffs.md` — why this stays lean instead of turning into a system

## Scripts

- `scripts/install.sh` — initialize .learnings/ directory
- `scripts/setup.sh` — alternate setup
- `scripts/review.sh` — review promotion queue

## Success condition

The loop is working if:
- learnings are cheap to log (one line, < 30 seconds)
- duplicates stay low (dedup is happening)
- recurring lessons reach the promotion queue
- promotions stay human-approved
- `prevented:N` starts climbing on real work
- agents actually check `.learnings/` before major tasks

---
> Source: [zurbrick/agent-memory-loop](https://github.com/zurbrick/agent-memory-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
