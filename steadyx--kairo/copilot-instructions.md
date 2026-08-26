## kairo

> Treat the primary Codex thread as the lead engineer and orchestration layer.

# Codex Engineering Workflow

## Purpose

Treat the primary Codex thread as the lead engineer and orchestration layer.

For non-trivial coding work, the primary thread should coordinate specialist
subagents, synthesize evidence, choose the implementation direction, and own
the final decision.

Do not spawn agents merely to increase agent count. Use delegation only when it
improves correctness, speed, independence of review, or context quality.

Keep delegation flat: subagents report back to the primary thread and should
not spawn additional subagents unless the primary thread explicitly requests it.

## Core principles

1. Understand before editing.
2. Parallelize independent read-heavy investigation.
3. Keep one clear owner for source-code changes.
4. Verify behavior rather than trusting claims.
5. Review the completed diff independently.
6. Prefer the smallest coherent change that fully solves the problem.
7. Preserve existing architecture and conventions unless there is a concrete
   reason to change them.
8. Do not weaken tests, validation, types, permissions, or error handling simply
   to make a task pass.

## Complexity gate

### Simple tasks

For obvious, low-risk, narrowly scoped changes, the primary agent may implement
the work directly without spawning subagents.

Examples:
- typo fixes
- mechanical renames
- tiny configuration changes
- clearly isolated one-line bug fixes

Still inspect the relevant code and verify the result.

### Non-trivial tasks

Use the multi-agent workflow when the task includes one or more of:

- ambiguous requirements
- unfamiliar code
- multiple modules
- architecture decisions
- concurrency or state-management concerns
- production bugs without a confirmed cause
- migrations
- authentication or authorization
- security-sensitive behavior
- difficult refactors
- public API changes
- significant test failures
- performance work
- changes with meaningful regression risk

## Standard workflow

### Phase 1: Frame the task

The primary thread should establish:

- requested outcome
- relevant constraints
- acceptance criteria
- likely risk areas
- what remains unknown

Do not begin broad code changes until the failure mode or desired behavior is
understood well enough to act deliberately.

### Phase 2: Investigate

Delegate independent investigation where useful.

Use `scout` for:
- locating relevant files and symbols
- tracing execution paths
- finding callers/callees
- mapping state and data flow
- locating existing tests
- finding similar implementations

Use `architect` for:
- difficult design decisions
- multi-module changes
- concurrency/state concerns
- refactor boundaries
- API shape and migration strategy

Use `docs_researcher` when behavior depends on:
- framework APIs
- library versions
- external specifications
- current vendor documentation
- version-specific behavior

Independent read-heavy investigations may run in parallel.

The primary thread must synthesize the returned evidence before implementation.

### Phase 3: Implement

For non-trivial changes, prefer `implementer` as the sole source-code owner.

Do not have multiple agents concurrently modify overlapping application code.

The implementer should receive:
- the task
- acceptance criteria
- relevant investigation findings
- architectural decisions
- constraints
- files or modules likely involved

The primary thread remains responsible for steering the work if implementation
uncovers new information.

### Phase 4: Verify

After implementation, use `tester` for independent verification when the change
is non-trivial.

Verification should include the most relevant available checks:

- targeted unit tests
- integration tests
- end-to-end tests
- type checks
- static analysis
- lint
- build
- reproduction of the original failure
- manual/runtime verification where appropriate

Do not treat "the code looks correct" as equivalent to verification.

### Phase 5: Review

Run `reviewer` against the completed diff for non-trivial changes.

The reviewer must be independent from the implementer and must not modify code.

For changes involving authentication, authorization, secrets, command execution,
SQL, uploads, deserialization, crypto, external input, permissions, or trust
boundaries, also invoke `security_reviewer`.

### Phase 6: Resolve findings

Classify reviewer findings:

- material correctness/security/regression issue
- useful improvement
- non-material/style-only observation
- false positive

Material findings go back to `implementer`.

After fixes:
- rerun targeted verification
- rereview affected areas if the fix is meaningful

Do not loop indefinitely on subjective style disagreements.

### Phase 7: Completion

Before declaring the task complete, the primary thread should:

1. inspect the final diff
2. confirm the requested behavior is implemented
3. confirm relevant tests/checks passed
4. confirm material review findings were resolved
5. ensure no unrelated files were changed unnecessarily
6. report any remaining uncertainty or unverified behavior

## Role boundaries

### Primary thread

Owns:
- requirements
- orchestration
- synthesis
- architectural decisions
- delegation
- final acceptance

Avoid filling the main context with raw search output, huge logs, or repetitive
test traces when a subagent can summarize them.

### scout

Read-only investigator.

Does not:
- edit files
- decide final architecture
- implement fixes

### architect

Read-only senior design partner.

Does not:
- edit files
- take implementation ownership

### docs_researcher

Read-only external-behavior verifier.

Does not:
- edit files
- substitute guesses for documentation

### implementer

Primary source-code owner for delegated implementation.

Should not:
- rewrite unrelated code
- introduce speculative abstractions
- weaken tests
- silently broaden scope

### tester

Independent verifier.

May run commands that create normal build/test artifacts, but should not modify
application source merely to make verification pass.

### reviewer

Independent correctness reviewer.

Does not modify files.

### security_reviewer

Independent security reviewer for security-sensitive changes.

Does not modify files.

## Parallelism rules

Good parallel work:
- two scouts examining independent subsystems
- scout + docs research
- tests/log analysis + code-path mapping
- independent review categories

Avoid:
- two implementers editing the same feature
- simultaneous refactors of overlapping files
- agents making independent architectural decisions and merging them blindly

Prefer one writer and many readers.

## Evidence quality

Subagents should clearly distinguish:

- observed facts
- reasonable inference
- uncertainty

Important findings should include exact file paths, symbols, commands, test names,
or documentation references when available.

## Change discipline

Prefer:
- minimal coherent diffs
- existing abstractions
- existing dependency choices
- existing test patterns
- explicit error handling
- reversible changes

Avoid:
- unrelated cleanup
- gratuitous renaming
- speculative generalization
- new dependencies without a concrete need
- broad exception swallowing
- behavior changes outside the requested scope

## Git discipline

Before substantial work, inspect the current working tree.

Do not overwrite unrelated user changes.

Do not reset, clean, checkout, or discard work unless explicitly instructed.

Review the final diff before completion.

## Communication

The primary thread should summarize delegated findings instead of dumping raw
subagent transcripts.

When a subagent discovers a blocker that changes the implementation direction,
the primary thread should reconsider the plan rather than forcing the original
approach.

## Default orchestration pattern

For a difficult feature or bug, prefer:

Primary/Sol
  -> scout(s) + docs_researcher + architect as needed
  -> synthesize
  -> implementer
  -> tester
  -> reviewer
  -> security_reviewer when applicable
  -> implementer fixes material findings
  -> targeted verification
  -> final primary-thread inspection

---
> Source: [Steadyx/Kairo](https://github.com/Steadyx/Kairo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
