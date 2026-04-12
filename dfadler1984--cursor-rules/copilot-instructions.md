## cursor-rules

> Dependency selection policy


# Dependencies

## Dependency selection

- Prefer zero or standard-library solutions first.
- When adding a dependency, choose small, well-maintained libraries with permissive licenses.
- Avoid transitive-heavy, unmaintained, or native-build dependencies unless justified.
- Document rationale for new dependencies in the PR description.

## Security checks

- Run a basic security and license check (e.g., `yarn npm audit --all` or equivalent) before introducing new dependencies.
- Avoid libraries with known critical vulnerabilities unless there is no alternative and a mitigation plan is documented.

## Performance and size

- Consider install size, runtime size, and build impact; prefer lighter alternatives.
- Avoid introducing heavy dependencies for trivial tasks.

## Lockfile policy

- When `package.json` changes, run `yarn install` and commit `yarn.lock` in the same PR.
- Pre-commit hook enforces this: it fails if `package.json` is staged without `yarn.lock`.
- CI uses `--frozen-lockfile`; outdated lockfiles will fail the install step.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dfadler1984)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dfadler1984)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
