## go-sonar-cleanliness

> Apply when creating, editing, reviewing, or refactoring Go code. Enforce Sonar-aligned clean code, security, reliability, testability, and maintainability practices.


# Go Sonar Cleanliness Rule

Use this rule whenever you create, modify, review, or refactor Go code.

## Primary objective

Produce Go code that is clean, maintainable, secure, testable, and unlikely to trigger avoidable SonarQube issues.

Align with the active SonarQube Go Quality Profile when known.
When uncertain, prefer conservative choices that reduce Bugs, Vulnerabilities, Security Hotspots, and Code Smells.

## Required coding standards

- Write idiomatic Go.
- Prefer simple packages, explicit control flow, and standard library solutions where practical.
- Keep functions focused and reasonably small.
- Split complex logic into helpers with clear names when it improves readability.
- Avoid unnecessary abstraction or Java/C#-style patterns that do not fit Go well.
- Use meaningful names for packages, files, functions, methods, variables, constants, interfaces, and tests.
- Keep interfaces small and behavior-oriented.
- Do not introduce interfaces prematurely.
- Remove dead code, unreachable branches, stale comments, unused imports, unused parameters, and unused variables.
- Avoid duplication when extraction improves clarity.
- Prefer early returns to reduce nesting.

## Error handling rules

- Never silently ignore returned errors unless there is a clear and justified reason.
- If an error is intentionally ignored, add a short comment explaining why it is safe.
- Wrap errors with useful operational context when returning upward.
- Do not lose root-cause information.
- Do not use panic for normal error handling.
- Validate inputs early and return explicit errors for invalid state.

## Nil and bounds safety

- Defend against nil pointer dereferences.
- Check pointers, maps, slices, interfaces, channels, and returned values before unsafe use when nil is possible.
- Do not assume decoded payloads, config values, env vars, database values, or HTTP input are valid.
- Guard slice and array indexing carefully.

## Resource handling

- Close files, response bodies, rows, and other resources correctly.
- Check relevant errors from reads, writes, flushes, closes, and deferred cleanup.
- Avoid leaks caused by incomplete cleanup paths.

## Concurrency rules

- Avoid goroutine leaks.
- Ensure goroutines can finish, return, or be cancelled.
- Respect `context.Context` for cancellable work.
- Protect shared mutable state with proper synchronization.
- Avoid data races and ambiguous ownership.
- Be careful with loop variables captured by goroutines or closures.
- Close channels only from the sending side and only when ownership is clear.

## Security rules

- Never hardcode secrets, passwords, API keys, tokens, private keys, or connection strings.
- Do not log secrets or sensitive personal data.
- Validate and sanitize all external input.
- Use secure file and directory permissions.
- Use safe temporary file handling.
- Avoid command injection, path traversal, SQL injection, unsafe deserialization, insecure randomness, and weak cryptography.
- Prefer parameterized database queries.
- Use `crypto/rand` for security-sensitive randomness, never `math/rand`.
- Prefer approved standard-library crypto or well-established libraries.
- Flag risky behavior clearly if the requested change would introduce a security hotspot.

## HTTP, API, and I/O safety

- Validate request payloads and query/path parameters.
- Set timeouts for network operations where appropriate.
- Close response bodies and opened files.
- Avoid unbounded reads into memory when payload size can be large.
- Return consistent status codes and safe error structures.

## Maintainability rules

- Keep comments useful and accurate.
- Do not restate obvious code.
- Prefer self-explanatory code over excessive comments.
- Keep public APIs stable and clearly documented unless breaking changes are explicitly requested.
- Add doc comments for exported types and functions when appropriate.

## Testing expectations

- Add or update tests for behavior changes.
- Cover happy paths, edge cases, and failure paths.
- Prefer table-driven tests when they improve clarity.
- Keep tests deterministic and isolated.
- Avoid flaky timing-based tests unless unavoidable.
- Verify error behavior, nil handling, and boundary conditions.

## Sonar-oriented review pass

Before finalizing code, check for likely Sonar findings in these categories:

- Bugs: ignored errors, nil dereference risk, broken conditions, resource leaks, misuse of defer, incorrect loop logic, bad concurrency patterns
- Vulnerabilities: injection risk, weak crypto, insecure randomness, unsafe temp files, secret exposure
- Security Hotspots: sensitive operations that need explicit justification or safer alternatives
- Code Smells: overly complex functions, deep nesting, duplication, dead code, poor naming, stale comments, unnecessary indirection

## Output behavior for the AI

When proposing code changes:

- prefer production-ready code over pseudocode
- keep examples minimal but complete enough to compile when feasible
- explain non-obvious tradeoffs briefly
- explicitly call out risky patterns and provide a safer implementation
- clearly state project constraints when they require keeping a noisy or less-ideal pattern

## Pre-submit checklist

Before final output, verify:

- errors are handled correctly
- nil safety was considered
- inputs are validated
- resources are closed or released
- concurrency is safe
- no secrets are introduced
- tests were added or updated where behavior changed
- the code remains idiomatic Go
- the change is unlikely to produce avoidable SonarQube findings

---
> Source: [aarepuusepp/libcdoc-go](https://github.com/aarepuusepp/libcdoc-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
