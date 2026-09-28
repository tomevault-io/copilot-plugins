## obsidian-oura-plugin

> - Never bundle a shared OAuth client secret or log authorization codes, tokens, credentials, or personal API responses.

# Oura plugin working notes

- Never bundle a shared OAuth client secret or log authorization codes, tokens, credentials, or personal API responses.
- Oura refresh tokens are single-use. Keep refresh serialized per plugin instance and invalidate in-flight refreshes only on disconnect. Cancelling sign-in or editing credentials must not discard rotated refresh tokens. Keep sign-in cancellation separate.
- Authentication failures must reach the command's notice before any note mutation.
- Run `npm test`, `npm run typecheck`, and `npm run build` after authentication changes; these checks are automated in CI. Live provider behavior and visual/device checks are separate evidence.
- Maintain the verification backlog and decisions in `docs/oauth-quality.md`; do not claim browser handoff or mobile support is verified without testing those environments.

- Use npm with the tracked package-lock.json; use `npm ci` in development and CI. Clear refresh tokens only for an explicit invalid_grant response, not a generic HTTP 400/401.

- Before publishing, select an unused version greater than the latest release, update package/lock/manifest/versions metadata together, and run `npm run check:version`. Release CI also verifies the tag matches.

---
> Source: [kinabalu/obsidian-oura-plugin](https://github.com/kinabalu/obsidian-oura-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
