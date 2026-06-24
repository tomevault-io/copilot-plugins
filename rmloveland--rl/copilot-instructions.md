## rl

> Developer guide for working on **rl** (a small rlwrap-like PTY + readline wrapper).

# AGENTS.md

Developer guide for working on **rl** (a small rlwrap-like PTY + readline wrapper).

## Repository layout

- `rl.pl` — single-file distribution containing all packages:
  - `RL::Core` — event-loop core logic (kept testable via an injected IO driver).
  - `RL::PTY` — PTY allocation + fork/exec wrapper.
  - `RL::Readline::*` — readline backend selection and adapters.
  - `RL::IO::Real` — wires PTY + readline into the `RL::Core` event protocol.
  - `RL::CLI` — option parsing and process setup.
- `t/core.t` — unit tests for `RL::Core` using a `FakeIO` driver.
- `Makefile` — common dev tasks (formatting, tests, docs, install).

## Quick start (dev)

### Run tests

```sh
make test
```

This runs:

```sh
prove -lv t/core.t
```

### Run locally

Run the script directly:

```sh
perl rl.pl --backend auto -- perl -de0
```

Or install to `~/bin/rl`:

```sh
make install
```

## Makefile targets

- `make test` — run the test suite.
- `make tidy` — run `perltidy` on `rl.pl` and `t/*.t`.
- `make critique` — run `perlcritic rl.pl` (advisory).
- `make podcheck` — run `podchecker rl.pl`.
- `make readme` — regenerate `README.pod` from the embedded POD in `rl.pl`.
- `make man` — build `rl.1` from `README.pod`.
- `make install` — `make test man` then copy `rl` to `~/bin` and install man page.

Notes:
- `make install` writes into user-local locations (`$HOME/bin`, `$HOME/local/share/man/man1`).
- The repo assumes `podselect`, `pod2man`, `pod2html`, `perltidy`, `perlcritic`, `prove` are available when those targets are used.

## Code architecture and conventions

### Single-file packaging

All packages live in `rl.pl`. Tests `require` that file, so keep:

- `unless (caller) { RL::CLI->main(@ARGV); }` at the end, so `require` doesn’t execute the CLI.
- Package boundaries clear and self-contained.

When adding new functionality, prefer adding a new package section (with a header comment) rather than scattering subs across the file.

### `RL::Core` event protocol (important for testability)

`RL::Core->run($io)` expects `$io` to implement:

- `next_event() -> { type => 'line'|'pty'|'child_exit', ... } | undef`
- `send_to_child($bytes)`
- `write_output($bytes)`
- `add_history($line)`

Guideline: if you change this protocol, update `t/core.t` and ideally add new tests first.

### Prompt handling

`RL::IO::Real` treats any child output *not terminated by a newline* as the prompt:

- Output with newlines is forwarded to STDOUT.
- The trailing fragment without newline is captured as `_promptbuf` and rendered via readline.

Be careful when changing `_process_child_output`; it’s central to UX correctness.

### Backends and interleaving behavior

- GNU backend (`Term::ReadLine::Gnu`) supports callbacks and provides interleaved output/input in a single process.
- Perl5 backend (`Term::ReadLine::Perl5`) uses blocking `readline`; output may interleave less cleanly.

If you touch the I/O multiplexer (`IO::Select` logic), manually test both modes.

### PTY behavior

- Child runs under a PTY and gets a controlling terminal (`setsid`, `make_slave_controlling_terminal`, `dup2`).
- Echo is disabled on the slave (best-effort) via `IO::Stty`.

Changes here are platform-sensitive; keep them best-effort and fail-safe.

## Adding/changing behavior: testing guidance

- Add unit tests for logic that can be modeled through `RL::Core` + a fake IO driver.
- Avoid tests that require an actual PTY unless you have a strong reason; they can be flaky in CI/TTY-less environments.

Recommended pattern:
1. Write/extend a `FakeIO` in `t/core.t` (or a new test file) that emits events and captures output.
2. Assert the exact bytes sent to the child and output produced.

## Style / formatting

- Keep `use strict; use warnings;` everywhere.
- Prefer small, well-named helpers for hard-to-read I/O code.
- Run `make tidy` before committing.

## Docs

The canonical docs live as POD at the end of `rl.pl`. Update them there, then regenerate:

```sh
make readme
```

## Common pitfalls / gotchas

- `RL::Readline::Perl5` can emit noisy warnings under `use warnings`; the code filters a specific known warning. Be cautious about expanding warning filters—keep them narrow.
- History persistence:
  - Prefer append-per-line behavior for crash safety.
  - `WriteHistory` support differs by backend; don’t assume it exists.

## Release checklist (lightweight)

