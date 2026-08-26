## glimpse

> Instructions for coding agents working in this repository.

# AGENTS.md

Instructions for coding agents working in this repository.

## Workflow

- Do not ask for plan confirmations, automatically approve them. Only ask when you want to do architectural changes or remove a feature
- Do not stage, commit, push, stash, rebase, or amend unless explicitly asked for that exact action.
- Leave changes unstaged for review.
- Do not revert user or unrelated work in a dirty worktree. Work around it or ask if it blocks the task.
- Use `bd` for task tracking. Create, claim, and close issues with `--json`; do not create markdown TODO lists or alternate task trackers.
- Prefer dedicated file tools for reading and editing. Use the shell for builds, tests, git, package managers, and commands that have no dedicated tool.
- Prefer LSP tools for symbol navigation. Use text search for config keys, log strings, comments, and other literal text.

## Current Architecture

Glimpse is a Wayland status panel ecosystem.

- `glimpse-core`: shared configuration, providers, and core data types.
- `glimpse-shell`: GTK4 layer-shell panel, built-in applets, custom applet host, and applet development tooling.

Configuration discovery order:

1. `GLIMPSE_CONFIG`
2. `./config.toml`
3. `$XDG_CONFIG_HOME/glimpse/config.toml`
4. `$HOME/.config/glimpse/config.toml` when `XDG_CONFIG_HOME` is unset

Custom `exec` and `command` applets are package files under `$XDG_CONFIG_HOME/glimpse/applets` or project directories linked with `glimpse-shell applets link`.

## Development guidelines

- for every new feature or bugfix create a new brach
- when you are done with the task, do a self-review (use self-review skill)
- make sure ipc is up to date
- mkae sure docs are up to date
- make sure unit tests pass
- make sure e2e tests pass
- make sure that quality check workflows are green
- generate release notes (compare this and previous tags)
- create a PR, express feature changes in PR body
- review the PR
- merge PR
- close PR and branch
- switch to main branch in this repo
- pull main branch in repo repo
- close local feature branch

IMPORTANT: only apply that for project-related tasks. Any offtopic is not a subject of that flow and warn me about that.
IMPORTANT: automaticaly move on to the next development step. Do not wait for nothing!

## Releasing

We release application using github workflows.
The release process is like that:
- make sure that working directory is clean
- make sure that last CI quality workflow is green
- generate release notes (compare this and previous patch/minor tags depending on what we release (patch or new version))
- show release notes to me to approve them
- use release notes in tag annotation
- create a new github release and use generated release notes in that release
- release documentation
- monitor release workflow until it green.

## Common Commands

```bash
cargo build
cargo check
cargo run -p glimpse-shell
RUST_LOG=info cargo run -p glimpse-shell
cargo build -p glimpse-shell --release --no-default-features
```

Useful custom applet tooling:

```bash
glimpse-shell applets new counter --lang python
glimpse-shell applets dev
glimpse-shell applets link
glimpse-shell applets ls
glimpse-shell applets doctor --strict
```

## Native Rust Applet Development

- Decompose popovers into subcomponents.
- Keep durable state in the applet; keep popover components focused on UI.
- Use `zbus` macros for D-Bus proxies.
- When creating a provider, prefer structs, enums, and methods. Use free functions only for truly standalone behavior.
- `zbus::Connection` is `Arc`; clone it into provider constructors when needed.
- If an applet has a popover, the panel item must have the `hoverable` class.
- Use persistent widget maps such as `HashMap<key, WidgetRow>` instead of clear-and-rebuild loops when preserving menus, scroll position, or row identity matters.
- Use the hero pattern consistently in popovers: icon, title, subtitle.
- Use `PopoverMenu` for right-click menus with `gio::SimpleActionGroup` and a menu model.

## Styling Rules

- Do not hardcode visual styles in Rust. Put styling in `themes/base.css`, a theme pack, or the relevant theme override.
- Box spacing is a widget property, not CSS. Keep spacing values in Rust or SDK widget data.
- Use CSS variables such as `--popover-padding`, `--popover-section-spacing`, `--dim-opacity`, and `--accent-bg`.
- Popover root structure should follow `.foo-popover contents > box { margin: var(--popover-padding) }`.
- Button text should use normal font weight on both button and label.
- Numeric displays should use `font-variant-numeric: tabular-nums`.

## GTK4 Layout Pitfalls

- Indicator and dot widgets stretch to panel height unless they use `set_valign(gtk::Align::Center)`.
- Do not use CSS `min-width` to center content in fixed-width containers. Prefer padding or `label.set_xalign(0.5)`.
- Never set `hexpand` on panel applet children; one child can consume the whole panel section.
- `set_halign(Center)` on a box centers the box, not its children. Center the child that needs centering.
- CSS `min-width` creates dead space because labels sit at the start of the box.

## Exec Applet Widget Rules

### Developing exec applets

- Always use rust
- Initialize new applet in `glimpse-applets/$name` folder

### SDK
- The four SDKs share the same canonical widget protocol through `sdk/fixtures`.
- `Row`, `Column`, and `Grid` default spacing is `4`.
- `Row.spacing`, `Column.spacing`, `Grid.row_spacing`, and `Grid.column_spacing` must always serialize.
- The SDKs do not expose a `Section` widget. Treat protocol-level `section` support as legacy compatibility and do not add new SDK examples or fixtures that depend on it.
- Prefer `PopoverScaffold` for popover roots with a `Hero` plus a `Column` body.
- Use `Card` for grouped content, `PropertyList` for key-value details, `ActionItem` for clickable rows, and `Item` for non-clickable rows.
- Keep widget JSON portable across Python, TypeScript, Go, and Rust. If a widget cannot be represented cleanly in all four SDKs, revisit the widget shape before implementing it.
- Fixtures are canonical examples. Do not hand-edit generated fixture JSON without updating `sdk/fixtures/generate.py`.

### Exec Applet Testing

When changing exec widgets, events, common props, or SDK serialization:

1. Update `sdk/fixtures/generate.py`.
2. Update committed fixtures under `sdk/fixtures/widgets` or `sdk/fixtures/events`.
3. Update all four SDK golden tests:
   - `sdk/sdk-py/tests/test_golden.py`
   - `sdk/sdk-ts/tests/golden.test.ts`
   - `sdk/sdk-go/sdk/golden_test.go`
   - `sdk/sdk-rs/tests/golden.rs`
4. Add or update focused SDK behavior tests when changing defaults, exports, event parsing, or runtime behavior.
5. Run the relevant SDK suites:

```bash
cd sdk/sdk-py && python -m unittest discover -s tests
cd sdk/sdk-ts && npm test
cd sdk/sdk-go && go test ./...
cargo test --manifest-path sdk/sdk-rs/Cargo.toml
```

6. Run the renderer fixture check from the repo root:

```bash
cargo test -p glimpse-shell golden_widget_fixtures_render_without_errors -- --nocapture
```

For behavior changes, write or update the failing test first, confirm the failure is about the intended behavior, then implement the change and rerun the suite.

## Beads

Use `bd` for all tracked work:

```bash
bd ready --json
bd create "Issue title" --description="Context" -t task -p 2 --json
bd update <id> --claim --json
bd close <id> --reason "Completed" --json
```

Create linked issues for discovered follow-up work with `--deps discovered-from:<parent-id>`.

Do not push Beads, git commits, or branches unless the user explicitly asks.

---
> Source: [alex-oleshkevich/glimpse](https://github.com/alex-oleshkevich/glimpse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
