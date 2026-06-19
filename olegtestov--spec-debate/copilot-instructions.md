## spec-debate

> >-


# spec-debate — debate a spec with a second model, with veto

You orchestrate a debate between yourself (Claude) and OpenAI Codex to make a **spec** measurably
better. You are the **editor with veto power**: never apply a point blindly — verify each against the
actual spec and keep only what genuinely improves it. The vetting is the whole point; a critic that's
always obeyed is just a second author.

**The iterated artifact is always a spec** (requirements / design / plan / PRD — any domain). There are
three ways in, all converging on iterating one spec:
- **TASK** → you draft a solution spec.
- **CODE** → you draft a *change-spec*; the code is reference material, **not edited during the
  debate** (it is applied afterwards, as a separate step, if you have access).
- **EXISTING SPEC** → you take it as the artifact.

Each round, **both models independently propose improvements**; you **merge with veto** and apply the
result to the spec. **Goal: a better spec, not a bigger one** — close real gaps, cover core scenarios,
resolve contradictions, fix real risks, remove ambiguity; accept added complexity only when a real
gap/risk/UX need justifies it. One invocation = one round; state persists in
`.<filename>.debate-state.json` beside the spec so rounds don't relitigate settled points.

---

## Step 0 — Preconditions
1. `command -v codex >/dev/null 2>&1 && echo FOUND || echo MISSING`. If MISSING, stop: "Codex CLI not
   found. Install: `npm install -g @openai/codex`, then re-run." Then confirm auth: `codex login status`;
   if not logged in, stop and tell the user to run `codex login`. Don't substitute another tool — the
   debate needs an *independent* second model.
