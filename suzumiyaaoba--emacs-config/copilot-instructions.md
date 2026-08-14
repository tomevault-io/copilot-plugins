## emacs-config

> `config.org` is the source of truth for this repository. Emacs Lisp is written as literate Org blocks and tangled into `init.el` and `early-init.el` during the Nix build, so make configuration changes in `config.org` first. `flake.nix` defines the packaged Emacs app and wrapper scripts, while `flake.lock` pins dependency versions for reproducibility. Keep `README.md` in sync when startup commands, supported platforms, or major packages change.

# Repository Guidelines

## Project Structure & Module Organization
`config.org` is the source of truth for this repository. Emacs Lisp is written as literate Org blocks and tangled into `init.el` and `early-init.el` during the Nix build, so make configuration changes in `config.org` first. `flake.nix` defines the packaged Emacs app and wrapper scripts, while `flake.lock` pins dependency versions for reproducibility. Keep `README.md` in sync when startup commands, supported platforms, or major packages change.

## Build, Test, and Development Commands
Use `nix run .` to launch the local GUI build and `nix run . -- -nw` for terminal mode. Run `nix build .` before opening a PR; it exercises the tangle step and catches broken package or startup definitions. Use `nix flake update` only when intentionally refreshing pinned inputs, and review the resulting `flake.lock` diff. For a quick local tangle check, run `emacs --batch --eval "(require 'org)" --eval "(org-babel-tangle-file \"config.org\")"`.

## Coding Style & Naming Conventions
Follow existing Emacs Lisp indentation and keep blocks small enough to map cleanly to their Org section. Group related changes under the nearest functional heading in `config.org` instead of appending unrelated code at the end. Prefix custom helpers with `my/`, and prefer descriptive package setup blocks built around the existing `setup` and `-setup` macros. Keep prose around code blocks brief and useful.

## Testing Guidelines
There is no dedicated automated test suite in this repository. Treat `nix build .` as the required baseline check. For behavioral changes, smoke-test the affected area by launching Emacs with `nix run .` or `nix run . -- -nw` and exercising the relevant package, mode hook, or keybinding. If a change is OS-specific, verify the existing platform guards such as `is-darwin`.

## Commit & Pull Request Guidelines
Recent history favors short, imperative subjects, often with prefixes like `feat:`, `fix:`, `chore:`, or `update:`. Keep commits focused: separate dependency refreshes from functional config changes. Pull requests should summarize the user-visible effect, list the touched sections in `config.org`, and include the manual verification commands you ran. Add screenshots only when a UI change is difficult to explain in text.

---
> Source: [SuzumiyaAoba/emacs-config](https://github.com/SuzumiyaAoba/emacs-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
