## llm-pair

> >-


# llm-pair — calibrated pairing with a peer LLM

Pair the main agent with a **peer LLM** (the "pair") to raise quality through
independent drafting, adversarial review, and consensus — **without** spending
the most expensive model + maximum reasoning on every change.

The pair backend is **Codex** (GPT-5.x), invoked directly via the `codex` CLI
(`codex exec`). The concept is backend-agnostic; see
[Portability](#portability--swapping-the-pair-backend).

## The two problems this solves

1. **Cost calibration.** Run at max (top model + `xhigh`) on everything and a
   one-line fix gets the same super-review as a risky migration; the usage window
   evaporates. This skill picks a **(model, reasoning-effort) tier** sized to the
   task.
2. **Reliability.** Pin the pair to **read-only**, a fixed **working directory**,
   **structured JSON output**, and **verified non-empty results** — and **never
   confabulate** a result the pair didn't actually return.

---

## CONFIG — the tier ladder (edit this block to tune)

The classifier returns a **verdict**; this table maps verdict → (model, effort).
This table is the single source of truth. Verify model IDs against `codex` /
your account; if a model is rejected, fall back one rung toward the stronger
model and note it.

| verdict     | model                  | effort   | when                                                          |
|-------------|------------------------|----------|--------------------------------------------------------------|
| `skip`      | — (do not call pair)   | —        | trivial **and** zero risk signals — not worth pairing at all  |
| `trivial`   | `gpt-5.3-codex-spark`  | `low`    | rename, comment, one-line tweak, isolated copy/config change  |
| `small`     | `gpt-5.4-mini`         | `low`    | small, well-contained change, low blast radius                |
| `normal`    | `gpt-5.5`              | `medium` | ordinary feature/fix, moderate scope                          |
| `big`       | `gpt-5.5`              | `xhigh`  | large, cross-cutting, or high breakage risk                   |

**Effort floor is `low`.** `none`/`minimal` are rejected by the current Codex tool
config (incompatible with the `web_search` tool) — never emit them.

**Fixed overrides (never run the classifier for these):**

- **Planning** (overall plan, implementation plan, implementation pre-work) →
  always `gpt-5.5 @ xhigh`.
- **Blocker / error diagnosis** → classifier runs, but the **floor is `small`**
  (never `skip` or `trivial`).

**Risk signals** (any one blocks `skip` and biases the verdict upward, often to
`big`): auth / authz / sessions, DB migrations or schema, money / billing /
payments, shared libraries or widely-imported utilities, infra / CI / deploy,
public API or contract changes, security-sensitive code, concurrency.

---

## When to pair — granularity (read this first)

**Pairing happens at the work-item boundary, not per edit.** Over-triggering
burns the window faster than the old always-max default.

- A ticket with 3–4 work items → pair ~3–4 times: **plan once** up front, then
  **review each work item** as it reaches a coherent, complete state.
- A single work item with a dozen edits → do **not** pair on each edit. Pair once,
  on the completed unit.
- **Blocker pairing is reactive** — only when an error **survives 2+ fix
  attempts** *or* **cascades into new errors**. Not the first failing test.

### Proactive triggering

You do **not** wait for the user to say "pair with codex." During substantial
**defined task work** (a ticket, a feature, "do XYZ-123"), invoke this skill at
the boundaries above (plan / work-item review / blocker). The `skip` verdict +
work-item granularity keep small work cheap or unpaired, so erring toward
invoking is safe. Don't pair on pure questions or trivial one-off edits.

---

## The four contexts

| Context                         | Collaboration pattern              | Tier               |
|---------------------------------|------------------------------------|--------------------|
| Overall plan / implementation plan / pre-work | **Parallel-draft → converge** | **Fixed: `gpt-5.5 @ xhigh`** |
| Implementation review           | **Draft → advisory review → integrate** | Classifier    |
| Blocker / error diagnosis       | **Advisory diagnosis**             | Classifier, floor `small` |

The pair is **always read-only** — it advises; the main agent writes and
integrates. Only give the pair write capability if the user explicitly asks it to
patch something this turn.

### Context A — Planning (parallel-draft → converge)

Do **not** draft a plan and hand it to the pair for critique — that anchors it on
your framing and you lose the approach you didn't consider. Instead:

1. **Dispatch the pair's independent draft in the background** (run the `codex
   exec` call below, or an Agent, asynchronously). Give it the **same source
   material you have** (ticket, requirements, relevant code) but **not your
   draft**. Use the plan schema.
2. **While it runs, draft your own plan** independently.
3. **Collect both, diff the ideas** — agreements, real disagreements, anything one
   side caught that the other missed.
4. **Iterate to consensus** (see [Consensus loop](#the-consensus-loop)) on the real
   disagreements only.
5. Present the converged plan + a short "what the pair caught / what we rejected"
   note.

Tier is **fixed at max** — planning is where deep reasoning pays for itself.

### Context B — Implementation review (draft → advisory review → integrate)

Triggered when a **work item reaches a complete state**.

1. Build the signal pack and **classify**. If `skip`, say so and move on.
2. Run the pair **read-only** at the classified tier over the work item's diff,
   using the review schema. Findings ordered by severity, file:line, explicit "no
   findings → say so."
3. **Integrate** correct findings; push back on wrong ones.
4. Iterate to consensus; present findings first, then "what we fixed / rejected."

### Context C — Blocker / error diagnosis (advisory diagnosis)

Triggered only by the reactive threshold (2+ failed fixes **or** cascading errors).

1. Classify (floor `small`); wide blast radius → `big`.
2. Run the pair **read-only** with the error, what you've tried, and the relevant
   code, using the diagnosis schema. Ask for ranked root-cause hypotheses + a
   concrete next probe — not a blind patch.
3. Apply the fix yourself; if it fails, escalate a tier and re-pair.

---

## The classifier (cheap, structured)

Run one fast-model classification. In Claude Code, use the REPL `haiku()`
shorthand with the schema below (clean structured output, ~free). The classifier
returns the **verdict**; map verdict → model+effort via
[CONFIG](#config--the-tier-ladder-edit-this-block-to-tune) — do **not** let the
classifier invent a model ID.

```js
const SCHEMA = {
  type: 'object',
  properties: {
    verdict:   { type: 'string', enum: ['skip','trivial','small','normal','big'] },
    riskFlags: { type: 'array', items: { type: 'string' } },
    rationale: { type: 'string' }
  },
  required: ['verdict','riskFlags','rationale']
}
```

**Signal pack** — a compact summary, not the whole diff: the context (review vs
blocker) + one-line task summary; `git diff --stat`; files touched and ±lines;
visible risk flags (see CONFIG).

**Rules:** any risk flag → `skip`/`trivial` off the table; blocker → floor
`small`; **explicit user override wins** ("pair on max", `--tier big`, a named
model/effort) → skip the classifier. Surface the verdict + rationale in one line
before dispatching ("Classified review as `small` → gpt-5.4-mini @ low").

---

## Hardened invocation — the `codex exec` contract

Every pair call uses this exact shape (validated):

```bash
# 1. Write the output schema for this context to a temp file (review/diagnosis/plan).
# Prereq: a writable temp dir (--output-schema needs a FILE path). Capture stderr too.
SCHEMA="$(mktemp)"; ERR="$(mktemp)"; cat > "$SCHEMA" <<'JSON'
{ ...the context schema... }
JSON

# 2. Feed the prompt via stdin; read the clean JSON final message from stdout.
printf '%s' "$PROMPT" | codex exec \
  --skip-git-repo-check \
  -s read-only \
  -C "<ABSOLUTE repo/worktree path>" \
  -m "<model from ladder>" \
  -c model_reasoning_effort="<effort from ladder>" \
  --output-schema "$SCHEMA" \
  - 2>"$ERR"
# stdout = the final message as JSON conforming to the schema.
# On empty/invalid stdout, read "$ERR" to classify the failure
# (auth/quota/rate-limit -> Opus fallback; schema/model/sandbox -> report). Never discard it.
```

Flag rationale — these are the whole hardening story:

- **`-s read-only`** — sandboxes the pair's model-generated **shell** commands to read-only, killing the common "Codex edited my files" failure. Scope note: it constrains shell/file-edit tools, **not** a separately-configured write-capable MCP server — for a hard guarantee also restrict tools/config or run with a clean config.
- **`-C <abs path>`** — pins the working root. Kills "read the wrong worktree." Never omit.
- **`-m` + `-c model_reasoning_effort=`** — the calibrated tier. (There is no
  `--reasoning-effort` flag; effort is a config override.)
- **`--output-schema <file>`** — forces the final answer into structured JSON you
  can parse. Use the review / diagnosis / plan schema for the context.
- **`--skip-git-repo-check`** — required so the pair can run on **non-git** content (a docs/skill dir, a worktree). Trade-off: it removes Codex's "not a git repo" early-failure guard, so a typo'd `-C` won't fail fast — double-check the `-C` path is right.
- **prompt via stdin + `-`** — avoids shell-quoting hell for large diffs/prompts.
- **`2>"$ERR"` (capture, don't discard)** — Codex puts progress, reasoning, and MCP/hook noise on stderr; the clean final message goes to stdout. Send stderr to a **file**, not `/dev/null`: you need it to classify a failure (auth/quota/rate-limit vs schema/model) for the fallback and honest error reporting. (Do **not** use `--json` — that's the noisy full event stream.)
- For **free-form** turns (plan reconciliation, consensus follow-ups) drop
  `--output-schema` and read the clean final message text from stdout.

**Prompt shaping (inline — keep it tight):** state the role and boundary ("You are
reviewing this diff. Do NOT propose edits to files; report findings only."), give
only the artifact that matters (diff / error / requirements), and name the exact
output structure. Don't dump conversation history.

**Suggested schemas:**

```jsonc
// review
{ "type":"object","additionalProperties":false,
  "properties":{
    "findings":{"type":"array","items":{"type":"object","additionalProperties":false,
      "properties":{"severity":{"type":"string","enum":["critical","high","medium","low","nit"]},
        "file":{"type":"string"},"line":{"type":"string"},
        "issue":{"type":"string"},"suggestion":{"type":"string"}},
      "required":["severity","file","line","issue","suggestion"]}},
    "summary":{"type":"string"},"residualRisk":{"type":"string"}},
  "required":["findings","summary","residualRisk"] }

// diagnosis
{ "type":"object","additionalProperties":false,
  "properties":{
    "hypotheses":{"type":"array","items":{"type":"object","additionalProperties":false,
      "properties":{"cause":{"type":"string"},
        "likelihood":{"type":"string","enum":["high","medium","low"]},
        "evidence":{"type":"string"}},
      "required":["cause","likelihood","evidence"]}},
    "nextProbe":{"type":"string"},"summary":{"type":"string"}},
  "required":["hypotheses","nextProbe","summary"] }

// plan
{ "type":"object","additionalProperties":false,
  "properties":{
    "goal":{"type":"string"},"steps":{"type":"array","items":{"type":"string"}},
    "filesTouched":{"type":"array","items":{"type":"string"}},
    "risks":{"type":"array","items":{"type":"string"}},
    "openQuestions":{"type":"array","items":{"type":"string"}}},
  "required":["goal","steps","filesTouched","risks","openQuestions"] }
```

**Output verification — never confabulate:**

- If stdout is empty / not valid JSON / a failure, **retry once** (fresh call).
- If it still fails, **report the failure honestly and stop** — do **not** fabricate
  the pair's opinion or pass your own analysis off as the pair's.
- If the pair was never successfully invoked, do not generate a substitute — try
  the [fallback](#fallback--when-codex-is-unavailable) or report.

---

## Fallback — when Codex is unavailable

If `codex exec` fails with a **rate-limit / quota / auth** error (or `codex` is
not on PATH), fall back to the **previous-generation Opus** as the pair, with
**fresh context**:

- Spawn via the Agent tool / Workflow `agent()` with `model: 'opus'` and a
  **self-contained** prompt — only the problem statement + the concrete artifact
  (diff, files, error). **Not** your conversation history or conclusions.
- **Why both matter:** *model diversity* (a genuinely different model catches what
  you're blind to — reusing your own model shares your biases) and *fresh context*
  (a reviewer that shares your reasoning rubber-stamps it).
- **Tooling caveat:** the subagent `model` param exposes only the family (`opus`),
  which resolves to the **current** latest — pinning the exact previous minor
  (e.g. 4.7 when 4.8 is latest) may not be selectable. If it can't be pinned, use
  `opus` + fresh context anyway, and flag that a real Codex pass is still worth
  doing once limits reset.
- The same calibration (classifier tier) and read-only/advisor discipline apply —
  the fallback only swaps the backend.

---

## The consensus loop

Pairing is **iteration to consensus**, not a single pass:

1. Integrate the pair's correct points; push back, with reasoning, where it's wrong.
2. If material disagreements remain, send a **targeted** round — explain your
   position, ask for a specific response.
3. **Stop** when the pair's last round is "no further concerns," or remaining
   disagreements are explicit, recorded trade-offs for the user to arbitrate.
4. **Escalate a tier** if a review/blocker round won't converge after ~2 passes or
   the work proves bigger than first classified — re-pair from the higher rung.
5. Present the converged result + a short **"what the pair caught / what we
   rejected and why"** note.

Don't accept the pair blindly; don't dismiss it reflexively. The reconciled
position is the deliverable.

---

## First-run setup (one-time, per machine)

`llm-pair` triggers reliably only if the host CLAUDE.md tells the agent to use it,
and `npx skills` does not display post-install messages. So on first invocation:

- Check whether the user's CLAUDE.md (global or project) already contains an
  `llm-pair` pairing rule.
- If not, **offer** to add the wiring block from the README's "Setup" section (the
  proactive-trigger + deferral rules). **Do not edit CLAUDE.md without consent.**
- Confirm prerequisites: the `codex` CLI on PATH (e.g. `brew install codex`) and
  authenticated; an Opus-capable agent for the fallback.

---

## Portability — swapping the pair backend

Designed to be open-sourced (`iotashan/llm-pair`). Bindings:

- **Classifier** — Claude Code's REPL `haiku()`. Any fast model with structured
  output works; keep the schema, swap the call.
- **Pair backend** — the `codex` CLI (`codex exec`). No Claude Code plugin
  required — only the standalone `codex` binary on PATH. To add another backend,
  replace the [invocation contract](#hardened-invocation--the-codex-exec-contract)
  and the model IDs in [CONFIG](#config--the-tier-ladder-edit-this-block-to-tune);
  contexts, classifier, fallback, and consensus loop are unchanged.
- **Fallback** — any Opus-capable agent (Claude Code's Agent/Workflow tools).

Local install wiring (skill symlinks, CLAUDE.md edits) is **not** part of the
shippable skill — see the README "Setup" section.

---
> Source: [iotashan/llm-pair](https://github.com/iotashan/llm-pair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
