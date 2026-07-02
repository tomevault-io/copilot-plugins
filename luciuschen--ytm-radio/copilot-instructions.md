## ytm-radio

> This file is the local source of truth for AI-assisted changes in this

# AGENTS.md

This file is the local source of truth for AI-assisted changes in this
repository. It is adapted from `~/repos/coding-guidelines/general.md` and
`~/repos/coding-guidelines/elisp.md`, plus project-specific ytm-radio rules.

## Project Shape

- Keep the project small until real complexity appears. Prefer one clear file
  with well-named sections over several tiny files with unclear boundaries.
- Split modules only around stable responsibilities such as state, external
  process management, source fetching, helper protocol handling, or UI display.
- Do not add abstraction layers for hypothetical providers. Add a layer only
  when it removes current duplication or gives a real owner to a workflow.
- Do not create vague `utils`, `common`, or pass-through wrapper modules.
- Keep public commands thin: collect interactive input, validate it, call
  internal functions, and show feedback.
- Prefer stock Emacs primitives: `completing-read`, `special-mode`, text
  properties, `start-process`, `make-network-process`, standard timers, and
  standard hooks.

## Diagnosis Discipline

- Find the root cause before changing behavior. Be able to name the failing
  layer before patching timing, caching, rendering, or control flow.
- Do not present a plausible explanation as root cause. Mark it as a
  hypothesis until code inspection, a reproduction, or a failing test confirms
  the actual failing path.
- If one fix fails, narrow the hypothesis and gather more evidence. Do not
  stack another speculative patch on top.
- After two failed fixes on the same issue, stop patching and switch to
  diagnosis only until the failing path is confirmed.
- Fix the layer that owns the problem instead of compensating elsewhere.
- Keep experiments narrow. Prove a new direction with the smallest useful
  slice before expanding scope.
- For dispatcher bugs, test the real dispatch path: keymaps, buttons,
  commands, hooks, async callbacks, and public entry points. Helper-level tests
  are not enough when the bug is in routing.
- For user-visible bug fixes, prefer red before green: reproduce the failure in
  a test or a minimal live check, confirm it fails, then change behavior.
- Do not leave heuristic shortcuts, silent partial implementations, duplicated
  logic, or dead code introduced during diagnosis.

## Emacs Lisp Rules

- Every `.el` file uses lexical binding.
- Loading package files must not alter active editing behavior. Activation
  happens through explicit commands or user-enabled modes.
- Use the `ytm-radio-` prefix for public API and `ytm-radio--` for private
  helpers and private modes.
- Never call another package's private double-dash symbols.
- Public commands and user-facing modes need `;;;###autoload`.
- Do not autoload internal helpers, variables, or private modes.
- Public `defun`, `defmacro`, `defcustom`, and `defvar` forms must have
  docstrings.
- Docstring first lines must be complete sentences ending in a period.
- Argument names mentioned in docstrings should be uppercased.
- Use precise `defcustom :type` declarations and always set `:group`.
- Use `defvar-local` and `setq-local` for per-buffer state. Major modes must
  make their state buffer-local.
- Read-only UI buffers derive from `special-mode`.
- Use text properties for data-bearing annotations; use overlays only for
  ephemeral visuals.
- Build render buffers from structured state, not by reparsing visible text.
- Prefer `when-let*`, `if-let*`, `pcase`, and `pcase-let` for structured
  conditional binding and destructuring.
- Use `user-error` for user-caused problems such as missing external programs,
  invalid configuration, or empty catalogs.
- Use `error` for programmer bugs. Catch errors only at external process,
  optional display, or top-level helper protocol boundaries where recovery is
  meaningful.
- Require runtime dependencies explicitly, for example `(require 'cl-lib)`.
  Do not rely on transitive loading.
- Avoid `eval-when-compile` for dependencies needed at runtime.
- Before using a newer Emacs API, verify when it was introduced and do not
  exceed the declared Emacs baseline without updating package metadata and docs.

## MELPA / Package Rules

- Main package first line must be:
  `;;; ytm-radio.el --- Short description -*- lexical-binding: t; -*-`
- The package description must not contain "for Emacs" or the package name.
  Keep it under 60 characters.
- The main package file must include `;; Author:`, `;; URL:`, `;; Version:`,
  and `;; Package-Requires:`.
- `Package-Requires` must list all direct dependencies with minimum versions,
  including the declared Emacs baseline.
