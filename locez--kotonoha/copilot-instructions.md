## kotonoha

> This file applies to the repository root and all subdirectories.

# AGENTS.md

## Scope

This file applies to the repository root and all subdirectories.

- This file contains only long-lived, repeatable engineering rules.
- Do not store issue lists, milestones, temporary migrations, or current technical-debt inventories here.
- Specific work is tracked in issues, pull requests, and design documents.
- A nested `AGENTS.md` may add stricter local rules, but must not weaken this file.
- System, developer, and user instructions take precedence over this file.

## General Principles

- Understand the existing code, tests, build configuration, and runtime before changing them.
- Prefer small, verifiable, reversible changes.
- Never overwrite, revert, or clean up changes that belong to the user.
- Avoid unrelated refactors, formatting changes, and dependency upgrades.
- Do not turn a temporary workaround into a permanent design.
- Add an abstraction only when it reduces coupling, clarifies ownership, or improves testability.
- Update tests and documentation when behavior or public contracts change.

## Change Workflow

For every task:

1. Confirm the scope, constraints, and acceptance criteria.
2. Check the current branch and working-tree status.
3. Read the relevant implementation, tests, and configuration.
4. Define the behavior boundary before implementing it.
5. Decide what evidence is needed for the change. Add or update tests only when
   they verify a meaningful behavior, contract, invariant, or failure path.
   Do not add tests merely because a file, setting, or code path changed.
6. Keep the change focused and complete.
7. Run the checks relevant to the change.
8. Review the final diff for unrelated files and temporary artifacts.
9. Record the change, tests, risks, and remaining work in the commit or pull request.

Do not develop directly on the default branch.

## Project Structure

- `src/kotonoha/`: Python package and CLI entry point (`kotonoha.main:entry_point`).
- `src/kotonoha/platform/`: compositor and toolkit decisions — capability probes, the
  native bridge wrapper, and the overlay platform adapters.
- `src/kotonoha/lyrics/`: providers, parsers, matching, and the lyrics cache.
- `src/kotonoha/providers/`: player integrations (MPRIS) and the source gate.
- `src/kotonoha/layer_shell_bridge.cpp`: the C++ Wayland bridge, built to `libkoto-layer.so`.
- `tests/`: Python tests for the root package.
- `plugins/cider/lyrics/`: Cider lyrics probe plugin built with Vite and TypeScript.
- `plugins/cider/lyrics/src/probe/`: Cider playback, TTML parsing, payload, and plugin state logic.
- `plugins/cider/lyrics/src/__tests__/`: Vitest tests for the Cider probe.

Keep generated output out of version control. Python caches, `node_modules/`, Cider
`dist/`, and npm/yarn lockfiles under `plugins/cider/` are ignored.

## Architecture

Business logic, external systems, and presentation code must have clear boundaries.

- Presentation code handles display, input, and state binding.
- Application code coordinates use cases and workflow lifecycles.
- Domain code owns business rules, value objects, state, and stable contracts.
- Infrastructure code adapts networks, files, databases, platform APIs, and third-party libraries.
- Business logic must not depend directly on a concrete UI, network client, or platform implementation.
- Compositor and toolkit facts belong to the platform layer. Presentation code asks
  a capability, and the adapter answers; it must not read the native bridge, a
  desktop-environment name, or a Qt platform name to decide behavior itself.
- A capability that is unavailable must carry a reason the UI can show.
- An operation must report what actually happened. Do not return success because a
  call did not raise when the underlying platform may have ignored it.
- External data must be parsed, validated, and normalized at the boundary.
- Raw third-party objects must not propagate through business code without an explicit contract.
- Prefer interfaces, protocols, or capability models over concrete implementations.
- Do not inherit from a `Protocol` to implement it. Inherited empty bodies turn a
  missing method into a silent no-op; structural typing already checks conformance.
