## dcbball

> Marketing and registration site for **DSC Hoops** and the **Workhouse Warrior Summer 2026** program.

# DSC Hoops - AGENTS.md

## What This Project Is

Marketing and registration site for **DSC Hoops** and the **Workhouse Warrior Summer 2026** program.

The live site is now the approved React/Vite redesign. The root route `/` and `/summer` both render the summer registration experience. The old static HTML summer page has been removed from the deployed site.

**Live:** https://rivarjaol.github.io/dcbball/
**Repo:** https://github.com/rivarjaOL/dcbball (note the capital `OL` in the org slug)

The old redesign preview repo, `ClariSortAi/dcball-preview`, was the prototype source. The production source now lives in this repo.

---

## Current Stack

| Layer | Tech |
|---|---|
| App | React 18 + Vite |
| Language | TypeScript |
| Routing | `react-router-dom` with routes for `/` and `/summer` |
| Styles | Tailwind CSS, custom tokens in `src/index.css`, shadcn-style UI primitives |
| Icons | `lucide-react` |
| Build | `npm run build` |
| Deployment | GitHub Pages via `.github/workflows/deploy.yml` |
| Registration | In-page React registration packet posts to Google Forms `formResponse` |
| Notifications | Google Apps Script (`form-submit-notification.gs`) triggered by the linked Google response Sheet |

---

## File Structure

```
/
├── index.html                         # Vite HTML shell
├── src/
│   ├── App.tsx                        # Providers and routes
│   ├── main.tsx                       # React entrypoint
│   ├── index.css                      # Tailwind layers, design tokens, global utilities
│   ├── pages/
│   │   ├── Summer.tsx                 # Live landing page and registration page
│   │   └── NotFound.tsx               # Fallback route
│   ├── components/
│   │   ├── RegistrationPacket.tsx     # In-page Google Form compatible intake flow
│   │   └── ui/                        # shadcn-style primitives
│   └── assets/                        # Hero images and Workhouse logo
├── public/                            # Static public assets
├── .github/workflows/deploy.yml       # Builds Vite app and deploys `dist`
├── form-submit-notification.gs        # Google Apps Script email notification handler
├── APPS_SCRIPT_SETUP_INSTRUCTIONS.md  # Sheet trigger setup and test instructions
├── CHANGELOG.md                       # Plain-language client-facing changelog
├── DSC_HOOPS_Registration_Form.pdf    # Legacy printable registration form
├── package.json
├── package-lock.json
├── tailwind.config.ts
└── vite.config.ts
```

---

## How to Run Locally

Install dependencies:

```bash
npm ci
```

Start the dev server:

```bash
npm run dev
```

Open:

```text
http://localhost:8080/
http://localhost:8080/summer
```

The Vite config uses port `8080` by default. If that port is busy, run:

```bash
npm run dev -- --host 127.0.0.1 --port 5173
```

Build production output:

```bash
npm run build
```

Run checks:

```bash
npm run test
npm run lint
```

Current lint status may include shadcn fast-refresh warnings in `src/components/ui/*`; these are warnings, not build blockers.

---

## Deployment

- **Trigger:** push to `main`
- **Workflow:** `.github/workflows/deploy.yml`
- **Pipeline:** checkout, setup Node, `npm ci`, `npm run build`, copy `dist/index.html` to both `dist/404.html` and `dist/summer/index.html`, upload `dist` to GitHub Pages
- **Vite base path:** production builds use `/dcbball/` in `vite.config.ts`
- **SPA routes:** `dist/summer/index.html` serves `/summer` with HTTP 200, and `404.html` remains as a fallback
- **No env vars or secrets are required**

---

## Registration And Intake Wiring

The live site uses an in-page React intake flow in `src/components/RegistrationPacket.tsx`.

Flow:

```text
Website registration packet
  -> Google Forms formResponse endpoint
  -> Google Form response Sheet
  -> Apps Script spreadsheet submit trigger
  -> email notification
```

Important URLs:

- **Summer Google Form public URL:** `https://docs.google.com/forms/d/1tIES8Y_c-Fp957NolfjGFlnlwN67uKyyzPD_ONJNnj8/viewform`
- **Summer Google Form endpoint used by the React intake:** `https://docs.google.com/forms/d/e/1FAIpQLScf7b9ChwsWEaoNOaeRpKzCMW6LctFmk-TXeEe1Z5McnMx2iQ/formResponse`
- **Responses sheet:** `https://docs.google.com/spreadsheets/d/1-_MBTiupAFNOcXvkDpn_FfNhiqAwZoNqULoPw4QFu5o/`
- **Notification emails:** `hoops@dscinternationalgroup.com` (to), `jason.t.rivard@gmail.com` (cc)

The site does **not** write directly to Google Sheets. It submits to the same Google Form that writes to the Sheet. This is what allows the same Apps Script spreadsheet trigger to fire.

The registration packet maps 1-to-1 to the current summer Google Form entry IDs. If the Google Form fields change, update `buildGooglePayload()` in `src/components/RegistrationPacket.tsx` and revalidate the public form metadata before deploying.

The Apps Script still relies on the exact response column header `"Athlete's Name"` when building the email subject.

---

## Design System

The redesign uses a brutalist basketball training style:

- Bone background, ink typography, orange action color
- Large condensed display type through the `Anton` font
- Monospace utility labels through `Chakra Petch`
- Tailwind custom tokens in `tailwind.config.ts` and `src/index.css`
- Heavy borders, offset shadows, grid-paper texture, and high-contrast registration panels
- Section IDs drive navigation: `#program`, `#schedule`, `#pricing`, `#register`

When editing UI, keep the registration packet and landing sections visually consistent with the current `Summer.tsx` and `RegistrationPacket.tsx` patterns.

---

## Safety Notes

- Public marketing site, no authentication
- No API keys or secrets in the frontend
- Registration data is not stored in this repo
- Draft registration data is saved only in browser `sessionStorage` under `workhouse-summer-registration-draft`
- The Google Form endpoint is public by design
- Do not commit real registration exports or response data
- Confirm with the client before changing email recipients, the response Sheet, or Apps Script behavior

---
> Source: [rivarjaOL/dcbball](https://github.com/rivarjaOL/dcbball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
