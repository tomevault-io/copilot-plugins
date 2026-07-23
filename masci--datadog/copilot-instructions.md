## datadog

> TypeScript **GitHub Action** that sends events and metrics to Datadog (`action.yml` + `src/`).

# Agent notes

TypeScript **GitHub Action** that sends events and metrics to Datadog (`action.yml` + `src/`).

- **Build:** `npm run build` (TypeScript → `lib/`)
- **Test:** `npm test`
- **Lint / format:** `npm run lint`, `npm run format-check`

Source lives under `src/`; tests under `__tests__/`.

## Release process

To release a new version of this action:

- Check out the latest version of the `main` branch
- Ask which version number you need to bump: bugfix, minor or major
- Make sure the `HEAD` has no tags yet
- Tag the `HEAD` with the desired semantic version number, prefix the version string with a lowercase letter `v` resulting in the format `vX.Y.Z`
- There is a special tag `vX` that always points to the latest major version. Move it so it points to `main`
- **Stop before pushing.** After tags exist locally, ask the user to confirm they want the tags pushed to the remote (show the exact `git push` commands you will run, including `--force` for `vX` if applicable). Do not run `git push` until they explicitly confirm.

---
> Source: [masci/datadog](https://github.com/masci/datadog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
