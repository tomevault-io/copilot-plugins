## blockinputkit

> - Keep `AGENTS.md` information concise to minimize token usage.

## Keep Guidance Current

- Keep `AGENTS.md` information concise to minimize token usage.
- Keep `AGENTS.md` accurate when changes create useful future-agent context.
- Put new rules in the narrowest `AGENTS.md` that covers the affected files.
- Categorize bullets inside of `AGENTS.md` files with their own sections, if there are enough points; split dense rules into short sub-bullets with bold imperative leads.
- Call out oversized guidance files or sections that should be split.
- When adding a nested `AGENTS.md`, also add sibling `CLAUDE.md` as `ln -s AGENTS.md CLAUDE.md`, then list the new scope below.
- Update `README.md` plus scoped guidance when dependencies, project structure, or lint rules change.

## Scoped Guidance

Read the nearest `AGENTS.md` before editing. Current scopes:

- `AGENTS.md`: repo-wide workflow.
- `.agents/AGENTS.md`: repo-local agent workflow guidance.
- `.agents/checks/AGENTS.md`: repo-local review, audit, and check workflows.
- `Sources/BlockInputKit/AppKit/AGENTS.md`: AppKit editor surface and cross-cutting editor rules.
- `Sources/BlockInputKit/AppKit/BlockItem/AGENTS.md`: AppKit block item reuse, chrome, markers, and metrics.
- `Sources/BlockInputKit/AppKit/Mutation/AGENTS.md`: AppKit store sync, editing operations, and command shortcuts.
- `Sources/BlockInputKit/AppKit/Reordering/AGENTS.md`: AppKit reorder handles, drag starts, and ordered-list reorder normalization.
- `Sources/BlockInputKit/AppKit/Selection/AGENTS.md`: AppKit keyboard, mouse, chrome, cancellation, and debug selection behavior.
- `Sources/BlockInputKit/AppKit/SyntaxHighlighting/AGENTS.md`: AppKit lite syntax highlighting and code parsing.
- `Tests/BlockInputKitTests/AGENTS.md`: test organization and coverage expectations.
- `Tests/BlockInputKitTests/AppKit/BlockItem/AGENTS.md`: AppKit block item tests.
- `Tests/BlockInputKitTests/AppKit/Mutation/AGENTS.md`: AppKit mutation and document-store tests.
- `Tests/BlockInputKitTests/AppKit/Performance/AGENTS.md`: AppKit large-document and hot-path performance tests.
- `Tests/BlockInputKitTests/AppKit/Reordering/AGENTS.md`: AppKit reordering tests.
- `Tests/BlockInputKitTests/AppKit/Selection/AGENTS.md`: AppKit multi-selection and selection chrome tests.
- `Tests/BlockInputKitTests/AppKit/SyntaxHighlighting/AGENTS.md`: AppKit syntax highlighting and code parser tests.
- `Tests/BlockInputKitTests/AppKit/Snapshots/AGENTS.md`: AppKit snapshot matrix and baseline expectations.

## Build And Test

- First-time setup: `./scripts/setup.sh`.
- Build: `./scripts/build.sh`.
- Run the demo app: `./scripts/run-demo.sh`.
- Run the 100k mutation benchmark: build `BlockInputKitDemo`, then run `.build/xcode/Build/Products/Debug/BlockInputKitDemo --benchmark-100k-mutations [iterations]`.
- Test: `./scripts/test.sh`, or pass focused identifiers as arguments when supported.
- Snapshot workflows use `./scripts/snapshots.sh`; verify snapshots before committing UI changes.
- Lint: `./scripts/lint.sh`.
- Ordered workflows must stay serial, never via `multi_tool_use.parallel`: build-then-run, build-then-test, record-then-verify, lint-then-commit.
- Add temporary logs early when useful; observe them yourself, then remove them after confirming the fix.

### `xcsift` Output

- Build/test/snapshot wrappers should pipe `xcodebuild` through `xcsift -f toon -w` when installed; treat TOON `status` and `summary` as the concise result. `status` is generally `success` or `failed`.
- `summary:` contains indented count fields such as `errors`, `warnings`, `failed_tests`, and `linker_errors`; it can also include `passed_tests`, `build_time`, `test_time`, and `coverage_percent`.
- Inspect TOON sections such as `errors[n]{file,line,message}`, `warnings[n]{file,line,message,type}`, `failed_tests`, `linker_errors`, `slow_tests`, `flaky_tests`, `build_info`, and `executables` when present.
- In `errors[n]{file,line,message}` rows, values are ordered as file path, line number, and quoted message.
- In `warnings[n]{file,line,message,type}` rows, values are ordered as file path, line number, quoted message, and warning type such as `compile` or `swiftui`.
- `linker_errors` entries include `symbol`, `architecture`, `referenced_from`, `message`, and `conflicting_files`; duplicate symbol failures list object paths in `conflicting_files`.
- `failed_tests` entries include `test`, `message`, `file`, `line`, and `duration`; `slow_tests` entries include `test` and `duration`; `flaky_tests` is a list of test names.
- `build_info` can include `targets[n]{name,duration,phases,depends_on}` rows with per-target timing, phases, and dependencies.
- `executables[n]{path,name,target}` lists built artifacts with their path, name, and target.

## Lint

- Use SwiftLint from the repo root without `--config` so nested configs apply.
- Install repo hooks with `./scripts/setup.sh`.
- Treat every SwiftLint warning or error as blocking; do not commit while lint reports any violation.
- New Swift should follow `.swiftlint.yml`: no force unwraps outside tests, no force casts, prefer `let`, max line length 150.
- Keep lint exceptions centralized in `.swiftlint.yml`; do not add inline SwiftLint suppression comments.

## Code Style

- Put private types below public types.
- Add concise comments only where they help future readers.
- Search for same-type companion files before editing behavior.
- Split large types into focused companions like `Type+Feature.swift`.
- Cover AppKit focus/selection changes with mounted-view tests when model-only assertions could miss visible `NSTextView` state.
- Document public completion popup configuration changes in `README.md`; overlay providers choose both popup parent view and frame in that parent's coordinate space.
- Document slash-command behavior in `README.md` when changing availability, suggestion construction, chip rendering, or click handling; keep the demo on `.anywhere` availability unless the product requirement changes.

## Commits

When creating commits, use an appropriate trailer in the message.

- If you are Claude: `Co-authored-by: Claude <noreply@anthropic.com>`
- If you are Codex: `Co-authored-by: Codex <noreply@openai.com>`

---
> Source: [afollestad/BlockInputKit](https://github.com/afollestad/BlockInputKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
