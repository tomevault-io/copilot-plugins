## ferry

> - Local `main` is the public branch, pushed to `origin/main`

# Ferry — Project Instructions

## Git Workflow

- Local `main` is the public branch, pushed to `origin/main`
- **NEVER switch branches or create branches** unless the user explicitly asks. This is a production machine running Dokku + Cloudflare Tunnel — branch switches destroy gitignored runtime config (`config.yml`), causing Docker to create a directory placeholder that takes the tunnel offline. Ship directly from main when on main.
- If already on a feature branch, `/ship` merges it to main. But do not create feature branches unprompted.
- `old-main` and `open-source-prep` are legacy branches — do NOT push these
- **NEVER** commit personal data (real domains, tunnel IDs, account IDs) to main
- Push with `git push origin main` (not `oss:main` — the oss branch was renamed)

## Testing

- Run all tests: `make test` or `make check` (lint + tests)
- Unit: `bats test/unit/`
- Integration: `bats test/integration/`
- Generators: `bats test/generators/all_generators.bats`
- Lint: `make lint`
- Syntax check: `bash -n ferry`

## Version

- Version lives in `ferry` script line 9: `FERRY_VERSION="x.y.z"`
- Changelog: `CHANGELOG.md`
- Always bump version BEFORE doing work if other agents may have made changes

## Skills

### /ship — Pre-release checklist and push

Invoke with `/ship` when ready to release. Works from main (direct push) or from a feature branch (merge to main). **NEVER creates or switches branches.**

1. **Branch check** — note current branch (do NOT switch)
2. **Lint** — `make lint` + `bash -n ferry`
3. **Tests** — all unit, integration, and generator tests (including Docker runtime HTTP 200)
4. **Docs check** — version in ferry matches changelog, no stale refs
5. **Personal data audit** — scan for real domains, tunnel IDs, credentials in staged files
6. **Version bump** — if not already done, bump FERRY_VERSION (patch/minor/major)
7. **Changelog** — add entry to CHANGELOG.md with date from `date -Iseconds`
8. **Commit** — stage specific files, commit with detailed heredoc message
9. **Merge to main** — only if on a feature branch: `git checkout main && git merge --no-ff {branch}`
10. **Tag** — `git tag v{VERSION}`
11. **Push** — `git push origin main && git push origin v{VERSION}`
12. **Cleanup** — optionally delete feature branch (if applicable)

Full skill spec: `.claude/skills/ship/SKILL.md`

**Red flags that STOP the process:**
- **Creating or switching branches** (destroys runtime config on this production machine)
- Personal data in staged files
- Any test failing
- Lint errors
- Version mismatch between ferry and changelog
- Merge conflicts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gastonmorixe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gastonmorixe)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
