## homebrew-oss

> This repository is a dedicated Homebrew tap for `MyOSS` command-line tools.

# AGENTS.md

## Purpose

This repository is a dedicated Homebrew tap for `MyOSS` command-line tools.

Keep work in this repository focused on Homebrew packaging, formula maintenance, release updates, and tap automation.

Do not use this repository for implementing application features from upstream projects such as `cli-bot` itself.

## Current Scope

Current formulae:

- `Formula/cli-bot.rb`

Intended future install patterns:

- `brew install joelee/oss/cli-bot`
- `brew install joelee/oss/camwatch`

The repository name is intended to map to the GitHub repository `joelee/homebrew-oss`.

## Current State

- The tap has been scaffolded locally.
- A `cli-bot` formula exists and installs from the published crates.io package.
- The formula currently targets `cli-bot` version `0.1.1`.
- The formula checksum currently matches the crates.io release metadata.
- A macOS GitHub Actions workflow exists at `.github/workflows/test-formula.yml`.
- The workflow is intended to run `brew install`, `brew test`, and `brew audit` for `joelee/oss/cli-bot`.

## Important Constraints

- This repository should stay packaging-focused.
- Upstream application changes belong in the upstream repositories, not here.
- Formula tests should avoid requiring live network access or a running Ollama instance.
- Prefer small, direct formula updates over broad tap refactors.
- Keep install instructions and caveats aligned with upstream defaults, especially the default Ollama model.

## Formula Notes

`Formula/cli-bot.rb` currently:

- installs from `https://static.crates.io/crates/cli-bot/cli-bot-0.1.1.crate`
- uses the checksum of the published crates.io artifact, not the local `target/package` archive
- builds with `cargo install`
- depends on `rust` at build time
- includes `caveats` about Ollama and the default `lfm2:latest` model

## Recommended Release Update Flow

When `cli-bot` publishes a new crates.io version:

1. Update `Formula/cli-bot.rb`
2. Change the crate version in `url`
3. Update the `sha256`
4. Run local Homebrew validation on macOS:
   `brew install --build-from-source joelee/oss/cli-bot`
   `brew test joelee/oss/cli-bot`
   `brew audit --strict joelee/oss/cli-bot`
5. Commit and push the tap update

## Known Gaps

- The tap has been prepared locally but may not yet be pushed to GitHub.
- Homebrew validation was not run locally in the original scaffolding session because that environment was Linux, not macOS.
- Additional formulae such as `camwatch` have not yet been added.

## Suggested Next Steps

1. Push this repository to `https://github.com/joelee/homebrew-oss`
2. Let the macOS GitHub Actions workflow validate the tap
3. Test `brew install joelee/oss/cli-bot` on a real macOS machine
4. Add more formulae under `Formula/` as needed
5. Optionally automate formula version bumps from upstream releases

## Verification

Useful commands for a Homebrew-focused session on macOS:

```bash
brew tap joelee/oss /path/to/homebrew-oss
brew install --build-from-source joelee/oss/cli-bot
brew test joelee/oss/cli-bot
brew audit --strict joelee/oss/cli-bot
```

---
> Source: [joelee/homebrew-oss](https://github.com/joelee/homebrew-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