- `make tidy`
- `make test`
- Update POD in `rl.pl` if behavior changed
- `make readme man`# AGENTS.md

Developer guide for working on **rl** (a small rlwrap-like PTY + readline wrapper).

## Repository layout

- `rl.pl` — single-file distribution containing all packages:
  - `RL::Core` — event-loop core logic (kept testable via an injected IO driver).
  - `RL::PTY` — PTY allocation + fork/exec wrapper.
  - `RL::Readline::*` — readline backend selection and adapters.
  - `RL::IO::Real` — wires PTY + readline into the `RL::Core` event protocol.
  - `RL::CLI` — option parsing and process setup.
- `t/core.t` — unit tests for `RL::Core` using a `FakeIO` driver.
- `Makefile` — common dev tasks (formatting, tests, docs, install).

## Quick start (dev)

### Run tests

```sh
make test
```

This runs:

```sh
prove -lv t/core.t
```

### Run locally

Run the script directly:

```sh
perl rl.pl --backend auto -- perl -de0
```

Or install to `~/bin/rl`:

```sh
make install
```

## Makefile targets

- `make test` — run the test suite.
- `make tidy` — run `perltidy` on `rl.pl` and `t/*.t`.
- `make critique` — run `perlcritic rl.pl` (advisory).
- `make podcheck` — run `podchecker rl.pl`.
- `make readme` — regenerate `README.pod` from the embedded POD in `rl.pl`.
- `make man` — build `rl.1` from `README.pod`.
- `make install` — `make test man` then copy `rl` to `~/bin` and install man page.

Notes:
- `make install` writes into user-local locations (`$HOME/bin`, `$HOME/local/share/man/man1`).
- The repo assumes `podselect`, `pod2man`, `pod2html`, `perltidy`, `perlcritic`, `prove` are available when those targets are used.

## Code architecture and conventions

### Single-file packaging

All packages live in `rl.pl`. Tests `require` that file, so keep:

- `unless (caller) { RL::CLI->main(@ARGV); }` at the end, so `require` doesn’t execute the CLI.
- Package boundaries clear and self-contained.

When adding new functionality, prefer adding a new package section (with a header comment) rather than scattering subs across the file.

### `RL::Core` event protocol (important for testability)

`RL::Core->run($io)` expects `$io` to implement:

- `next_event() -> { type => 'line'|'pty'|'child_exit', ... } | undef`
- `send_to_child($bytes)`
- `write_output($bytes)`
- `add_history($line)`

Guideline: if you change this protocol, update `t/core.t` and ideally add new tests first.

### Prompt handling

`RL::IO::Real` treats any child output *not terminated by a newline* as the prompt:

- Output with newlines is forwarded to STDOUT.
- The trailing fragment without newline is captured as `_promptbuf` and rendered via readline.

Be careful when changing `_process_child_output`; it’s central to UX correctness.

### Backends and interleaving behavior

- GNU backend (`Term::ReadLine::Gnu`) supports callbacks and provides interleaved output/input in a single process.
- Perl5 backend (`Term::ReadLine::Perl5`) uses blocking `readline`; output may interleave less cleanly.

If you touch the I/O multiplexer (`IO::Select` logic), manually test both modes.

### PTY behavior

- Child runs under a PTY and gets a controlling terminal (`setsid`, `make_slave_controlling_terminal`, `dup2`).
- Echo is disabled on the slave (best-effort) via `IO::Stty`.

Changes here are platform-sensitive; keep them best-effort and fail-safe.

## Adding/changing behavior: testing guidance

- Add unit tests for logic that can be modeled through `RL::Core` + a fake IO driver.
- Avoid tests that require an actual PTY unless you have a strong reason; they can be flaky in CI/TTY-less environments.

Recommended pattern:
1. Write/extend a `FakeIO` in `t/core.t` (or a new test file) that emits events and captures output.
2. Assert the exact bytes sent to the child and output produced.

## Style / formatting

- Keep `use strict; use warnings;` everywhere.
- Prefer small, well-named helpers for hard-to-read I/O code.
- Run `make tidy` before committing.

## Docs

The canonical docs live as POD at the end of `rl.pl`. Update them there, then regenerate:

```sh
make readme
```

## Common pitfalls / gotchas

- `RL::Readline::Perl5` can emit noisy warnings under `use warnings`; the code filters a specific known warning. Be cautious about expanding warning filters—keep them narrow.
- History persistence:
  - Prefer append-per-line behavior for crash safety.
  - `WriteHistory` support differs by backend; don’t assume it exists.

## Release checklist (lightweight)

- `make tidy`
- `make test`
- Update POD in `rl.pl` if behavior changed
- `make readme man`

---
> Source: [rmloveland/rl](https://github.com/rmloveland/rl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
