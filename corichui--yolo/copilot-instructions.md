## yolo

> <!-- YOLO:routing-block start -->


<!-- YOLO:routing-block start -->
## YOLO workflow (this is a YOLO-initialized repo)

This project uses YOLO (reasoning-first). Feature state lives in **git, not in tracked
status files** — see `.claude/yolo/conventions.md`. This block is the primary human-readable
routing guide; the machine auto-triggers live in each skill's `description`. The confidence
tiers and does-NOT-route taxonomy below are unique to this block.

> **File resolution:** YOLO may be installed as a plugin. If `${CLAUDE_PLUGIN_ROOT}` is set,
> resolve framework paths (`.claude/yolo/...`, `.claude/skills/...`) against it; otherwise read
> them project-relative. Project state (`workspace/`, this `CLAUDE.md`) is always project-local.
> See *Where YOLO's files live* in `conventions.md`.

### Routing feature intent (auto-detect, then confirm at the gates)

When a prompt expresses intent to **build / add / implement / create** a feature, route it to
the `yolo-feature` skill on your own — don't wait to be told "use YOLO." For feature intent
this takes **precedence over generic brainstorming**. Auto-routing is safe because the cheap
phase (intent capture + brief) is reversible and the **plan gate stops before any expensive or
irreversible work**, so a wrong guess costs at most a throwaway brief.

**Routes to `yolo-feature` (feature intent):**
"add CSV export", "build a settings page", "implement OAuth login", "create an endpoint for
X", "support dark mode", "let users reset their password".

**Does NOT route — handle normally (the common false positives):**
- **Questions / understanding** — "how does X work?", "where is Y?", "why does Z happen?".
  Answer directly; reach for `yolo-research` only if deeper code exploration helps.
- **Trivial edits** — a typo, a one-line or single-file fix, a rename, a config tweak. Below
  the ceremony threshold: just make the edit, no brief.
- **Refactors / cleanups** with no new user-facing capability — do them directly unless the
  user explicitly wants it tracked as a feature.
- **Bug reports** — "X is broken", "fix the 500 on login" → route to **`yolo-debug`** for a
  systematic reproduce → isolate → fix → verify pass (a genuinely trivial one-line fix can
  still just be made). If the fix turns out to need new capability, `yolo-debug` hands off to
  `yolo-feature`.
- **Other YOLO acts** — decisions, status, intake, roadmap (see *Other intents*).

**Confidence tiers:**
- **High** (clear feature intent) → route: capture intent, draft the brief, run research (if
  the change is non-trivial) + plan, then **STOP at the plan gate**.
- **Ambiguous** (could be a feature, or just a question/fix) → ask one line first:
  *"Treat this as a feature (YOLO — brief → plan gate), or handle it directly (no YOLO)?"*
- **Low** → don't route; answer normally.

**Escape hatch (per prompt):** "skip YOLO" / "skip the ceremony" → handle directly: no brief,
no gates — this means you are NOT running a YOLO feature at all, so there are no gates to skip.
It is distinct from the two-gate consent overrides below (which keep the YOLO flow but
pre-approve a gate); if it IS a feature you still want landed, use one of those instead.

### The two gates
Intent capture and brief drafting run freely; the user's "build X" request authorizes research
+ plan; then **STOP at the plan gate** (before writing code) and **STOP at the ship gate**
(before merging to base). "Just go" pre-consents the plan gate; "just ship it" pre-consents the
ship gate; "don't ask me anything" pre-consents both; "walk me through each step" adds
confirmations — all per-feature prose overrides, see `.claude/yolo/conventions.md` *The two gates*.

### Other intents
"where do things stand / where was I" → `yolo-status`; "break this epic into features / plan a
milestone" → `yolo-roadmap`; "use the project docs as context for a feature" → `yolo-intake`; "help me decide X
vs Y" → `yolo-decide`; "debug this / why is X failing / fix the 500" → `yolo-debug`; "set up
YOLO here" → `yolo-init`.

**Individual steps** can also be invoked alone: "explore the code first" → `yolo-research`;
"just make a plan" → `yolo-plan`; "verify this" → `yolo-verify`; "ship it / land this" →
`yolo-finish`.
<!-- YOLO:routing-block end -->

---
> Source: [CoriChui/yolo](https://github.com/CoriChui/yolo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
