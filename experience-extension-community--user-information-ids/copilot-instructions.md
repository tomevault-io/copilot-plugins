## user-information-ids

> Project context for AI assistants working on this codebase. Read by Claude

# CLAUDE.md

Project context for AI assistants working on this codebase. Read by Claude
Code's `claude` CLI automatically; equivalently helpful as a reference for
human contributors.

## What this repo is

A parameterizable community sample of an Ellucian Experience extension that
displays a person's identification credentials, sourced from Ethos Data
Connect. One card + one full-page view, role-aware.

## How to build, lint, test

All from within `extension/`:

```bash
nvm use && npm install
npm run lint
npm test
npm run build-prod
npm run build-dev
npm run deploy-dev     # requires EXPERIENCE_EXTENSION_UPLOAD_TOKEN in .env
```

## Parameterization surface — the rule that matters most

**Every institution-specific value lives in one of these files. Nothing else.**

- `extension/src/config/institution.js`
- `extension/src/config/credentialTypes.js`
- `extension/src/config/roles.js`
- `extension/src/utils/branding/tokens.js`
- `extension/src/utils/fontLoader.js` (reads `institution.typekit`)

When making changes, **respect this surface**:

- Do not hardcode a color hex anywhere outside `tokens.js`.
- Do not hardcode an email domain anywhere outside `institution.js`.
- Do not hardcode a role string outside `roles.js`.
- Do not hardcode the DC pipeline name outside `institution.js`.
- Do not hardcode a credential label or Ethos code outside `credentialTypes.js`.

If a component needs a new institution-specific value, **add it to the surface
first**, then read from it in the component. The README's Configuration
Reference table must be updated in the same PR.

## Conventions enforced by the org

- Apache-2.0 license; SPDX header `// SPDX-License-Identifier: Apache-2.0` on
  every `.js` / `.jsx` source file.
- Conventional Commits.
- DCO sign-off (`git commit -s`).
- Functional React components only. Hooks rules strictly followed.
- Tests: Jest + React Testing Library + jest-axe. Aim for zero axe violations.
- WCAG 2.2 AA accessibility.
- Built from `@ellucian/react-design-system/core` where possible.

See the org's [CONTRIBUTING.md](https://github.com/experience-extension-community/.github/blob/main/CONTRIBUTING.md) for the full list.

## Where to read first

- `_plans/user-information-ids-plan.md` (root of the parent repo) — the
  original plan if it's still around.
- `docs/architecture.md` — data flow and why the parameterization surface is
  split the way it is.
- `docs/adoption-guide.md` — what an adopter has to edit.

## Things to ask Claude to do (and things not to)

**Good asks:**

- "Add a new credential type called X" — edit `credentialTypes.js`, no other
  file should need changes.
- "Switch the focus-ring color" — edit `tokens.js`, that's it.
- "Add a test for the noDataText branch of the card" — extend
  `UserInformationCard.test.jsx`.

**Bad asks (push back on these):**

- "Hardcode a brand color for now and we'll fix it later" — no, edit
  `tokens.js`.
- "Add a card just for our institution" — that breaks the community-sample
  purpose. Build it in a sibling repo.
- "Hardcode an email" — set `studentEmailDomain` / `employeeEmailDomain`
  instead.

---
> Source: [experience-extension-community/user-information-ids](https://github.com/experience-extension-community/user-information-ids) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
