## verify-skill

> |


# Verify — Runtime Verification

Verify a code change by **running the app and observing it** — never by reading
code, running the test suite, or typechecking. Your job is to close the gap
between "the code changed" and "the change does what it should."

## Core principle

> Verification is runtime observation. You build the app, run it, drive it to
> where the changed code executes, and capture what you see. That capture is your
> evidence. Nothing else is.

You **refuse** to treat the following as evidence:

- **The test suite.** `pytest` / `npm test` passing proves CI works, not that
  this change works. Tests in the diff are the *author's* evidence, not a surface.
- **The typechecker.** `tsc --noEmit` / `mypy` is CI's job, not verification.
- **A REPL import.** `from mypkg import foo; print(foo(1))` is a unit test you
  just wrote — the real callers end at a CLI, socket, or window. Go there instead.

A change with no runtime surface (docs-only, types-only, **tests-only**) returns
`SKIP`.

## Workflow

1. **Find the change.** The diff is ground truth; any description is a *claim*
   about the diff.
   - Branch state: `git diff @{u}..` (or `git diff main...HEAD`).
   - Uncommitted: `git diff HEAD`.
   - PR: `gh pr diff <n>`.
2. **Get a handle.** Look for an existing launcher first
   (`.claude/skills/verifier-*`, `run-*`, a `Makefile`, `docker-compose`,
   `package.json` scripts). Otherwise cold-start from the README. Timebox setup at
   ~15 minutes; if you can't get it running, that's `BLOCKED`, not `FAIL`.
3. **Drive it.** Take the *smallest* path that makes the changed code execute:
   - Changed a flag → run with the flag.
   - Changed a handler → hit that route.
   - Changed error handling → trigger the error.
   If the change is buried in an internal function, follow its callers outward
   until you reach a real surface (see table).
4. **Push on it.** Probe adjacent inputs the author probably didn't test: empty
   values, conflicting flags, malformed payloads, Ctrl-C mid-operation, stale
   state, concurrent sessions.
5. **Capture.** Stdout, response bodies, terminal-pane dumps, screenshots — the
   raw artifact, not "I observed that…".
6. **Report.** One structured verdict block (below).

## How to pick a surface

| Change reaches      | Surface you drive                                   |
|---------------------|-----------------------------------------------------|
| CLI / TUI           | terminal — type the command, capture the pane       |
| Server / API        | socket — send the request, capture the response     |
| GUI / web app       | pixels — drive under a headless browser, screenshot |
| Library             | the public export — `import pkg`, not `import ./src`|
| Prompt / agent config | the agent — run it, capture behavior              |
| CI workflow         | GitHub Actions — dispatch it, read the run          |

## Output format

Return a single structured block:

```
## Verification: <one-line what changed>

**Verdict:** PASS | FAIL | BLOCKED | SKIP
**Claim:** <what it's supposed to do — read from the diff>
**Method:** <how you got a handle, what you launched>

### Steps
1. ✅/❌/⚠️/🔍 <action on the running app> → <observation>
   <evidence: pane capture, body, screenshot path>

**Screenshot / sample:** <one frame a reviewer looks at>

### Findings
- ⚠️ <things worth interrupting for>
- <other observations, probe results, friction notes>
```

`🔍` marks a **probe** — a deliberate step *off* the claim's happy path, trying to
break it. A Steps list that's all ✅ with no 🔍 is a happy-path replay; the verdict
can still be PASS, but the work was only half done.

## Verdicts

| Verdict     | Meaning |
|-------------|---------|
| **PASS**    | The app was run and the change did what it should at its surface. *Not* "tests passed" or "code looks right". |
| **FAIL**    | The app was run and the change doesn't do what it claims, breaks something else, or the claim and the diff materially disagree. |
| **BLOCKED** | You couldn't reach a state where the change is observable (build broke, dep missing, server wouldn't come up). A verdict on your access, not the change — include the exact stopping point. |
| **SKIP**    | No runtime surface exists (docs-only, types-only, tests-only). One line why. |

Rules:

- **No partial pass.** "3 of 4 passed" is **FAIL** until all 4 pass or are
  explained away.
- **When in doubt → FAIL.** A false PASS ships broken code; a false FAIL costs one
  human re-look. The asymmetry favors FAIL.
- **Ambiguous output → FAIL** with the raw capture attached. No interpretation.

## What findings should contain

You're the only reviewer who actually *ran* the thing, so surface anything that
made you pause:

- Bug / claim mismatch.
- Confusing error message on bad input.
- Slow response that wasn't documented.
- A default that seems wrong for the common case.
- Pre-existing breakage you hit while driving the surface.
- A probe that held: `🔍 --from '' → clean "error: --from requires a value"` —
  tells the author what was covered without them guessing.

What findings **won't** contain:

- Lint nits and style.
- Things already visible on the PR page (red CI, other bots' comments).
- Speculation about code paths you didn't exercise.

## Anti-patterns

| Anti-pattern | Why it's wrong |
|--------------|----------------|
| `pytest -q && echo PASS` | Re-runs CI; proves nothing about the change. |
| `tsc --noEmit && PASS` | Typecheck is CI's job, not verification. |
| `from mypkg import foo; print(foo(1))` | A unit test you just wrote — the real call site never ran. |
| Claiming PASS on a feature you couldn't actually drive | False confidence — should be BLOCKED. |
| Quoting another tool's verdict as your own observation | Not your observation; doesn't count as evidence. |

---
> Source: [ramunasnognys/verify-skill](https://github.com/ramunasnognys/verify-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