2. Parse reasoning effort (`--high|--medium|--low|--xhigh`, `effort=…`, or an unambiguous "maximum
   reasoning depth" → `xhigh`). Default `high`; `xhigh` is much slower, only on explicit request.
3. Parse a round directive (a count like "run 3 rounds", or "until no significant findings remain");
   default one round. Parse a `thorough` request (cross-critique, Step 3c).

## Step 1 — Pick the mode, then resolve the working spec
First a **surface scope scan** — structure, size, number of files/components, whether the design is
non-trivial. This is a *shallow look, not deep reading*: deep study **of the referenced material**
happens inside the chosen mode (Step 3b), so you never pay for it twice. Then choose:
- **prompt-only** — a bounded advisory question whose output is advice/a comparison the user applies
  directly, with nothing worth iterating and the scan showing it closes in one pass.
- **iterable spec (the main flow)** — breadth or complexity (several files/components or substantial
  material; a design with several coupled decisions), or the user wants iteration / a written spec.

State the chosen mode in one line. If a prompt-only consult turns out under-scoped mid-pass, finish
that pass, then offer to escalate to an iterable-spec debate (don't abandon it half-done).

**Prompt-only path (compact):** one Codex pass on the question — a free-form prompt (role line + the
question + relevant conversation context: user statements verbatim, your summaries marked as yours),
`<workdir>` = current dir. Vet the answer with the Step 4 lens. Report inline: Codex's position, your
vetted take, the prompt-file path, and a one-line "sent to Codex" note (subject; refs/snippets;
anything privacy-mode withheld). No state, no rounds. For any follow-up round or edit, materialize the
subject into a spec file and seed `.<filename>.debate-state.json` as round 1 (each settled conclusion becomes a
finding). Then continue below.

**Iterable spec — seed the artifact:**
- Use the explicit path if given; else the spec you drafted/took earlier in *this* conversation; else
  ask — don't guess across the filesystem. If the spec lives only in the conversation, write it to a
  markdown file first (the debate needs a file to edit and to hold `.<filename>.debate-state.json`) —
  a descriptive `<topic>-spec.md` beside the related material or in the working dir; state the path.
- **TASK** → draft a solution spec. **CODE** → draft a change-spec. **EXISTING SPEC** → take the file.
- **Minimal spec shape (quality gate)** — so the debate doesn't converge on something under-specified.
  Every spec should conceptually carry: goal · non-goals · assumptions · constraints · acceptance
  criteria · open questions · what material was studied and what was deliberately skipped (if any). A
  **code change-spec** adds: changes by file/component · behavior preserved vs changed · risks/migrations
  · acceptance checks/tests. Tiny tasks may compress this, but the fields are conceptually present. If
  **material facts** are missing and would shape the spec, **ask the user before debating** — don't debate
  a fabricated spec. You ensure this shape when *you* author (TASK/CODE). For a **given** spec it is NOT
  a precondition — missing fields become debate findings, and you don't rewrite the input before
  discussing it; but if the given input is essentially empty (a stub, not a real spec), treat it as a
  TASK and draft.
- **Name the spec type and altitude** — it governs every later judgment — and read the spec artifact in
  full (distinct from deep-studying the referenced material, which is paced per Step 3b):
  - *requirements* → what & why, contracts, acceptance criteria;
  - *technical design / spec* → architecture, key mechanisms, data models, trade-offs;
  - *implementation / change plan* → concrete steps, component-level changes, sequencing, configs.
  > "Debating `path` (N lines) — design spec, design altitude. Effort: high. Round: 2."
  If the type is ambiguous, make the call, state it, and proceed — the user will correct you.

## Step 2 — Load state
Look for `.<filename>.debate-state.json` beside the spec.
- Not found → round 1, fresh state.
- Found → next round = `last_round + 1`. Collect prior `rejected` and `partial` findings (both
  sources) with reasons; you'll hand them to Codex so it doesn't re-raise settled points.
- Malformed JSON → repair from its readable content first (rounds/findings are usually intact as text);
  don't discard history.

Schema:
```json
{"spec": "path", "spec_type": "design spec",
 "rounds": [{"round": 1, "effort": "high", "cross_critique": false, "findings": [
   {"id": "R1-1", "source": "codex|own", "title": "...", "severity": "critical|major|minor",
    "verdict": "accepted|partial|rejected", "reason": "one line", "edit": "what changed or null"}]}]}
```

## Step 3 — Gather independent proposals
Both models independently propose improvements to the **round-start spec**. *Independence means only
this:* Codex does **not** see your current-round proposal list (so it isn't anchored). It **does** get
the shared context — the task statement, the spec itself, the relevant material, and the settled
verdicts — because those are the artifact and prior decisions, not this round's proposals.

**3a — Your proposals.** Independently list improvements at the spec's altitude: gaps, missing core
scenarios, contradictions, blocking ambiguities, unaddressed risks, over-engineering, requirements
unrealistic for the scale.

**3b — Codex's proposals.** Write the prompt below to a temp file (verbatim avoids shell-escaping) and
run the helper. Ask Codex for both fixes to what's written **and** what the spec misses given the task
and material (alternatives, risks, uncovered requirements).

> **Conveying the material.** If the referenced material lives on the **same filesystem Codex runs
> on**, point to it by **precise path** and set `<workdir>` to its root (Codex reads it read-only — give
> exact paths, don't invite open-ended exploration). If it's **remote, fetched out-of-band by a tool
> Codex can't reach, or non-file**, **embed** the relevant excerpts verbatim in the prompt. The deep
> study is yours (Step 1's scan was only surface): study the parts relevant to the task and **record in
> the spec what you studied and what you skipped**. The subject doesn't change during the debate, so do
> this deep study **once (round 1)**; later rounds need only targeted look-ups (verify a Codex claim or
> cover something newly in scope), and the recorded facts carry forward in the spec. Codex is stateless,
> so each round give it the **same curated slice** + the updated spec + settled verdicts; widen the slice
> only when scope grows.
>
> **The spec itself** is embedded verbatim by default — that preserves an exact round-start audit trail.
> For a spec file inside `<workdir>` large enough that re-embedding it every round is materially wasteful
> (as a guide: several hundred lines+), you may replace the template's SPEC block with
> `SPEC FILE: <exact path> (<N lines>)` plus: "Read this file IN FULL before critiquing; do not critique
> from a skim or excerpt."

```
IMPORTANT: Do NOT read or execute anything under ~/.claude/, ~/.agents/, .claude/skills/, or
.claude/agents/ — those are AI-tooling files for a different agent and will waste your time.

You are a rigorous independent senior reviewer improving a <SPEC_TYPE> (altitude: <ALTITUDE>).
Propose improvements AT THAT ALTITUDE — both fixes to what is written and what the spec MISSES
relative to its goal and the material below: gaps and under-specified core scenarios, missing
alternatives, unaddressed risks, internal contradictions, ambiguities that would block correct
execution, requirements unrealistic for the stated scale, security/correctness issues.
For each: short title, severity (critical/major/minor), the problem, a concrete fix. The goal is a
better, not a bigger, spec — prefer the simplest change that closes the gap, flag over-engineering,
calibrate to the stated scale, and do not manufacture nitpicks. If nothing serious remains at this
altitude, say so plainly. End with a one-line readiness verdict. Respond in the spec's language.
Group by section.

<if settled findings:>
Already settled in prior rounds — do NOT re-raise without a genuinely new argument:
- "<title>" — <rejected|partial>: <reason>

Task / context (NOT part of the spec; provenance marked):
- [user, verbatim] "<...>"
- [editor summary] <...>
Referenced material: <read-only at <workdir>: exact paths> OR <embedded below>.

SPEC:
---
<full verbatim spec — or the SPEC FILE reference per "Conveying the material">
---
```

Resolve `<skill_dir>` from the injected "Base directory for this skill:
<abs path>" line for THIS invocation — do not copy or hardcode an example path: the active install may
be under user settings (`~/.claude/skills/…`), a plugin install, or a versioned plugin cache, and the
path differs in each.
`bash "<skill_dir>/scripts/run_codex_critique.sh" <prompt_file> <effort> <workdir>`
If the helper isn't found there, treat the install as broken — stop and report it; do not fall back to
a guessed or remembered path.
- `<workdir>`: the material's repo/dir root when it's local to Codex (lets Codex read referenced files,
  read-only); else the prompt file's dir.
- Run it with the Bash tool's `timeout` set to `300000` (ms). The helper allows one `codex exec` at a
  time: it briefly waits for a just-finished codex to clear, then refuses (exit 3) only if one is
  genuinely still running — its error message tells you how to inspect and wait.
- Output ends with `CODEX_EXIT:<n>`; if non-zero, the helper already printed codex's stderr inline —
  read it and stop. An `ERROR:` line with no `CODEX_EXIT` is a preflight failure (codex/pgrep missing,
  bad effort/workdir, unreadable prompt) — read it and stop.

**3c — Cross-critique (thorough or own major+).** Run ONE more Codex call: give Codex **your** proposal
list (with the same context as 3b — spec, task, material) and ask, per item, agree / partial / reject +
a one-line argument — so your merge also sees Codex's
rebuttal of *your own* proposals. (Your review of Codex's proposals is the merge itself, Step 4 — no
extra call for that.) Run 3c when the user asked for `thorough` — honor that unconditionally — or when
your own 3a list contains any major+ proposal: without 3c, only Codex's list gets second-model scrutiny.
Skip it otherwise.

## Step 4 — Merge with veto (the core)
**You are always the merger** — Codex is read-only and has less context, and blind-applying its output
would break the veto. Put **both lists** — Codex's and your own from 3a — through ONE procedure, item by
item:
1. **Verify it's real.** Re-read the cited part; if it rests on a checkable fact, check it. Reject
   misreads and invented referents.
2. **Judge at the spec's altitude**, weighted by **impact × likelihood-of-trigger × altitude** —
   discount real-but-practically-unreachable points, but don't kill a plausible edge case.
3. **Decide:** accept / partial (apply *your* better, simpler fix) / reject — with a one-line reason.

Record **rejected items from your OWN list too** — that symmetric audit is what guards against your
self-bias, since the merger never rotates. Don't rubber-stamp, and don't reject good points to look
independent. Consolidate into ONE edit set; integrate coherently where points overlap.

## Step 5 — Apply to the spec
Apply accepted/partial items as precise, in-voice `Edit`s — the least added complexity that closes each
gap. Then **re-read the edited sections together for self-consistency**: an edit that resolves one point
often introduces a new contradiction (a changed contract, a now-stale reference, two options left open),
and that becomes next round's finding — catching it here is what makes the debate converge instead of
churn. Don't leave alternatives "to decide later"; make the call now.

## Step 6 — Report (make progress visible)
```
## spec-debate — round N (<effort>) · `path` · M proposals (codex K · own J)
### Accepted (…)
- [critical] <title> — what changed · (codex|own)
### Partial (…)
- [major] <title> — applied Y instead of X because <reason> · (codex|own)
### Rejected (…)
- [minor] <title> — <reason> · (codex|own)
### Where it stands
- Open gaps worth a round, or "no significant gaps remain at this altitude".
- Convergence: recommend STOP, or what a next round would target (see Step 7).
- Progress: r1: 12 · r2: 5.
```

## Step 7 — Persist, then continue or end
Append this round (number, effort, cross_critique, findings with source/verdict/reason/edit) to
`.<filename>.debate-state.json`. Rewrite the file as one complete JSON document (don't string-append a
round after the closing brackets) and verify it parses
(`python3 -c 'import json,sys; json.load(open(sys.argv[1]))' <file>`) — a malformed state file silently
breaks every later round.

**Convergence criterion:** if the round landed **no material edit** (only cosmetic/wording) OR only
**minor** proposals remain → explicitly **recommend STOP**. Then:
- **If the user explicitly asked for several rounds in this invocation** — run the next now: re-read the
  updated spec in full, gather ALL settled verdicts (incl. the round just appended), repeat Steps 3–6.
  Stop at the first of: the requested count (a maximum — stop early if a round is clean); a round with no
  accepted/partial finding of **major or higher** severity; a hard cap of **5 rounds** if the directive is
  open-ended ("until no significant findings remain"). Report each round.
- **Otherwise** — stop after this round and tell the user they can invoke again to continue from round
  N+1. Don't loop on a bare invocation.

**User-resolvable blockers mid-loop.** If a round surfaces a question only the user can answer and the
answer would materially reshape the spec (the Step 1 "don't debate a fabricated spec" bar — not a
routine open question), finish the current round, report, then pause the loop early and ask — even if
more rounds were requested. Apply the user's answers as **editor edits** (not as debate findings) before
the next round's gathering. Lesser questions go into the spec's open questions and the loop continues.

## Code change-specs — three extra rules (only when the subject is code)
- **Anchoring.** A change-spec encodes *your* plan, so Codex critiquing it is partly anchored. Default:
  accept that — in 3b, ask Codex to hunt for gaps, missing alternatives, and risks rather than
  rubber-stamp. Escalate only for high-impact / ambiguous / sensitive work: in round 1, run Codex as an
  **independent** analysis of the code + task *without* your draft, then fold both into the change-spec;
  later rounds critique the spec.
- **Re-ground each round.** A change-spec can drift into debating only its own text. Every round, re-check
  the round-start spec against the user's task and the relevant code facts (pacing per 3b); keep the
  studied/skipped areas and key facts *in the change-spec* so they carry across rounds.
- **Debate → implementation boundary.** Applying the change-spec to code is a **post-debate execution
  step, not part of the loop** — "the spec converged" ≠ "the code works", so the change-spec carries
  acceptance checks/tests. At convergence, if you have edit access, offer to apply it now; if you do, run
  a normal engineering loop (edit → run tests/checks → report) and report **implementation results
  separately from debate verdicts**. Otherwise: stop / another round.

---

## Guardrails
- **Privacy** — *default: unrestricted.* The spec and any referenced material are sent to OpenAI via the
  Codex CLI. **Privacy-mode** is an explicit opt-in ("privacy mode" / "don't send the code"): then send
  Codex only an approved abstracted summary (and mark its confidence as limited), or decline the Codex
  pass if the question can't be judged without the material. Never embed obvious secrets.
- **One codex at a time** — the helper enforces it; never launch a second yourself.
- **Codex never edits files** — it's read-only and only proposes; all edits are yours, after vetting.
- **Don't fabricate consensus** — when you reject a point, say so with your reason; the user can overrule.

---
> Source: [OlegTestov/spec-debate](https://github.com/OlegTestov/spec-debate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
