## viscribe

> This is the canonical agent guide for the Viscribe repository. Keep the root

# Agent Guidelines for Viscribe

This is the canonical agent guide for the Viscribe repository. Keep the root
`AGENTS.md` as a lightweight discovery pointer or symlink to this file so
tooling can find the shared instructions while `.agents/` remains the source of
truth.

Viscribe is a Python and TypeScript SDK for structured image understanding with
OpenAI-compatible vision models. The public surface is intentionally small:
image methods live under `viscribe.images` and `client.images` in Python, and
under `images` and `client.images` in TypeScript.

## Maintenance Contract

- Keep this file concise and repo-wide.
- Put narrow, reusable workflows in `.agents/skills/**`.
- Update this file when repository structure, required verification, licensing
  guidance, or durable agent conventions materially change.
- Do not codify one-off preferences or task-local decisions here.
- Preserve user changes in the working tree. Do not revert unrelated edits.

## Start Here By Task

- Product and SDK architecture principles:
  `.agents/ARCHITECTURE_PRINCIPLES.md`
- Shared agent setup and config:
  `.agents/README.md`
- Stress-testing plans, public API changes, or larger refactors:
  `.agents/skills/grill-me/SKILL.md`
- Creating or refining shared skills:
  `.agents/skills/skill-creator/SKILL.md`
- Python SDK implementation:
  `python/src/viscribe/`
- Python tests and examples:
  `python/tests/`, `python/examples/`
- TypeScript SDK implementation:
  `typescript/src/`
- TypeScript tests and examples:
  `typescript/tests/`, `typescript/examples/`
- CI, issue templates, PR templates, and dependency automation:
  `.github/`
- Public docs and package READMEs:
  `docs/`, `README.md`, `CONTRIBUTING.md`, `SECURITY.md`,
  `python/README.md`, `typescript/README.md`
- Shared assets:
  `assets/`

Read the smallest set of files needed for the task. More-specific guidance in a
subdirectory takes precedence over this root guide for that scoped area.

## Default Planning Behavior

Use `.agents/skills/grill-me/SKILL.md` before finalizing non-trivial public API
changes, SDK architecture decisions, refactor strategies, compatibility breaks,
or release plans. Skip it for small mechanical tasks, direct implementation
requests with an already-approved plan, or narrow verification work.

## Project Structure

```text
viscribe/
├─ python/                  # Python package, tests, and runnable examples
├─ typescript/              # TypeScript package, tests, and runnable examples
├─ docs/                    # Mintlify documentation site
├─ assets/                  # Shared README/package assets
├─ .github/                 # GitHub Actions, Dependabot, and templates
├─ .agents/                 # Shared agent instructions and skills
├─ README.md                # Root package documentation
├─ CONTRIBUTING.md          # Contribution and branch workflow
├─ SECURITY.md              # Vulnerability reporting policy
├─ ROADMAP.md               # Product direction
└─ LICENSE                  # MIT license
```

## Core Commands

Python:

- Install/sync dependencies from `python/`: `uv sync`
- Test from `python/`: `uv run python -m pytest`
- Lint from `python/`: `uv run ruff check .`
- Run an example from `python/`: `uv run python examples/<name>.py`

TypeScript:

- Install dependencies from `typescript/`: `npm install`
- Test from `typescript/`: `npm test`
- Typecheck from `typescript/`: `npm run typecheck`
- Build from `typescript/`: `npm run build`
- Run examples from `typescript/`: `npm run example`,
  `npm run example:describe`, `npm run example:classify`,
  `npm run example:ask`, `npm run example:compare`, or
  `npm run example:client`

Docs:

- Install the Mintlify CLI if needed: `npm install -g mint`
- Run Mint with an LTS Node version supported by the CLI. Mint currently
  rejects Node 25+, so switch to Node 22 or 24 before running `mint` if the
  active shell is newer. Temporary fallback:
  `cd docs && npx -y -p node@22 node "$(command -v mint)" dev --no-open`
- Preview docs locally from `docs/`: `mint dev --no-open`
- Validate the docs build from `docs/`: `mint validate`
- Check docs links from `docs/`: `mint broken-links`
- Validate docs config from the repo root:
  `python3 -m json.tool docs/docs.json >/dev/null`

Root and agent files:

- Install root tooling: `npm install`
- Check formatting: `npm run format:check`
- Format supported files: `npm run format`
- Check a commit or PR title: `printf '%s\n' "feat: example" | npm run commitlint -- --verbose`
- Run semantic-release locally only in dry-run mode unless explicitly publishing:
  `npm run release -- --dry-run`
- Validate JSON: `python3 -m json.tool .agents/config.json`
- Validate YAML: `uv run --with pyyaml python -c "import pathlib, yaml; [yaml.safe_load(p.read_text()) for p in pathlib.Path('.github').rglob('*.yml')]"`
- Lint GitHub Actions workflows: `actionlint .github/workflows/*.yml`
  or, if `actionlint` is not installed, `go run github.com/rhysd/actionlint/cmd/actionlint@latest .github/workflows/*.yml`
- Check whitespace: `git diff --check`

## Verification Defaults

| Change scope                                                                                                                                | Minimum verification                                                                                                                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `python/**`                                                                                                                                 | From `python/`: `uv run python -m pytest` and `uv run ruff check .`                                                                                                                                                              |
| `typescript/**`                                                                                                                             | From `typescript/`: `npm test`, `npm run typecheck`, and `npm run build`                                                                                                                                                         |
| `README.md`, package READMEs, or examples                                                                                                   | Run the affected examples when credentials are available, plus the relevant package tests                                                                                                                                        |
| `docs/**`                                                                                                                                   | `python3 -m json.tool docs/docs.json >/dev/null`, targeted docs content review, `npm run format:check`, and `git diff --check`; run `mint validate` or `mint dev --no-open` when the Mintlify CLI is available                   |
| `.github/**`                                                                                                                                | Validate YAML, run `actionlint .github/workflows/*.yml` or the `go run github.com/rhysd/actionlint/cmd/actionlint@latest .github/workflows/*.yml` fallback, then run the workflow-equivalent local commands for touched packages |
| `.agents/**`                                                                                                                                | `python3 -m json.tool .agents/config.json`, affected skill validation when relevant, and `git diff --check`                                                                                                                      |
| Root tooling (`package.json`, `package-lock.json`, `release.config.cjs`, `commitlint.config.cjs`, `prettier.config.cjs`, `.prettierignore`) | `npm install`, `npm run format:check`, commitlint smoke test, and `npm run release -- --dry-run` when release config changed                                                                                                     |
| Root metadata (`LICENSE`, `.gitignore`, `CONTRIBUTING.md`, `SECURITY.md`, `ROADMAP.md`)                                                     | Targeted review plus `npm run format:check` and `git diff --check`                                                                                                                                                               |

Prefer targeted verification that matches the touched surface. If a command
cannot run because credentials or a local toolchain are missing, say so clearly
in the final answer.

## Repo Rules

- Keep changes scoped and aligned with the existing dual-package structure.
- Keep Python and TypeScript API behavior aligned unless language idioms require
  different naming.
- Use Python `snake_case` option names and TypeScript `camelCase` option names.
- Keep image operations under the images namespace. Do not add direct root-level
  image methods unless the user explicitly requests a new public API shape.
- Python exposes sync helpers and async `a*` helpers. TypeScript is async-native
  and should not expose `a*` aliases.
- Preserve the result-wrapper contract for image methods: structured `data`,
  provider `raw`, and usage metadata.
- Reuse the existing OpenAI-compatible Chat Completions flow unless a task
  explicitly changes providers or transport.
- Preserve structured-output parsing, strict JSON Schema conversion, refusal
  handling, finish-reason errors, usage metadata, and image source validation
  when extending image methods.
- Keep support for image URL, base64, and local path inputs consistent across
  Python and TypeScript.
- Keep docs, tests, and examples in sync with actual public APIs.
- Update `docs/` whenever public APIs, examples, package setup, supported
  environment variables, model configuration, or project workflow changes.
- Keep `docs/` focused on the current open-source Python and TypeScript SDKs,
  and remove pages that describe unavailable product surfaces.
- Update `.env.example` when adding or changing required environment variables.
- Keep `.github/` focused on the current repo shape. TypeScript CI runs from
  `typescript/`; Python CI runs from `python/`.
- Do not hand-edit generated or installed artifacts unless the task is
  specifically about those artifacts:
  - `python/.venv/**`
  - `python/.pytest_cache/**`
  - `python/.ruff_cache/**`
  - `typescript/node_modules/**`
  - `typescript/dist/**`
  - `typescript/coverage/**`
