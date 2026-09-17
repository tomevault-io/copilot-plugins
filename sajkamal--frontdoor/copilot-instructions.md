## frontdoor

> Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.

# Prototype Instructions

Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.

Before making substantial visual changes, use the Product Design plugin's `get-context` skill when the visual source is unclear or no longer matches the current goal. When the user gives durable prototype-specific design feedback, preferences, or decisions, record them in `AGENTS.md`.

When implementing from a selected generated mock, treat that image as the source of truth for layout, component anatomy, density, spacing, color, typography, visible content, and hierarchy.

Build app UI in `src/`. Keep `.openai/hosting.json`, `worker/index.js`, `scripts/prepare-sites-build.mjs`, and `tests/sites-worker.test.mjs` intact so the same local prototype can be handed to Sites. Before a Sites handoff, run `npm run build` and `npm run test:sites`; the build must leave `dist/client/index.html`, `dist/server/index.js`, and `dist/.openai/hosting.json`.

## Prototype decisions

- This is a fictional demonstration practice named Dr Rick Jones Urology. Never use a real clinician's identity, likeness, address, availability, patient information, or contact details.
- Use the selected Guided Care Map direction: warm white and mineral-grey surfaces, deep navy type, surgical teal actions, eucalyptus accents, and restrained cobalt highlights.
- The experience should feel clinical, trustworthy, calm, specialist-led, and rich rather than sterile or corporate.
- Prefer an editorial private-practice feel: real original imagery, larger readable type, fewer repeated card grids, generous whitespace and distinct visual chapters.
- Core journeys are condition education, interactive 3D anatomy and procedure visualisation, doctor profile and credentials, specialties, clinic locations and schedules, appointment preparation, and referral enquiry preparation.
- Appointment and referral actions are demonstration-only review handoffs. They must never submit, book, transmit, or persist personal or clinical information.

---
> Source: [sajkamal/frontdoor](https://github.com/sajkamal/frontdoor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
