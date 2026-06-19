## agentic-code-reasoning-skills

> >


# Agentic Code Reasoning

## Purpose
Reason about code behavior using structured semi-formal analysis without executing repository code.

This skill enforces a certificate-based reasoning process: you must state premises, trace concrete code paths with file:line evidence, and derive formal conclusions. You cannot skip sections or make unsupported claims.

## Modes
- `compare` — determine if two changes produce the same behavior
- `diagnose` — find the root cause of a bug in a small number of files
- `explain` — answer a code question with verified evidence
- `audit-improve` — review code for security, API misuse, or maintainability

Choose a mode before exploring files. If unsure, prefer `explain`.

### Mode selection guide
| Trigger | Mode |
|---------|------|
| "Are these two patches/implementations equivalent?" | `compare` |
| "Where is the bug?" / failing test / single defect | `diagnose` |
| "What does this code do?" / "Why does X happen?" | `explain` |
| "Is this code secure?" / "Review for issues" | `audit-improve` |

### Activation gates

Before selecting a mode, check whether this skill is appropriate for the task. **Do not activate** this skill when:

- The task requires **broad file enumeration** (e.g., "list all files that need to change for this refactor"). This skill is designed for deep analysis of a small number of files, not for wide-coverage listing.
- The task is a **large-scale structural change** spanning many files (e.g., directory reorganization, rename propagation across a monorepo).
- The expected output is a **flat list of files** rather than a reasoned diagnosis with evidence.

For `diagnose` mode specifically:

| Condition | Use `diagnose` | Do NOT use `diagnose` |
|-----------|---------------|----------------------|
| Root cause scope | Likely 1–5 files | Many files (10+) across the codebase |
| Task nature | Single defect, specific test failure, error trace | Broad refactoring, feature addition, structural reorganization |
| Expected output | Ranked root cause with file:line evidence | Exhaustive list of files to modify |
| Evidence style | Deep code path tracing | Directory-level pattern matching |

If the task does not fit `diagnose`, consider using the agent without this skill — unrestricted exploration may produce better results for broad enumeration tasks.

---

## Core Method
Apply this process in every mode. **Complete each section in order. Do not write a later section before completing earlier ones.**

When a certificate template exists for your selected mode (see the mode sections below), **use that template as your primary guide** — it is the concrete implementation of Steps 1–6 for that mode.

### Step 1: Task and constraints
Write a short task statement and list constraints (e.g., no repository execution, static inspection only, file:line evidence required).

### Step 2: Numbered premises
Before concluding anything, write numbered premises grounded in known facts.

```
P1: [fact about the task, inputs, or expected behavior]
P2: [fact about relevant files, tests, or specifications]
P3: ...
```

Do not treat guesses as premises. Every later claim must reference a premise by number.

### Step 3: Hypothesis-driven exploration
Exploration priority is not a fixed reading order; choose the next action by discriminative power — what unresolved uncertainty it resolves.
Before opening any file, write:

```
HYPOTHESIS H[N]: [what you expect to find and why]
EVIDENCE: [what supports this hypothesis — cite premises or prior observations]
CONFIDENCE: high / medium / low
```

After reading, record:

```
OBSERVATIONS from [filename]:
  O[N]: [finding with file:line]
  O[N]: [another finding with file:line]

HYPOTHESIS UPDATE:
  H[M]: CONFIRMED / REFUTED / REFINED — [explanation]

UNRESOLVED:
  - [remaining questions]

NEXT ACTION RATIONALE: [why the next file or step is justified]
OPTIONAL — INFO GAIN: [what uncertainty this action resolves; which hypothesis/claim it would confirm vs refute]
```

Steps 3 and 4 work together: Step 3 is your real-time exploration journal. Step 4 is the accumulated function-behavior record you build *during* Step 3 — **add a row to Step 4 each time you read a function definition in Step 3.** Do not reconstruct the table from memory after the fact.

### Step 4: Interprocedural tracing
Update this table **in real time during Step 3** — add each row the moment you read a function definition. Do not write this table all at once from memory.

For every function or method encountered on a relevant code path, record:

| Function/Method | File:Line | Behavior (VERIFIED) | Relevance to test |
|-----------------|-----------|---------------------|-------------------|
| [name] | [file:N] | [actual behavior after reading the definition] | [which test(s) and why this function is on the relevant path] |

**Rules:**
- Read the actual definition. Do not infer behavior from the name.
- Mark the Behavior column VERIFIED only after reading the source.
- If source is unavailable (third-party library), mark UNVERIFIED and note the assumption. Search for type signatures, documentation, or test usage as secondary evidence. Optionally probe language behavior with an independent script.
- Trace through conditionals, mapping tables, and configuration — not just the happy path.
- For exception handling inside loops or multi-branch control flows: after recording the inferred behavior, ask "if this trace were wrong, what concrete input would produce different behavior?" Trace that input through the code before finalizing the row.

### Step 5: Refutation check (required)
This step is **mandatory**, not optional.

**Scope**: Apply counterfactual reasoning not only at the final conclusion, but at every key intermediate claim — especially:
- "No test exercises this difference" — before asserting this, describe what such a test would look like and show you searched for exactly that pattern.
- "This behavior is X" for a non-trivial control flow — before asserting this, ask what evidence would exist if the behavior were not X.
- "These test outcomes are identical/different" — before asserting this, state what evidence would refute it.

For `compare` and `audit-improve`:
```
COUNTEREXAMPLE CHECK:
If my conclusion were false, what evidence should exist?
- Searched for: [what]
- Found: [what — cite file:line]
- Result: REFUTED / NOT FOUND
```

For `explain` and `diagnose`:
```
ALTERNATIVE HYPOTHESIS CHECK:
If the opposite answer were true, what evidence would exist?
- Searched for: [what]
- Found: [what — cite file:line]
- Conclusion: REFUTED / SUPPORTED
```

### Step 5.5: Pre-conclusion self-check (required)

Before writing the formal conclusion, check each item below. If any answer is **NO**, fix it before Step 6.

- [ ] Every PASS/FAIL or EQUIVALENT/NOT_EQUIVALENT claim traces to a specific `file:line` — not inferred from function names.
- [ ] Every function in the trace table is marked **VERIFIED**, or explicitly **UNVERIFIED** with a stated assumption that does not alter the conclusion.
- [ ] The Step 5 refutation or alternative-hypothesis check involved at least one actual file search or code inspection — not reasoning alone.
- [ ] The conclusion I am about to write asserts nothing beyond what the traced evidence supports.

### Step 6: Formal conclusion
Write a conclusion that:
- References specific numbered premises and claims (e.g., "By P1 and C2…")
- States what was established
- States what remains uncertain or unverified
- Assigns a confidence level: HIGH / MEDIUM / LOW

---

## Compare

Goal: determine whether two changes produce the same relevant behavior.

*This template implements Steps 1–6 of the Core Method for `compare` mode.*

### Certificate template

Complete every section. Do not skip to FORMAL CONCLUSION without completing ANALYSIS.