- Never commit secrets, API keys, private tokens, local credentials, or `.env`
  files.

## Contribution And Release Flow

- Normal work should branch from and open pull requests against `develop`.
- Treat direct pushes to `main` as blocked by branch protection. Use `main` only
  for maintainer release promotion or urgent hotfix PRs.
- Use Conventional Commit and semantic-release-compatible wording in PR titles
  and commits: `feat:`, `fix:`, `perf:`, `bump:`, `docs:`, `test:`,
  `refactor:`, `ci:`, `build:`, or `chore:`.
- Mark breaking changes with `!` or a `BREAKING CHANGE:` footer.
- Merging release-ready work into `main` is the release trigger. Keep release
  notes clear enough for automated release tooling and package consumers.
- The root semantic-release config creates GitHub releases and tags, publishes
  the TypeScript package from `typescript/` to npm, then dispatches
  `.github/workflows/release.yml` for Python only when semantic-release creates
  a real release. Release-triggering types are `feat`, `fix`, `perf`, `revert`,
  `bump`, and breaking changes. Non-release types such as `docs`, `chore`, and
  `ci` do not dispatch package publishing.
- TypeScript npm publishing is automated with `@semantic-release/npm` and
  `pkgRoot: "typescript"`. The release job installs and builds the TypeScript
  package before running semantic-release. Configure npm trusted publishing for
  `itsperini/viscribe` using `.github/workflows/ci.yml`, or provide an
  `NPM_TOKEN` repository secret with publish access.
- Python PyPI publishing is automated through `.github/workflows/release.yml`,
  which accepts the semantic-release version as a manual dispatch input. The
  canonical `itsperini/viscribe` repo applies that version to `python/`,
  mirrors its tracked tree into `ViscribeAI/python-sdk` with `git archive`, then
  dispatches that repo's `.github/workflows/release.yml`. The mirrored
  `python-sdk` repo publishes the package from `python/` via PyPI trusted
  publishing.
- Keep the Python release workflow path exactly
  `.github/workflows/release.yml` and keep the old repo name
  `ViscribeAI/python-sdk`; these are part of the trusted-publishing identity.
- Keep canonical CI and CodeQL jobs scoped to `itsperini/viscribe`. The
  mirrored `ViscribeAI/python-sdk` repo should run the Python publishing jobs in
  `.github/workflows/release.yml`, not canonical semantic-release or CodeQL.
- The source repo needs a repository Actions secret named
  `PYTHON_SDK_RELEASE_PAT`. It must be a fine-grained GitHub PAT that can push
  to and dispatch workflows in `ViscribeAI/python-sdk`: repository access only
  to `ViscribeAI/python-sdk`, `Contents` read/write, `Actions` read/write,
  `Workflows` read/write, and default `Metadata` read. Never commit or print the
  token value.
- Python and TypeScript published package versions must match the
  semantic-release version. Before merging a release-triggering PR, make sure
  both `python/**` and `typescript/**` have corresponding implementation,
  tests, examples, or docs changes. If one SDK is intentionally unchanged,
  document the reason clearly in the PR.
- Before opening or updating a PR, run the checks that match the touched
  package. Prefer full package verification for public API, packaging, release,
  or cross-language changes.
- Community-facing docs may invite users to star the repo, follow
  `https://x.com/itsperini`, connect on
  `https://www.linkedin.com/in/itsperini`, and join the Discord community at
  `https://discord.gg/uJN7TYcp`.

## Licensing Notes

- The repository uses the MIT license.
- Preserve license notices when moving, copying, or substantially modifying
  third-party code.

## Shared Agent Setup

- `.agents/AGENTS.md` is the canonical root guide.
- `.agents/README.md` documents the shared agent configuration.
- `.agents/skills/` contains reusable workflows such as `grill-me` and
  `skill-creator`.
- When creating or editing shared skills, follow
  `.agents/skills/skill-creator/SKILL.md` and keep skills concise with
  progressive disclosure.

## Git and Tooling Notes

- Use `rg` or `rg --files` for searching when available.
- Avoid destructive git commands such as `reset --hard` unless the user
  explicitly requested them.
- Do not revert unrelated working-tree changes.
- Keep commits focused and describe verification performed in PR notes.

---
> Source: [itsperini/viscribe](https://github.com/itsperini/viscribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