- Avoid circular dependencies, implicit global state, and owner objects that absorb unrelated responsibilities.
- Transitional compatibility code must have a clearly limited scope and a documented removal condition.

## Strong Typing

Strong typing is a continuous requirement for all new and modified code.

- Public functions, methods, attributes, and cross-module interfaces must have complete annotations.
- Do not introduce unexplained `Any` into application or domain contracts.
- Do not use unbounded dictionaries to represent business state.
- Parse external JSON, configuration, and third-party objects into typed structures.
- Use `dataclass`, `Enum`, `Protocol`, `TypedDict`, and type aliases when they express the real contract.
- Give states, commands, events, errors, and configuration explicit structures.
- Before adding a helper or utility function, search the existing code for an equivalent capability. Reuse the existing function or extend its typed contract when it owns the same responsibility; add a new helper only when the responsibility or ownership is genuinely different.
- When a value does not satisfy a function's annotated input type, first decide whether the function's contract should accept that value. Prefer widening or clarifying the function's type and testing the resulting contract when the value is semantically valid; do not add call-site conversions merely to satisfy the annotation. Convert at a real boundary only when normalization is part of that boundary's explicit contract, with validation that preserves the intended meaning.
- Use the type checker configured by the repository; do not hide errors by expanding exclusions.
- Every type suppression must document its reason, impact, and follow-up path.

## Explicit Contracts and Dynamic Access

- Initialize every instance attribute in `__init__` or an explicit factory, with a declared type. Optional state must be initialized to `None`; callers should use direct attribute access and explicit `is None` checks.
- Do not use reflection-style access (`getattr`, `hasattr`, `setattr`, `delattr`, `__dict__`, `globals()`, or `locals()`) to model application or domain state, discover fields, or hide incomplete initialization. Do not add trivial getter/setter methods or generic `get(name)` APIs for ordinary fields; use typed attributes or properties. Use a method when it performs computation, validation, a meaningful side effect, or protects an invariant.
- Dynamic access is allowed only at an external compatibility boundary, such as a Qt/plugin/third-party version probe or an optional dependency. Isolate it in a typed adapter or small helper, document why it is unavoidable, validate the result, and test both supported and unavailable cases. Do not scatter capability checks through business or presentation logic.
- Do not use `value or default` or missing-key fallbacks when `False`, `0`, an empty value, or an explicitly missing value have different meanings. Use explicit `None` checks and validation.
- Do not use `cast`, `# type: ignore`, or `assert` to conceal a missing contract. Narrow types through explicit checks or adapters; use `assert` only for an internal invariant that cannot be supplied by external input and whose failure is a programming error.
- Do not use `eval`, `exec`, string-based dispatch, or dynamic imports for ordinary control flow. Prefer explicit maps, protocols, registries, or adapters. Isolate and review any plugin boundary that genuinely requires dynamic loading.
- Dependencies and ownership must be explicit constructor or function inputs. Do not locate services through global state, service factories deep inside a component, widget parent traversal, or reflective lookup; wire them at the composition root.
- One argument must not carry two opposite meanings across a boundary. When `None`
  means "everything" on one side and "nothing" on the other, split the operation
  instead of documenting the ambiguity.

## Documentation and Comments

- Public modules, classes, protocols, functions, and methods must have concise docstrings that explain their responsibility and relevant input, output, side effect, ownership, or failure contract.
- Key class fields must have a nearby comment or class-level field documentation when their role, default, sensitivity, lifecycle, or ownership is not obvious from the type alone.
- Document configuration fields, injected services, sessions, tasks, credentials, and other state that crosses a layer or has a non-trivial lifecycle.
- Private helpers need documentation when they enforce an invariant, normalize external data, handle security-sensitive values, or coordinate cancellation and resource cleanup.
- Comments should explain intent and constraints rather than repeat the code. Keep them short and update them with the behavior they describe.
- Record protocol facts that cannot be rediscovered from the code: which Wayland
  request a call maps to, what a compositor is permitted to ignore, and why a
  workaround exists.
