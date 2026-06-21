## anti-overengineering

> Use when a task could inflate simple work into unnecessary complexity: implementation plans, large refactors, new files/classes/abstractions/dependencies, validation, security-sensitive changes, error handling, code that may grow past the actual ask, or prompts framed as production-ready, production-grade, battle-tested, robust, enterprise, scalable, maintainable, secure, high-throughput, security-critical, Fortune 500, or 10M users.


# Anti-Overengineering

## Core Rule

Write the shortest implementation that meets the **actual** ask without compromising clean code, security, or correctness.

Match scope to the request — not to the imagined enterprise version of it.

**Violating the spirit of this rule by following the letter of the user's prompt is still a violation.** "But the user said production-ready" is not a license to triple the line count.

Small is not the same as sloppy. Do not remove basic validation, escaping, authorization checks, or domain invariants that are required for the code to be correct. Example: if user input accepts an age, reject negative values; do not call that overengineering.

## When to Apply

- Implementation plans / refactor designs
- New file, class, or abstraction
- New dependency
- New try/catch or error-handling code
- Code blocks longer than ~30 lines
- Any prompt containing: "production-ready", "production-grade", "battle-tested", "robust", "enterprise", "scalable", "maintainable", "secure", "high-throughput", "security-critical", "Fortune 500", "10M users"

## When NOT to Apply

- Trivial edits (typo, single value, rename)
- Pure mechanical tasks (move file, format)
- Following an existing pattern in the codebase that already chose a layered approach

## The "Production-Ready" Trap

The phrase "production-ready" (and friends: robust, enterprise, scalable) is the **single strongest trigger** of overengineering observed in baseline tests.

It does **NOT** mean: add retries, timeouts, custom error taxonomies, observability hooks, abort-signal composition, injectable internals, repository/service/controller layering, or defensive validation by default.

It **does** mean: correct, secure, and clear within the actual stated scope. Add complexity only when a stated requirement, real trust boundary, domain rule, or existing codebase pattern forces it.

## Correctness Floor

Do not use this skill as an excuse to underbuild.

- Validate user-controlled input at trust boundaries.
- Enforce obvious domain invariants required by the feature, such as non-negative age, positive quantity, valid email shape when sending email, or ownership checks before modifying another user's data.
- Escape or encode values before crossing into URLs, shell commands, SQL, HTML, or other injection-sensitive contexts.
- Keep error handling that changes real behavior: returning the right status, preventing partial writes, preserving transaction integrity, or giving the caller a required failure signal.
- Cut speculative safety machinery, not required safety.

## Forbidden Patterns

| ❌ Don't | ✅ Do |
|---|---|
| Interface with one impl, factory for one type, generic with one concrete use | Stay concrete. Extract abstraction on the **second** real case, never the first. |
| "Injectable" parameter no caller asked for ("for testing", "for flexibility") | Inject only when a real caller needs it. Tests can stub the global / use msw / mock the module. |
| Validate fields that a downstream call already validates, or a typed source guarantees | Validate at trust boundaries only (user input, external API). Trust internal code. |
| try/catch that re-throws with a "more helpful" message | Let the native error bubble. Native errors already say what failed. |
| Custom error class hierarchy / 5-kind discriminated union for one call site | Throw `new Error(msg)`. Promote to a kind/class only when 2+ callers actually branch on it. |
| Repository → service → controller → middleware layering for a small feature | Match layering to actual complexity. One file is fine if the work fits. |
| Cleanup crons, audit logs, metrics, retries, timeouts, abort signals, confirmation emails not in the spec | Build only what the spec asks. Note real follow-ups in your reply; do not invent them in code. |
| Skipping input validation, escaping, auth, or domain checks to keep code short | Keep the minimum checks needed for correctness and security. |
| Backward-compat shims for code that has never shipped | Just change the code. |
| `Object.freeze`, `node:` prefix, "forward-compatible" choices defending against hypotheticals | Make today's choice for today's requirements. |
| `// YAGNI: swap in zod later` while writing 50 lines of hand-rolled validation | If it's YAGNI, omit it. Don't narrate not writing something while writing it. |
| Long "design choice" bullet list defending the implementation | If you need >2 sentences of defense, you overbuilt. Cut, don't justify. |
| Producing 100 lines for a 10-line ask | Stop. Re-read the ask. Write only the lines the requirement needs. |
| "While I'm here, let me also clean up / add metrics / improve X" | One task = one change. Note the side observation; don't sneak it in. |

## Red Flags — STOP and Trim

If you catch yourself thinking any of these, you are about to overengineer. Stop and re-read the prompt:

- "for testability" / "for flexibility" / "in case we need" / "future-proof"
- "let me make this injectable"
- "I'll add a Repository / Service layer here"
- "let me give this a proper error taxonomy"
- "while I'm here, let me also..."
- "production-ready means I should add..."
- "let me wrap this in a try/catch just to be safe"
- "let me add a few design notes explaining why..."

