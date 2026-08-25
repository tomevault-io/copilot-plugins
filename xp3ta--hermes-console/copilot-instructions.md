## hermes-console

> Read `docs/PROJECT_BRIEF.md`, `docs/SECURITY_POLICY.md` and

# Agent release boundaries

Read `docs/PROJECT_BRIEF.md`, `docs/SECURITY_POLICY.md` and
`docs/RELEASE_DISTRIBUTION.md` before changing release or distribution files.

## Non-negotiable channel separation

- Google Play accepts only the signed `play` AAB. Never commit it, attach it to
  a GitHub Release, or upload it as a public workflow artifact.
- GitHub Releases and Obtainium accept only signed `full` release APKs. Never
  publish `play`, `qa`, `debug` or `profile` artifacts there.
- The `qa` flavor is internal physical-test evidence and must never be public.
- Never commit or expose `key.properties`, keystores, passwords, tokens,
  mappings, diagnostic bundles, pairing payloads or private server details.
- Build outputs stay ignored. A request to build does not authorize upload,
  publication, tagging, repository visibility changes or Play submission.
- Every publication action requires explicit owner approval for that exact
  channel and artifact.

Use path-specific staging; never use `git add -A` or a broad recursive stage in
this repository.

---
> Source: [xP3ta/hermes-console](https://github.com/xP3ta/hermes-console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