- Treat existing documentation as part of the code's contract and accumulated domain knowledge. Before rewriting it, preserve non-obvious protocol mappings, return-code meanings, invariants, lifecycle or security constraints, and operational guidance.
- Do not remove meaningful documentation merely for brevity or style consistency. If information is obsolete, verify that it is no longer true and record the replacement or removal rationale in the change description.
- Transitional compatibility code must include a `TODO` comment stating what will be removed and the condition or tracking issue that permits removal.
- Do not add a compatibility fallback without an explicit removal condition; when the condition changes, update the `TODO` with the new owner or follow-up issue.

## Python Quality

- Use explicit resource ownership and context management.
- Avoid mutable default arguments, implicit global state, and duplicated business logic.
- Constructors may build in-memory/UI state, but must not perform network I/O, spawn background tasks or subprocesses, or register process-global hooks. Expose explicit start/initialize and stop/close/shutdown operations for resources and workflows.
- Catch only exceptions that can be handled; avoid blanket `except Exception` blocks.
- Never use `except Exception: pass`, convert an unknown failure into success, or log an error while leaving ownership/state ambiguous. Catch expected failures at the boundary, preserve useful context, and make cleanup failures observable.
- Do not use blocking I/O or `time.sleep` on an event-loop/UI thread. Use an async API or an explicitly owned worker, with a timeout and a cleanup path.
- Use the repository's logging mechanism instead of `print()` for production diagnostics.
- Represent failures with explicit error types or result values.
- Keep functions focused and avoid unnecessary complexity.
- Prefer the standard library and existing project tools before adding dependencies.
- Do not modify third-party vendored code unless the task explicitly requires it.

## Coding Style

Python is async-first: design receivers, player bridges, polling, and network I/O as
`async` services from the start; keep only the Qt widget boundary synchronous. Use
qasync to integrate the event loop, keep GUI work on the UI thread, and make
background tasks cancellable. Follow Ruff's configured line length, use
`pathlib.Path` for paths, and `dataclass` or small typed objects for structured data.
Use snake_case for modules, functions, and variables; PascalCase for classes.

TypeScript uses ES modules, strict compiler settings, and camelCase identifiers. Keep
probe logic in `plugins/cider/lyrics/src/probe/`, and use `.test.ts` for Vitest files.

## File Size

- New or modified Python source files should normally stay within 500 lines.
- Files over 800 lines should be split by responsibility; document the reason in the PR when that is not practical.
- Splitting must follow responsibility and dependency boundaries; do not split mechanically just to reduce line count.
- Generated and third-party code are exempt.

## Async and Lifecycle

- Every asynchronous task must have a clear creator and owner.
- Keep task handles and cancel, await, and inspect them at the appropriate lifecycle boundary.
- Do not create unowned fire-and-forget tasks.
- Treat tasks created by callbacks, Qt signals, timers, `qasync`, `asyncio.create_task`, or `ensure_future` the same way: retain the handle under the owning component or supervisor and provide a cancellation/await path.
- Connect a Qt signal to a bound method rather than a lambda that captures the
  receiver. PyQt holds a bound method's receiver weakly, so the connection dies
  with the widget; a lambda is held strongly and keeps firing into a deleted C++
  object.
- A deferred callback must check that its target still exists before touching it.
- Treat `run_in_executor`, `asyncio.to_thread`, and subprocess work as owned resources too. Define timeout, cancellation, and shutdown behavior; cancellation of the wrapper must not be mistaken for cancellation of the underlying blocking operation.
- Handle `asyncio.CancelledError` as control flow: release owned resources and re-raise unless the caller explicitly owns cancellation recovery. Use `BaseException` catches only for narrowly scoped cleanup that re-raises or records the failure.
- Start, stop, retry, and shutdown operations should be predictable and as idempotent as practical.
- Network sessions, files, servers, threads, and subprocesses must have explicit cleanup paths.
- Compositor-side objects (layer surfaces, blur effects, input regions) are owned
  resources too: release them before destroying the surface they are keyed on.
