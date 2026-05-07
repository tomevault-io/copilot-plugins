## release-process

> Release process for this repo — tag-driven, same steps every time


# Release process (aport-agent-guardrails)

- **Releases are not automatic on merge to main.** A release happens only when a **tag** `v*` is pushed.
- **Process (repeat every release):**
  1. Bump `version` in **package.json** and update **CHANGELOG.md** (version section + date). Merge to `main`.
  2. From local `main`: `git pull origin main`, then `git tag vX.Y.Z` (tag must match `package.json` version), then `git push origin vX.Y.Z`.
  3. CI runs: [Release workflow](.github/workflows/release.yml) publishes to npm and creates the GitHub Release.
- **Do not** create the release in the GitHub UI — the workflow creates it from the tag. Do not prompt the user to "create a release" manually; direct them to tag and push.
- Reference: [RELEASE.md](RELEASE.md) and [PUBLISHING.md](PUBLISHING.md) at repo root.

---
> Source: [aporthq/aport-agent-guardrails](https://github.com/aporthq/aport-agent-guardrails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