```
DEFINITIONS:
D1: Two changes are EQUIVALENT MODULO TESTS iff executing the relevant
    test suite produces identical pass/fail outcomes for both.
D2: The relevant tests are:
    (a) Fail-to-pass tests: tests that fail on the unpatched code and are
        expected to pass after the fix — always relevant.
    (b) Pass-to-pass tests: tests that already pass before the fix — relevant
        only if the changed code lies in their call path.
    To identify them: search for tests referencing the changed function, class,
    or variable. If the test suite is not provided, state this as a constraint
    in P[N] and restrict the scope of D1 accordingly.

STRUCTURAL TRIAGE (required before detailed tracing):
Before tracing individual functions, compare the two changes structurally:
  S1: Files modified — list files touched by each change. Flag any file
      modified in one change but absent from the other.
  S2: Completeness — does each change cover all the modules that the
      failing tests exercise? If Change B omits a file that Change A
      modifies and a test imports that file, the changes are NOT EQUIVALENT
      regardless of the detailed semantics.
  S3: Scale assessment — if either patch exceeds ~200 lines of diff,
      prioritize structural differences (S1, S2) and high-level semantic
      comparison over exhaustive line-by-line tracing. Exhaustive tracing
      is infeasible for large patches and produces unreliable conclusions.

If S1 or S2 reveals a clear structural gap (missing file, missing module
update, missing test data), you may proceed directly to FORMAL CONCLUSION
with NOT EQUIVALENT without completing the full ANALYSIS section.

PREMISES:
P1: Change A modifies [file(s)] by [specific description]
P2: Change B modifies [file(s)] by [specific description]
P3: The fail-to-pass tests check [specific behavior]
P4: The pass-to-pass tests check [specific behavior, if relevant]

ANALYSIS OF TEST BEHAVIOR:

For each relevant test:
  Test: [name]
  Claim C[N].1: With Change A, this test will [PASS/FAIL]
                because [trace from changed code to test assertion outcome — cite file:line]
  Claim C[N].2: With Change B, this test will [PASS/FAIL]
                because [trace from changed code to test assertion outcome — cite file:line]
  Comparison: SAME / DIFFERENT outcome

For pass-to-pass tests (if changes could affect them differently):
  Test: [name]
  Claim C[N].1: With Change A, behavior is [description]
  Claim C[N].2: With Change B, behavior is [description]
  Comparison: SAME / DIFFERENT outcome

EDGE CASES RELEVANT TO EXISTING TESTS:
(Only analyze edge cases that the ACTUAL tests exercise)
  E[N]: [edge case]
    - Change A behavior: [specific output/behavior]
    - Change B behavior: [specific output/behavior]
    - Test outcome same: YES / NO

COUNTEREXAMPLE (required if claiming NOT EQUIVALENT):
  Test [name] will [PASS/FAIL] with Change A because [reason]
  Test [name] will [FAIL/PASS] with Change B because [reason]
  Diverging assertion: [test_file:line — the specific assert/check that produces a different result]
  Therefore changes produce DIFFERENT test outcomes.

NO COUNTEREXAMPLE EXISTS (required if claiming EQUIVALENT):
  If you already observed a semantic difference, name that difference first and test whether one concrete relevant test/input reaches the same assertion outcome on both sides.
  When claiming EQUIVALENT after observing a semantic difference, anchor the no-counterexample argument to that exact difference with one concrete relevant test/input and the same traced assertion outcome on both sides; otherwise mark the impact UNVERIFIED.
  If NOT EQUIVALENT were true, a counterexample would be this specific test/input diverging at [assert/check:file:line].
  I searched for exactly that anchored pattern:
    Searched for: [specific pattern — the observed difference, relevant test/input, and assertion/check]
    Found: [result — cite file:line, or NONE FOUND with search details]
  Conclusion: no counterexample exists because [brief reason]

FORMAL CONCLUSION:
By Definition D1:
  - Test outcomes with Change A: [PASS/FAIL for each test]
  - Test outcomes with Change B: [PASS/FAIL for each test]
  - Since outcomes are [IDENTICAL/DIFFERENT], changes are
    [EQUIVALENT/NOT EQUIVALENT] modulo the existing tests.

ANSWER: [YES equivalent / NO not equivalent]
CONFIDENCE: [HIGH / MEDIUM / LOW]
```

### Compare checklist
- **Structural triage first**: compare modified file lists, check for missing modules or test data before any detailed tracing
- For large patches (>200 lines), rely on structural comparison and high-level semantic analysis rather than exhaustive line-by-line tracing
- Identify changed files for both sides
- Identify fail-to-pass AND pass-to-pass tests
- For each function called in changed code, read its definition and record in the interprocedural trace table (Step 4)
- Trace each test through both changes separately before comparing
- When a semantic difference is found, trace at least one relevant test through the differing path before concluding it has no impact
- Provide a counterexample (if different) or justify no counterexample exists (if equivalent)

---


## Diagnose

Goal: identify the root cause of a single defect, not just the crash site.

**Scope constraint:** This mode is designed for defects whose root cause resides in a small number of files (typically 1–5). For tasks requiring broad file enumeration across many files, do not use this mode — the structured analysis will over-constrain the output and reduce coverage.

### Certificate template

Complete phases in order. Each phase depends on the previous one.