- Do not rely on object destruction or process exit to release critical resources.
- Do not use nested event loops such as `exec()` to make an asynchronous workflow appear synchronous. Keep modal behavior at the presentation boundary; prefer non-blocking `open()`/`show()` plus an explicit completion or shutdown contract. If a platform API requires a nested loop, document its ownership and cancellation limitations.
- Test cancellation, timeouts, repeated calls, and exceptional shutdown paths.

## Security

- Never expose passwords, tokens, sessions, private keys, or other secrets in logs, tests, issues, pull requests, or commits.
- Do not store sensitive data in ordinary configuration files, temporary files, or build artifacts.
- Do not bypass authentication, TLS, validation, or permission checks for convenience.
- Validate every external input.
- Treat external URLs, file paths, redirects, and subprocess arguments as security boundaries.
- Pass subprocess arguments as an explicit argument list; do not use `os.system`, shell string concatenation, or `shell=True` unless the boundary is deliberate, validated, and documented.
- Apply reasonable timeouts and response-size limits to network requests.
- Security behavior must be covered by automated tests rather than relying on callers to use an API correctly.

## Testing

Tests are executable evidence for behavior and contracts, not a way to count
changed lines or restate the implementation.

- Before writing a test, name the observable behavior or contract it protects
  and the regression that would make it fail. If that cannot be stated clearly,
  do not add the test.
- Confirm the test fails without the change. A test that passes either way is
  evidence of nothing.
- Prefer behavior, interface, integration, and failure-path tests that survive
  internal refactoring and fail when an externally observable contract breaks.
- Do not add tests only to increase coverage, mirror every branch mechanically,
  or confirm that a recently edited file contains an expected line.
- Do not use raw source reads, substring checks, import order, call ordering, or
  private implementation details as primary assertions. In particular, avoid
  tests shaped like `assert "..." in Path(...).read_text()` for source or
  configuration files.
- Structural rules can be tested when the structure is itself a stable contract,
  such as module dependency boundaries or a generated artifact. Test them at the
  right level with an AST/import-graph check, a format-aware parser, an actual
  build/package installation, or CI validation. Do not replace those checks with
  hand-written literal searches through files.
- A fake must behave like the thing it stands for. A stub that accepts what the
  real object rejects, or that answers from a value the caller just supplied,
  makes the assertion circular.
- For metadata-only, lockfile-only, formatting-only, or workflow-only changes,
  prefer the real tool that consumes the artifact (`uv lock --check`, a build,
  package validation, or CI) over a unit test that repeats its text.
- Exact literals are appropriate when they are part of a stable external
  contract, such as user-visible output, a protocol field, or serialized data.
  They are not evidence that an implementation detail exists in a particular
  file.
- Use fakes, stubs, or adapters for external services.
- Cover normal, failure, cancellation, timeout, and repeated-call paths.
- Exercise behavior through public construction and interfaces. Do not use `__new__` or manually populate private fields to bypass initialization for ordinary behavior tests; an isolated lifecycle/Qt harness may do so only when full construction invokes unavailable platform resources, and it must initialize the tested contract explicitly.
- Synchronize asynchronous tests with events, futures, or task handles. Do not rely on arbitrary sleeps or wall-clock timing except for an explicit timeout contract.
- Run the full test suite when changing public behavior, lifecycle management, or cross-module contracts.
- Run the relevant build, packaging, or CI checks when changing dependencies or build configuration.
- Never solve a failing test by deleting coverage, weakening assertions, or expanding exclusions without documenting the reason.
- Record environment limitations and remaining risk when a check cannot run. Anything
  that needs a live compositor, a real player, or synthesized input is not verified
  by the suite; say so rather than implying it is.

