## solana-rug

> **This repo is published under a pseudonym.** No real names, email addresses, or

# AGENTS.md — Instructions for AI agents working in this repo

## Identity Rules

**This repo is published under a pseudonym.** No real names, email addresses, or
personally-identifying information may appear anywhere in the code, commits, git
metadata, or documentation.

- Allowed author/committer: `GraphTheory <codegraphtheory@pm.me>`
- Any real-world names or personal emails are strictly forbidden.

## Git Best Practices

### NEVER change the remote URL.

The remote URL contains an embedded GitHub token. Changing it will break push
access. Push directly with `git push origin main --force`.

### Always set BOTH author AND committer on every commit.

Git has two fields per commit: `Author` and `Committer`. Setting only `--author`
leaves the committer as the local git config default, which may contain real
identifying info. Always use both:

```
GIT_COMMITTER_NAME="GraphTheory" \
GIT_COMMITTER_EMAIL="codegraphtheory@pm.me" \
git commit --amend --author="GraphTheory <codegraphtheory@pm.me>" --no-edit
```

### Squash before pushing.

This repo maintains a single-commit history. Before pushing, squash:

```
git reset --soft $(git rev-list --max-parents=0 HEAD)
GIT_COMMITTER_NAME="GraphTheory" \
GIT_COMMITTER_EMAIL="codegraphtheory@pm.me" \
git commit --amend --author="GraphTheory <codegraphtheory@pm.me>" --no-edit
```

### Force push is expected.

The remote branch is force-pushed every time. That's normal for a single-commit
repo. Use `git push origin main --force`.

## CI Workflow

The CI workflow at `.github/workflows/ci.yml` runs on every push:
1. Lint with ruff
2. Type check with mypy
3. Run unit tests (`pytest -k "not slow"`)
4. Build check (`uv build`)

The token used to push must have `workflow` scope. If you get a "refusing to
allow a Personal Access Token to create or update workflow" error, regenerate
the token with `workflow` scope enabled.

## File Cleanliness

- `.gitignore` blocks `__pycache__/`, `*.pyc`, `src/`, `*.egg-info/`, `.venv/`
- Before committing, strip any pycache files: `git rm -r --cached __pycache__/ 2>/dev/null`
- The repo should have ~16 tracked files. Check with `git ls-files | wc -l`

## Release Workflow

A tagged push (`v*.*.*`) triggers `.github/workflows/release.yml`:
1. Builds the package with `uv build`
2. Publishes to PyPI (needs `PYPI_TOKEN` secret)
3. Creates a GitHub Release with the built artifacts

### Required Secrets

Add these in GitHub → Settings → Secrets and variables → Actions:

| Secret | Value | Where to Get It |
|--------|-------|-----------------|
| `PYPI_TOKEN` | `pypi-...` | https://pypi.org/manage/account/token/ — create a token scoped to the `solana-rug` project |
| `CLOUDFLARE_API_TOKEN` | `...` | Cloudflare dashboard → My Profile → API Tokens → Create token with `Cloudflare Pages:Edit` permission |
| `CLOUDFLARE_ACCOUNT_ID` | `...` | Cloudflare dashboard → right sidebar → Account ID |

### Cloudflare Pages Setup

Before the first deploy, create the project in Cloudflare Pages:
1. Go to Cloudflare Dashboard → Pages
2. Create a new project called `solana-rug`
3. Skip the "connect to git" option (we deploy via Actions)
4. Add the two secrets above to GitHub

### How to Release

### How to Release

```bash
# Tag the current HEAD with a version
git tag v0.1.0
git push origin v0.1.0
```

The CI workflow will build, publish to PyPI, and create a GitHub Release automatically.

### Cloudflare Pages

Each push to `main` auto-deploys the `docs/` folder to Cloudflare Pages via
`.github/workflows/deploy-cloudflare.yml`. The site is a single HTML file with
no build step — deploys in seconds.

This project has a support token on PumpSwap:
- Token: `F4J5LKyEQraMem8nspPAzwHXaaKMMDsxyt7GUK94pump` ($RUG)
- DEX: pumpSwap (RUG/SOL pair)
- Chart: https://dexscreener.com/solana/4sHKYieWsGtrmtqjdXPRzSdVywXZ1jUQGbM8QbkBXMB9

---
> Source: [rugpullnet/solana-rug](https://github.com/rugpullnet/solana-rug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