```
PHASE 1: TEST / SYMPTOM SEMANTICS

What does the failing test or bug report describe?
State as formal premises:
  PREMISE T1: The test calls [X.method(args)] and expects [behavior]
  PREMISE T2: The test asserts [condition]
  PREMISE T3: The observed failure is [error type / wrong output / hang]
  ...

PHASE 2: CODE PATH TRACING

Trace the execution path from the test entry point into production code.
For each significant method call, record:

| # | METHOD | LOCATION | BEHAVIOR | RELEVANT |
|---|--------|----------|----------|----------|
| 1 | ClassName.method(params) | file:line | [verified behavior] | [why it matters to PREMISE T[N]] |
| 2 | ... | ... | ... | ... |

Build the call sequence: test → method1 → method2 → ...

PHASE 3: DIVERGENCE ANALYSIS

For each code path traced, identify where the implementation diverges
from the test's expectations. State as formal claims:

  CLAIM D1: At [file:line], [code] produces [behavior]
            which contradicts PREMISE T[N] because [reason]
  CLAIM D2: ...

Each claim MUST reference a specific PREMISE and a specific code location.

PHASE 4: RANKED PREDICTIONS

Based on divergence claims, produce ranked predictions:
  Rank 1 ([confidence]): [file:line range] — [description]
    Supporting claim(s): D[N]
    Root cause / symptom: [which one]
  Rank 2 ([confidence]): ...
```

### Exploration protocol
Use the hypothesis-driven format from Step 3 during exploration. Number hypotheses H1, H2… and observations O1, O2… for traceability.

### Diagnose checklist
- State what the failing behavior expects (Phase 1)
- Trace from entry point toward production code with per-method records (Phase 2)
- Every divergence claim must reference a specific premise (Phase 3)
- Rank candidates and cite supporting claims (Phase 4)
- Distinguish symptom site from root cause — if the crash site differs from the origin of incorrect state, investigate upstream
- Check for indirection: is the bug in a class not directly called by the test?

---

## Explain

Goal: answer a code question with verified semantic evidence.

### Certificate template

Complete every section. Do not write FINAL ANSWER before ALTERNATIVE HYPOTHESIS CHECK.

```
QUESTION: [restate the question]

FUNCTION TRACE TABLE:
| Function/Method | File:Line | Parameter Types | Return Type | Behavior (VERIFIED) |
|-----------------|-----------|-----------------|-------------|---------------------|
| [function1]     | [file:N]  | [param types]   | [ret type]  | [ACTUAL behavior]   |
| [function2]     | [file:N]  | [param types]   | [ret type]  | [ACTUAL behavior]   |

DATA FLOW ANALYSIS:
Variable: [key variable name]
  - Created at: [file:line]
  - Modified at: [file:line(s), or NEVER MODIFIED]
  - Used at: [file:line(s)]

(Repeat for each key variable)

SEMANTIC PROPERTIES:
Property 1: [e.g., "map is immutable after initialization"]
  - Evidence: [specific file:line]
Property 2: ...
  - Evidence: [specific file:line]

ALTERNATIVE HYPOTHESIS CHECK:
If the opposite answer were true, what evidence would exist?
  - Searched for: [what you looked for]
  - Found: [what you found — cite file:line]
  - Conclusion: REFUTED / SUPPORTED

FINAL ANSWER:
[answer with explicit evidence citations]

CONFIDENCE: [HIGH / MEDIUM / LOW]
```

### Explain checklist
- Read actual definitions — do not infer behavior from names
- Fill every row in the function trace table with VERIFIED behavior
- Track key variables from creation through modification to usage
- Identify semantic properties with per-property file:line evidence
- Check the opposite answer before finalizing
- After identifying an edge case, verify whether downstream code already handles it before reporting it as a finding
- State uncertainty when downstream behavior is not fully verified

---

## Audit-Improve

Goal: inspect code for risks or improvement opportunities, grounded in traced evidence.

### Sub-modes
- `security-audit` — injection, auth bypass, path traversal, secrets, unsafe defaults
- `refactor-review` — oversized units, duplication, mixed responsibilities, fragile flow
- `code-smell-check` — hidden coupling, dead branches, poor naming, hard-to-test design
- `api-misuse-check` — incorrect API usage, wrong assumptions about library semantics

### Sub-mode focus