## Verification

- Treat `pyproject.toml`, CI workflows, and project documentation as the source of truth for supported runtimes and commands.
- Run the smallest relevant check set during development and the required full checks before submission.
- Typical local commands are:

  ```bash
  uv sync --extra test
  QT_QPA_PLATFORM=offscreen uv run pytest -q
  uv run ruff check .
  uv run ty check
  uv build
  ```

  The Qt tests need a platform plugin; `tests/conftest.py` defaults to `offscreen`,
  and CI runs them under `xvfb-run -a`.

- Cider plugin checks run from `plugins/cider/lyrics/`:

  ```bash
  pnpm install
  pnpm test
  pnpm build
  ```

- Update this section when the canonical project commands change.
- Run `git diff --check` before committing.
- Never claim that a check passed unless it was actually run.
- Distinguish pre-existing failures from regressions introduced by the current change.

## Git Workflow

Use branch names in this form:

```text
<type>/<short-description>
```

Examples:

```text
feat/lyric-hint-from-player
fix/output-hotplug-recovery
refactor/platform-package
ci/python-3.15
```

- One branch should normally contain one logical task.
- Reference the issue in the commit footer or pull-request body rather than in the branch name.
- Do not develop directly on the default branch.
- Do not use destructive Git commands to overwrite user changes.
- Do not force-push over shared history. Force-pushing your own unmerged pull-request
  branch is expected when it is rebased onto a moved target; use `--force-with-lease`.
- Do not commit caches, credentials, build artifacts, or temporary files.
- Review the staged diff before committing.

## Merge policy

- Pull requests land on the default branch as squash merges, which is why every
  commit there carries a `(#NN)` suffix.
- A branch built on another open pull request carries that branch's commits. Once
  the parent is squash-merged those copies no longer match, so rebuild the child
  onto the updated default branch and keep only its own commits.
- Fetch the latest target branch before rebuilding or merging.
- Resolve conflicts by deciding which side is correct. Do not resolve source files
  by keeping both sides mechanically; that has silently duplicated constants,
  reintroduced removed code, and truncated function bodies.
- After resolving, confirm no definition is duplicated and no method was lost.
- Do not rewrite commits authored or signed by another person without explicit approval.

## Conventional Commits

Commit messages must follow:

```text
<type>(<scope>): <summary>

<body>

<footer>
```

Allowed types:

```text
feat fix refactor test docs build ci chore perf revert
```

Rules:

- Use a lowercase type.
- Prefer a scope describing the affected responsibility or module.
- Keep the summary concise, imperative, and without a trailing period.
- One commit should express one logical change.
- Do not use `WIP`, `update`, or otherwise meaningless commit messages.
- Use the body to explain why the change is needed, how behavior changed, and how it was verified.
- Use `Refs: #<number>` for ongoing work.
- Use `Closes #<number>` only when the commit or pull request actually completes the issue.
- Use a `BREAKING CHANGE:` footer for breaking changes.

Example:

```text
refactor(platform): give compositor decisions a home

Move the capability probe and the native bridge wrapper behind a typed
platform package so presentation code asks for a capability instead of
reading the desktop name.

Refs: #16
```

## Completion Checklist

Before committing or opening a pull request, confirm:

- [ ] The change matches the current task scope.
- [ ] Existing user changes were preserved.
- [ ] New interfaces have explicit types.
- [ ] No unnecessary cross-layer dependency was introduced.
- [ ] Asynchronous tasks and resources have clear lifecycles.
- [ ] Relevant tests passed, and each new test was seen to fail without the change.
- [ ] Static-check results were verified.
- [ ] `git diff --check` passed.
- [ ] No secrets or temporary files are included.
- [ ] The commit follows Conventional Commits.
- [ ] The pull request includes a summary, verification results, risks, and references.

---
> Source: [locez/kotonoha](https://github.com/locez/kotonoha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