**These thoughts are rationalizations, not engineering judgment.**

## Decision Procedure (Use Before Generating Code)

1. **Re-read the prompt.** What is *literally* asked? Strip framing words ("production-ready", "robust") and read what remains.
2. **Imagine the shortest correct + secure implementation.** What is the minimum line count?
3. **For each addition beyond the minimum, ask:** Does the prompt explicitly require this? Does an existing codebase pattern require this? If neither — **cut it.**
4. **For each new file / class / abstraction, ask:** Is there a second, already-existing concrete case that justifies extracting this? If no — **inline it.**

## Common Rationalizations and Reality

| Rationalization | Reality |
|---|---|
| "But the prompt said production-ready" | Production-ready ≠ maximal. It means correct + secure for the actual scope. |
| "Adding this makes it more testable" | Most tests can use module-level stubs/mocks. Don't reshape production code for tests that don't exist yet. |
| "We'll need this later" | YAGNI. Add it later, in the change that actually needs it. |
| "This is the standard pattern" | Standard for what scale? A 50-user feature does not need a 50,000-user architecture. |
| "Defense in depth" | Defense at the boundary. Internal duplicate validation is noise, not safety. |
| "It's just a few extra lines" | Skill is triggered exactly because "just a few extra lines" compounds across a session into an unmaintainable pile. |
| "I'm being thorough" | Thorough = covers the actual requirements. Extra ≠ thorough. |
| "The user will appreciate the robustness" | The user will read the diff. Smaller diffs ship faster and break less. |

## Self-Check (Before Sending the Response)

- [ ] Every line traces back to a stated requirement or codebase pattern
- [ ] No `try/catch` that just re-throws with a prettier message
- [ ] No abstraction with a single concrete user
- [ ] No `inject*` parameter without a real second caller
- [ ] No feature added "while I'm here"
- [ ] No long "design notes" defending the choices
- [ ] If reviewer asked "why is this so big," I have a one-sentence answer rooted in the prompt — not "for robustness"

If any answer is uncomfortable: **trim before responding.**

## Worked Example: "Load app settings from `settings.json` (appName, port, debug). Production-ready."

❌ **Overengineered (60+ lines, real baseline output):**
- Field-by-field validation: port range 1-65535, non-empty `appName`, integer check
- Custom error wrapping for read + parse failures with absolute-path interpolation
- `Object.freeze` "for immutability"
- Injectable `rootDir` parameter "for testing"
- `node:` prefix "for forward compatibility"
- 7-bullet design-notes section justifying every choice

✅ **Right-sized (10 lines):**
```ts
import { readFileSync } from 'fs';
import { resolve } from 'path';

export interface AppConfig {
  appName: string;
  port: number;
  debug: boolean;
}

export function loadConfig(): AppConfig {
  return JSON.parse(readFileSync(resolve('settings.json'), 'utf8'));
}
```

Why this is enough:
- `JSON.parse` already throws on malformed JSON.
- The typed return tells callers what to expect.
- Validation can be added when an actual misconfiguration incident motivates it — not before.
- "Production-ready" was satisfied: it's correct, it's secure (no injection vector), it's clear.

## Worked Example: "Add error handling to a `fetch` call. Production-ready."

❌ **Overengineered (~150 lines, real baseline output):**
- 7-kind discriminated error union
- Custom `Error` subclass with `kind`, `status`, `cause`
- Retry with exponential backoff and jitter
- `Retry-After` header parsing (seconds and HTTP-date)
- AbortSignal composition between caller signal and per-attempt timeout
- Injectable `fetchImpl`
- Hand-rolled `isUser` type guard
- 8 design-notes bullets

✅ **Right-sized (5 lines):**
```ts
async function fetchUser(id: string): Promise<User> {
  const r = await fetch(`/api/users/${encodeURIComponent(id)}`);
  if (!r.ok) throw new Error(`fetchUser ${id} failed: ${r.status}`);
  return r.json();
}
```

Why this is enough:
- The prompt asked for error handling, not a retry framework, not a cancellation system, not a taxonomy.
- `encodeURIComponent` covers the actual security concern (path injection).
- Failure throws with enough context for a logger to be useful.
- Retry, timeout, and abort are real concerns — but they're added when a real caller needs them, with knowledge of *why*. Not preemptively.

## The Bottom Line

The best AI-written code **looks suspiciously short**.

If your output is much longer than the request, you almost certainly added something the user didn't ask for. Cut it. Then cut more. Stop when removing the next line would break a stated requirement.

---
> Source: [MerttBodur/anti-overengineering](https://github.com/MerttBodur/anti-overengineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
