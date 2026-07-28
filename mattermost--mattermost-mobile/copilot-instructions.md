## mattermost-mobile

> **Read [GUIDELINES.md](./GUIDELINES.md) before editing any flow.** It is the single authoring spec (header contract, selectors, gotchas, CI, checklist).

# Maestro — Agent entrypoint

**Read [GUIDELINES.md](./GUIDELINES.md) before editing any flow.** It is the single authoring spec (header contract, selectors, gotchas, CI, checklist).

Quick rules:

- Use `id:` testIDs only — never `point:` except system overlays with no accessibility node.
- Every flow needs the header block in GUIDELINES §2; run `cd detox/maestro && npm run validate-headers`.
- Start authenticated flows with `runFlow: ../../subflows/auth/login.yml`.
- Setup and CI: [README.md](./README.md).

Do not duplicate rules here — update GUIDELINES.md instead.

---
> Source: [mattermost/mattermost-mobile](https://github.com/mattermost/mattermost-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