- Package metadata belongs in the main package file only. Split implementation
  files must not duplicate `Package-Requires`.
- Split implementation files still need formal license metadata, preferably
  `;; SPDX-License-Identifier:`.
- Keep required MELPA checklist attribution such as `;; Assisted-by: ...` in
  the main package file when tooling materially assisted the package.
- Every distributable `.el` file ends with `(provide 'feature)` and
  `;;; file.el ends here`.
- Run byte-compilation with zero warnings.
- Run `checkdoc` with zero warnings on distributable Elisp files.
- Run `package-lint` with zero warnings for MELPA/ELPA-style package changes.
  If `package-lint` is unavailable locally, say so explicitly in the final
  report.
- When using `package-lint` on split implementation files, configure the main
  file instead of duplicating package metadata.

## ytm-radio Boundaries

- Do not implement YouTube or YouTube Music reverse-engineering in Elisp.
  Treat `yt-dlp` and the Rust helper as compatibility boundaries.
- Account access belongs in the external Rust CLI under `helper/`. Do not add
  a Python helper or an Emacs dynamic module.
- Keep the helper short-lived by default: one command reads configuration,
  writes one JSON response to stdout, and exits.
- The supported account-auth workflow is a browser login window driven by
  `auth login-window` through Chromium DevTools or Firefox-family WebDriver
  BiDi. Do not add browser-cookie database import, copied-header import,
  Dia-specific restart commands, or other fallback auth paths unless the
  product decision changes.
- Do not duplicate browser-specific cookie database crypto in Rust.
- Version the helper JSON envelope. Emacs must reject unsupported schema
  versions instead of guessing.
- Store login browser options in Emacs configuration; store session material
  only in the dedicated helper auth file with private permissions.
- Never write cookie contents or auth headers to Emacs durable state, stdout,
  logs, fixtures, or test failure messages.
- Store durable state separately from process state. Do not persist process
  objects, sockets, timers, or IPC handles.
- Keep child-frame rendering deterministic from current track/player state.
  Do not derive behavior from the displayed buffer text.

## Rust Helper Rules

- Keep authentication details out of stdout, logs, fixtures, and test failure
  messages.
- Live network checks stay separate from deterministic unit tests.
- Run `cargo fmt --check`, `cargo clippy -- -D warnings`, and `cargo test`
  for helper changes.
- Helper command output must remain machine-readable JSON on stdout; diagnostic
  text belongs on stderr.

## Tests and Verification

- `make check` is the normal local verification path.
- For behavior depending on YouTube, YouTube Music, `yt-dlp`, browser cookies,
  or `mpv`, keep network/live checks separate from deterministic unit tests.
- Match test weight to change size. Use the smallest test that proves the
  behavior.
- For user-visible bug fixes, add or update a test that proves the regression
  unless an existing test already covers the real dispatch path.
- Tests must fail when the code is wrong. Avoid assertions that merely lock in
  implementation details.
- Read the changed diff before finalizing. Remove duplicated logic, dead code,
  and temporary diagnostics.

## Documentation Discipline

- User-visible changes must update user documentation in the same change when
  they affect commands, key bindings, defaults, configuration, setup, or
  workflows. Use `README.md` for user operation and `prd.md` for product
  behavior, scope, and UX decisions.
- Code is the source of truth. If code and docs diverge, fix docs immediately.
- Optimize Markdown for rendered reading, not source-width aesthetics. Do not
  rewrap unchanged prose just to satisfy a column width.
- When documentation is hard to read, improve structure with headings, focused
  bullets, or tables instead of source-only line wrapping.

## Postmortem Conventions

The `postmortem/` directory records design decisions and lessons learned. Read
relevant records before significant workflow, architecture, or integration
changes.

Name postmortem files with a three-digit sequence number followed by a short
slug, for example `001-helper-boundary.md`. Do not use date-prefixed filenames.

Write a postmortem when:

- adding or changing a user-visible workflow;
- choosing between non-obvious architectural approaches;
- integrating an optional dependency or external system;
- reverting or abandoning an approach, especially to document why it was wrong;
- deliberately deferring a known limitation.

Postmortems are historical records, not current product documentation. Do not
rewrite old records just to match current behavior; write a new record for the
later design and optionally add a short superseded note to the old one.

Postmortems must explain why, not restate the code. A record that only
describes what was done adds no value.

---
> Source: [LuciusChen/ytm-radio](https://github.com/LuciusChen/ytm-radio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
