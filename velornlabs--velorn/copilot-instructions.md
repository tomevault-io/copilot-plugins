## release-process

> Use the documented Windows and macOS release flow


# Release Process

Before doing release, packaging, tagging, publishing, or release-cleanup work:

- Read `docs/AI_RELEASE_HANDOFF.md`
- Read `docs/RELEASE_PROCESS.md`
- Follow `.github/workflows/release.yml` for Windows and macOS release builds
- Keep secrets only in GitHub Actions repository secrets
- Never commit certificate files, passwords, app-specific passwords, or API keys
- Prefer throwaway test tags when validating release automation

If the release workflow changes, update `docs/RELEASE_PROCESS.md` too.

---
> Source: [VelornLabs/velorn](https://github.com/VelornLabs/velorn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
