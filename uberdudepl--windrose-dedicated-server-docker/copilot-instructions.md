## windrose-dedicated-server-docker

> When writing GitHub release notes for a tag:

When writing GitHub release notes for a tag:

- Provide output in two separate markdown code blocks:
  - First block: release title only.
  - Second block: release body only.
- Do not include literal labels like "TOP:" or "DESCRIPTION:" in the output.
- TOP title style must be: `vX.Y.Z — Short headline` (example: `v1.2.3 — Safer waters ahead`).
- The title may be playful and funny in pirate style, as long as it stays clear and readable.
- Keep both blocks copy-paste friendly.
- Output exactly two fenced markdown code blocks, with no extra text before, between, or after them.
- Write in English.
- Use a playful pirate tone, but keep it readable and professional.
- Start with the release name and version.
- Add a short one-sentence summary.
- Use sections like Changes, Notes, Captain's note, and Upgrade.
- Keep technical facts accurate and concrete.
- Add light pirate flavor only to headings, transitions, and short commentary.
- Do not affect code, identifiers, commands, or environment variables.
- Keep the release note short enough to scan quickly.
- Description starts with "## ⚓ Windrose Dedicated Server Docker" and the version number
- Include a concise migration update for Docker runtime scripts, and explicitly reference the canonical paths in `/opt/windrose/scripts`.
- Include the current deprecation status of root compatibility wrappers (`entrypoint.sh`, `healthcheck.sh`) in every release note until those wrappers are fully removed.

Before creating or publishing a new tag:

- Update `IMAGE_TAG` in `.env.example` to the new version.
- Update all stable tag references in `README.md` to the new version (quick start image example, `IMAGE_TAG` default value in config table, `IMAGE_TAG` in the quick start code block, update/stable guidance lines).
- Update `brokol/docker.md` while preparing release notes so the Docker Hub description source stays current.
- Validate that old stable version references are gone from `.env.example` and `README.md`.
- Confirm the release notes include an updated wrapper deprecation status entry when the wrappers still exist or their status has changed.
- Do not include `brokol/docker.md` in release/version-bump commits unless the user explicitly requests it.
- In every release process, remind the user to manually update the Docker Hub description from `brokol/docker.md`.
- Do not automate Docker Hub description updates via CI or workflow automation.
- Commit and push these release-related documentation changes (excluding `brokol/docker.md` unless the user explicitly requests it) to `main` first.
- Only then create and push the release tag.
- If a tag was created too early, move it to the latest `main` commit before publishing release notes.

---
> Source: [UberDudePL/windrose-dedicated-server-docker](https://github.com/UberDudePL/windrose-dedicated-server-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
