## gh-smart-clone

> This repository publishes `gh-smart-clone`, a script-based GitHub CLI extension, plus an agent skill that teaches future agents when to use it.

# Agent Instructions

This repository publishes `gh-smart-clone`, a script-based GitHub CLI extension, plus an agent skill that teaches future agents when to use it.

## Project Shape

- `gh-smart-clone` is the extension executable. GitHub CLI expects the repo name and root executable name to match.
- `test/run-tests.bash` is the main test suite. It uses a fake `gh` binary and real local `git` repos so tests can verify remotes and local config without touching GitHub.
- `skills/gh-smart-clone/SKILL.md` is a scenario-led agent skill. It is not a README duplicate; it should teach agents when to use normal, OSS, and contribution modes.
- `README.md` is user-facing documentation.
- `CHANGELOG.md` records user-visible changes.

## Core Product Invariants

Preserve the three-mode model:

- Normal mode, `gh smart-clone OWNER/REPO`, is for first-party or maintained repos. It must not create forks.
- OSS mode, `gh smart-clone --oss OWNER/REPO`, is for external inspection/debugging. It must not create forks.
- Contribution mode, `gh smart-clone --contribute OWNER/REPO`, is the only mode that may create or reuse forks. It should place the checkout under the upstream OSS path, clone the fork as `origin`, and configure `upstream` to the original project.

Keep path semantics clear:

- The `owner/repo` path segment represents canonical project identity.
- The `oss/` root segment represents external-work relationship.
- Fork owner is a push mechanism, not the project identity.

Treat GitHub mutations as explicit:

- Fork creation must require `--contribute`.
- `--dry-run` and `--print-path` must never create forks, clone, or mutate remotes.
- Existing checkout mutation must require `--contribute --reconfigure`.
- Do not silently rewrite remotes or local git identity for an existing checkout.

## Keep Surfaces in Sync

When changing CLI behavior, update all relevant surfaces in the same change:

- `gh-smart-clone` help text and `VERSION`
- `README.md`
- `CHANGELOG.md`
- `test/run-tests.bash`
- `skills/gh-smart-clone/SKILL.md`
- `skills/gh-smart-clone/agents/openai.yaml`, if skill-facing behavior or positioning changes

At minimum, if a feature changes how agents should choose between normal, `--oss`, `--contribute`, or `--reconfigure`, update `SKILL.md`. The skill is the agent-facing operating model.

## Skill Guidance

`SKILL.md` should stay scenario-led. Prefer guidance about when and why to use each mode over exhaustive flag documentation.

The frontmatter `description` is the trigger surface for agents. Keep it broad enough to trigger for GitHub cloning, OSS checkouts, fork creation/reuse, remotes, and local git identity setup. Do not reduce it to a vague slogan.

Do not hardcode personal defaults in the skill. Account names, emails, and SSH aliases may appear only as clearly labeled examples.

## Testing Expectations

Run these before committing:

```sh
shellcheck gh-smart-clone test/run-tests.bash
./test/run-tests.bash
```

Validate the skill when `SKILL.md` changes. The local validator may need PyYAML:

```sh
python3 /path/to/quick_validate.py skills/gh-smart-clone
```

If your Python environment lacks PyYAML, use a temporary venv outside the repo. Do not vendor validator dependencies into this project just for local validation.

## Test Design

Keep tests non-mutating by default:

- Use the fake `gh` in `test/run-tests.bash` for fork and clone behavior.
- Use real local `git` repos inside temporary directories to verify remotes and git config.
- Avoid tests that create real forks, clone public repos, or mutate GitHub state.
- Live `gh` checks, if useful, should be dry-run or metadata-only and not required for CI.

For safety behavior, add tests for both success and refusal cases. Important refusal cases include missing required forks with `--no-fork`, wrong-parent forks, non-fork repos at the fork target, same upstream owner as fork owner, and existing checkouts without `--reconfigure`.

## Release and Packaging Notes

This is a script extension, so the root executable must remain executable:

```sh
chmod +x gh-smart-clone test/run-tests.bash
```

After pushing behavior changes, verify GitHub Actions passes. For local installed-extension checks:

```sh
gh extension upgrade smart-clone
gh smart-clone --version
gh smart-clone --contribute --dry-run OWNER/REPO
```

Only use real non-dry-run `--contribute` when the user explicitly wants to create/reuse a fork and accepts that GitHub state may change.

---
> Source: [tmchow/gh-smart-clone](https://github.com/tmchow/gh-smart-clone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
