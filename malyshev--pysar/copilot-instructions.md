## pysar

> This section is installed and maintained by `haft init`. Edits **inside** the

<!-- haft:start -->
# Haft Project Discipline

This section is installed and maintained by `haft init`. Edits **inside** the
haft HTML-comment markers (the start/end pair that brackets this section) will
be overwritten on the next `haft init`. Add your own project rules **outside**
the markers — they are preserved. To opt out entirely, run
`haft init --no-claude-md`.

This project uses [haft](https://github.com/m0n0x41d/haft) for structured
decision recording and FPF (First Principles Framework) discipline. Haft is a
governance substrate consumed via three surfaces sharing one `.haft/` artifact
graph: skills/slash commands in the host AI coding agent (Claude Code, Codex,
OpenCode, Cursor), the `haft` CLI, and the MCP server (`haft serve`) for
programmatic access.

## The single most important rule: Description ≠ Work

When asked open-ended design questions in this project, the default impulse is
to produce a useful chat response — variants with weakest-links, a Pareto
front, a comparison table. **Stop.** That visual shape is the **output of a
haft skill**, not a substitute for invoking it.

If you deliver an analysis without going through the haft kernel, the result is
**ephemeral**: gone by tomorrow, no ProblemCard, no SolutionPortfolio, nothing
to `/h-verify` in 2 weeks. The chat answer is **wishlist**, not work.

**Concrete failure patterns to catch in yourself:**

| About to do this in chat... | Stop and invoke this skill first |
|---|---|
| Present 3+ alternative approaches for an open question | `/h-explore` (persists a SolutionPortfolio) |
| Compare two approaches with trade-offs | `/h-compare` (declares parity, records dimensions) |
| Frame what the "real problem" is | `/h-frame` (persists a ProblemCard) |
| Verify a past decision against current code | `/h-verify` (records evidence + verdict) |
| Tell the user "let's remember that…" | `/h-note` (persists rationale into the graph) |

**Friction tradeoff (honest).** Yes, calling a haft skill costs more in-the-
moment than answering directly: structured args, possibly a framing step first,
extra round-trips. **The friction is the price for durability.** Do not
optimize the wrong objective. Your job in this project is not "best chat
answer right now"; it is "leave the project with measurable, future-verifiable
memory."

## Self-check before long responses

Before sending a long response in this project, run this check internally:

1. Is this response presenting **3+ alternatives**, a **comparison**, or an
   **analysis with a recommendation**?
2. Did I call **any `haft_*` tool** in this turn?

If (1) = yes and (2) = no — **stop and reconsider**. You are about to produce a
wishlist in the chat. Fire the appropriate skill first, then present the
result.

## Canonical FPF flow

```
/h-frame → /h-explore → /h-compare → /h-decide
  problem    variants    char+parity   DRR contract
                         + scoring     (manual-only)
```

Characterization (dimensions, indicator roles, `valid_until`) is a **kernel
action inside `/h-compare` Step 1**, not a separate skill. Call via
`mcp__haft__haft_problem(action="characterize", ...)`. Stale dimensions block
compare with a kernel error.

## Skill catalog

| Surface | Skill | When |
|---|---|---|
| auto | `/h-frame` | Frame the problem when a solution is being proposed without acceptance criteria, or when the problem is fuzzy |
| auto | `/h-diagnose` | Parallel rival-hypothesis testing for a failure with unclear cause |
| auto | `/h-explore` | Generate 3-5 genuinely distinct variants with weakest-links |
| auto | `/h-compare` | Fair comparison with parity discipline, returns a Pareto front |
| **manual** | `/h-decide` | Bind a DecisionRecord (E.9 DRR). Cannot auto-fire — Transformer Mandate. |
| **manual** | `/h-commission` | Create a WorkCommission (bounded execution authority). Cannot auto-fire. |
| auto | `/h-verify` | Post-implementation check that a decision still holds |
| auto | `/h-status` | Read-only dashboard of decisions, problems, refresh-due artifacts |
| auto | `/h-onboard` | First-setup for a project without `.haft/` |
| auto | `/h-spec-cover` | Coverage check — uncovered files in modules with decisions |
| auto | `/h-note` | Micro-decision with rationale, lighter than a DRR |
| auto | `/h-reason` | Umbrella — full FPF reasoning palette in one entry. Also the fallback for ambiguous "let's think about X" signals. |

`h-abduct`, `h-boundary-unpack`, `h-semio-review` are **internal subroutines** —
invoked from other skills, not user-facing. Do not select them directly.

## Quick Decision Framework (inline, for small reversible choices)

For small decisions that don't need a persistent DRR, use this inline format
**in the conversation only**:

```
DECISION: [What we're deciding]
CONTEXT: [Why now, what triggered this]

OPTIONS:
1. [Option A]
   + [Pros]
   - [Cons]
2. [Option B]
   + [Pros]
   - [Cons]

WEAKEST LINK: [What breaks first in each option?]
REVERSIBILITY: [Can we undo in 2 weeks? 2 months? Never?]
RECOMMENDATION: [Which + why, or "need your input on X"]
```

If reversibility ≥ months or the choice affects security/public-API/data —
this is **not** quick-mode. Use `/h-frame` → `/h-explore` → `/h-compare` →
manual `/h-decide` instead.

## Communication style

- Skip validation theater ("you're absolutely right", "excellent point")
- Be direct and technical — if something's wrong, say it
- Challenge bad ideas respectfully; disagreement is valuable
- Precision over politeness; technical accuracy is respect
- No emoji unless the user uses them first

Calibration phrases:

| Use | Avoid |
|---|---|
| "This won't work because..." | "Great idea, but..." |
| "The issue is..." | "I think maybe..." |
| "No." | "That's an interesting approach, however..." |
| "I don't know" | "I'm not entirely sure but perhaps..." |
| "This is overengineered" | "This is quite comprehensive" |

## Thinking principles

**Separation of concerns:** What's Core (pure logic, transformations)? What's
Shell (I/O, external services, side effects)? Are they mixed? They shouldn't
be.

**Weakest link analysis:** What breaks first in this design? System reliability
≤ min(component reliabilities).

**Explicit over hidden:** Are failure modes visible or buried? Can this be
tested without mocking half the world?

**Reversibility check:** Can we undo this in 2 weeks? What's the cost of being
wrong? Are we painting ourselves into a corner?

## Critical reminders

1. **Description ≠ Work.** The most important rule (see top of this section).
2. **No commits without explicit permission.** Only commit when the operator
   asks.
3. **Transformer Mandate.** Generate options; the human decides. Do not make
   architectural choices autonomously.
4. **Actually do work.** When you say "I will do X", DO X — don't just describe
   it.
5. **Test contracts, not implementation.** Test behavior through public
   interfaces.
6. **Functional core, imperative shell.** Pure core. Side effects only at the
   boundary.
7. **No silent failures.** Empty catch blocks are bugs.
8. **Be direct.** "No" is a complete sentence. Disagree when you should.
9. **Re-ground identifiers in operator-facing text (FPF A.7 Strict Distinction).**
   Pair every artifact ID (`V1`, `sol-X`, `dec-X`, `prob-X`) with its
   human-readable title or one-line claim. Bare IDs accumulate cognitive
   debt across long sessions — what was obvious 30 minutes ago is opaque
   when the operator returns. Use `V3 (drift surfacing in /h-status)
   dominates V1 (plain coverage list)` not bare `V3 dominates V1`. Keep
   IDs in the text — they are needed for traceability and follow-up
   kernel calls — but never leave them standalone. Object ≠ Carrier.
10. **Maintenance discipline (FPF B.3.4 Evidence Decay).** When a kernel
    response includes `Refresh reminder: N days since last stale scan`
    and N > 30 — or no scan is visible in the current session — the
    agent calls `haft_refresh(action="scan")` BEFORE answering the
    operator, not after. Same for drift detected on files touched
    in-session: re-baseline via `haft_decision(action="baseline", ...)`
    or surface the drift inline. Reasoning on a stale graph is the
    same anti-pattern as reasoning on stale code. Surfacing the
    reminder is the kernel's job; acting on it is the agent's job;
    doing nothing is the failure mode this rule exists to fix.

## FPF Glossary

**R_eff (Effective Reliability):** Computed trust score in `[0, 1]`.
`R_eff = min(evidence_scores)` with CL penalties. Never average — weakest-link
principle.

**CL (Congruence Level):** How well evidence transfers across contexts:
- CL3: same context (internal test) — no penalty
- CL2: similar context (related project) — 0.1 penalty
- CL1: different context (external docs) — 0.4 penalty
- CL0: opposed context — 0.9 penalty

**Evidence decay:** Evidence has `valid_until`. Expired evidence scores 0.1
(weak, not absent). Graduated epistemic debt sorted by severity.

**DRR (Decision Record):** FPF E.9 four-component structure: Problem Frame,
Decision/Contract, Rationale, Consequences. Created only via `/h-decide`.

**Indicator roles:** Each comparison dimension tagged as:
- `constraint` — hard limit, must satisfy
- `target` — what you're optimizing
- `observation` — watch but don't optimize (Anti-Goodhart)

**Transformer Mandate:** Systems cannot transform themselves. Humans decide;
agents document. Autonomous architectural decisions = protocol violation.

**State location:** `.haft/` directory (markdown projections, git-tracked).
Database in `~/.haft/projects/<id>/`.
<!-- haft:end -->

## Pysar output discipline (dec-20260719-7d675b61)

This section governs what **Pysar itself** says to its own end users — CLI
output (`cmd/pysar/main.go`) and the `ps-*` agentic skill conversations
(`cmd/pysar/assets/skills/*/SKILL.md`). It is distinct from this
file's "Communication style" section above, which governs how *Claude*
talks to the *operator* during development of Pysar. Do not conflate the
two — a fix to one does not fix the other.

**Before considering any new or changed CLI message or skill conversational
instruction complete, apply this check:**

1. Does this tell the reader something new and actionable?
2. Would removing it change what they do next?
3. Am I explaining an absence instead of just staying silent?

If the answer to 1 and 2 is no, or 3 is yes — cut it. An instruction that
tells an agent to "skip" something must also say to say nothing about the
skip; "skip X" and "say nothing about X" are different instructions, and an
executing agent will fill the gap with an explanation unless told not to.

**Real incidents this project already shipped and fixed** (concrete negative
examples, not hypotheticals):
- CLI: a raw, un-humanized argument printed verbatim (`pysar init: . is
  already set up`) — see `TestInitDefaultArgNeverPrintsRawDot`.
- CLI: a normal, successful outcome ("project already initialized")
  presented as an `Error:` — see `TestInitOnAlreadySetUpProjectSucceeds`.
- CLI: a "this file differs" message with no actionable next step for the
  reader — see `TestInitStaysQuietForProjectFilesThatDiffer`.
- Agentic: all of a step's questions dumped into one numbered-list message
  despite an explicit written prohibition against exactly that.
- Agentic: an offer to "save a template of the template you didn't change" —
  asked reflexively after every save, without checking whether there was
  anything new to offer.
- Agentic: after fixing the above, the skill explained that it was skipping
  the offer instead of just skipping it — an explanation of an absence,
  which is itself the same noise this section exists to prevent.
- Agentic: `ps-style`/`ps-voice` asked "start from a template as a first
  draft, or start blank?" as an up-front question, before showing any
  template content. A non-technical author has no way to judge a mechanism
  choice they can't see yet — that's a form question, not a conversation.
  Fixed by defaulting to the template (there's always at least one) and
  showing its actual content first; only ask when 2+ named templates exist
  to pick between, since that's a real content-level choice.

**`ps-*` skill files reference this section by name rather than re-deriving
their own noise-prevention wording.** If you are writing or editing a
`SKILL.md`, point back here instead of writing a new version of this rule.

**Known gap (see `dec-20260719-7d675b61`'s own weakest_link, stated
honestly, not hidden):** the self-check above is not machine-enforced for
the agentic surface — it depends on being applied. If a session ships a 4th
recurrence, this section did not prevent it by itself; treat that as
evidence the mechanism needs strengthening, not evidence to ignore it.

---
> Source: [malyshev/pysar](https://github.com/malyshev/pysar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