| Sub-mode | Primary question | Key requirement |
|---|---|---|
| `security-audit` | Is this unsafe operation reachable? | Verify a concrete call path for every confirmed finding |
| `refactor-review` | What is the safest minimal change? | Always propose the smallest effective refactoring first |
| `code-smell-check` | Is there concrete coupling or testability harm? | Trace coupling to a specific dependency — do not flag patterns without evidence |
| `api-misuse-check` | Does the usage violate the documented contract? | Read the API definition or documentation before claiming misuse |

### Certificate template

```
REVIEW TARGET: [file(s) / module / component]
AUDIT SCOPE: [which sub-mode(s) and what property is being checked]

PREMISES:
P1: [fact about the code's purpose or expected security properties]
P2: [fact about the API contract or framework requirements]
...

FINDINGS:

For each finding:
  Finding F[N]: [title]
    Category: security / refactor / smell / api-misuse
    Status: CONFIRMED / PLAUSIBLE (needs more evidence)
    Location: [file:line range]
    Trace: [code path that leads to this issue — cite file:line at each step]
    Impact: [what can go wrong and under what conditions]
    Evidence: [specific file:line proof]

COUNTEREXAMPLE CHECK:
For each confirmed finding, did you verify it is reachable?
  F[N]: Reachable via [call path] — YES / UNVERIFIED

RECOMMENDATIONS:
R[N] (for F[N]): [specific fix or mitigation]
  Risk of change: [what could break]
  Minimal safe change: [smallest effective fix]

UNVERIFIED CONCERNS:
- [issues that need more context or are speculative]

CONFIDENCE: [HIGH / MEDIUM / LOW]
```

### Audit-Improve checklist
- Define the review target and scope clearly
- State the risk or quality property being checked as a premise
- Trace the relevant code path — do not flag isolated lines without context
- Separate CONFIRMED from PLAUSIBLE findings
- For each confirmed finding, verify it is reachable via a concrete call path
- For refactoring, propose the safest minimal change first
- Do not report speculative security issues as confirmed vulnerabilities
- For API misuse, read the actual API definition or documentation before claiming misuse

---

## Guardrails

### From the paper's error analysis
1. **Do not assume behavior from names.** Read the actual function definition. The canonical failure: assuming Python's builtin `format()` when a module-level function with different semantics shadows it.
2. **Do not claim test outcomes without tracing.** Trace each test through the relevant code path before asserting PASS or FAIL.
3. **Do not confuse symptom with root cause.** A crash site (e.g., StackOverflowError in a recursive method) may not be the origin of incorrect state. Trace upstream to find where the bad state was created.
4. **Do not dismiss subtle differences.** If you find a semantic difference between compared items, trace at least one relevant test through the differing code path before concluding the difference has no impact.
5. **Do not trust incomplete chains.** After building a reasoning chain, verify that downstream code does not already handle the edge case or condition you identified — e.g., via exception handlers, default values, or guard clauses. Confident-but-wrong answers often come from thorough-but-incomplete analysis.
6. **Handle unavailable source explicitly.** When a function's source is not in the repository (third-party library), mark it UNVERIFIED in trace tables. Search for type signatures, documentation, or test usage as secondary evidence. Do not guess behavior from the function name.

### General
7. Do not treat style preferences as findings unless they affect maintainability or correctness.
8. Do not hide uncertainty — state what is unverified.
9. Do not skip the refutation check. It is mandatory in every mode.
10. **Do not fabricate to fill template sections.** If you cannot verify a claim, write "NOT VERIFIED" or "N/A" rather than inventing plausible-sounding content. An incomplete but honest certificate is more valuable than a complete but fabricated one.

---

## Minimal Response Contract

Every response using this skill must include:

| Element | Required in |
|---------|-------------|
| Selected mode | All |
| Numbered premises | All |
| Interprocedural trace table | All (when functions are on the code path) |
| Per-item analysis (per-test, per-method, or per-function) | compare, diagnose, explain |
| Refutation / alternative-hypothesis check | All |
| Formal conclusion with premise/claim references | All |
| Confidence level | All |

---
> Source: [KunihiroS/agentic-code-reasoning-skills](https://github.com/KunihiroS/agentic-code-reasoning-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
