## ofa-calculator-embed

> Hey future-me / Cursor agent 👋

# Cursor Project Context — OFA Calculator

Hey future-me / Cursor agent 👋  
This repo is for a **client-facing, embeddable, two-form lead calculator** for **High Performance Providers** (HPP). It's intended to be dropped into sites like **Go High Level** / **HubSpot** / **WordPress** using a tiny `<script>` tag. The embed should **not** expose private API keys and should be configurable from the server.

This file is here so the agent has the full story and doesn’t “optimize” away decisions we already made.

---

## 1. What we’re building

We are building **two separate widgets** (two different embeds):

1. **Impact Analysis (Widget A)**  
   - Input: `# of employees`, optional location, CRM fields  
   - Math is config-driven  
   - Output is a set of opioid/dependence risk indicators for an employer group  
   - On submit → **Go High Level** (GHL) **and** our **Referral Tool** (with referral cookie)

2. **Return-on-Community (Widget B)**  
   - Input: `state`, `county` → try API lookup for population → fallback to manual population  
   - Uses the same CRM fields as Impact  
   - Output is counts/indicators, **not** dollar savings  
   - On submit → **Go High Level** and **Referral Tool**

**Client requirement:** HPP’s marketing team must be able to embed **either one** independently on different pages. That’s why we have **two loader scripts**.

---

## 2. High-level architecture

**Pattern:** host-page script → iframe → React app → server routes

- **Loader scripts (UMD):**
  - `/cdn/leadcalc-impact.min.js`
  - `/cdn/leadcalc-community.min.js`
  - These are built with **Rollup** and live in `public/cdn/` so Next.js can serve them.
  - They expose **one global**: `window.OFACalculator.init(...)`

- **Iframe apps (React, Next.js App Router):**
  - `/embed/impact`
  - `/embed/community`
  - These pages run in an iframe, render the forms, and use `postMessage` back to the parent for:
    - bootstrapping (API base, theme, config version, referral token)
    - dynamic height (no scrollbars on host page)

- **API routes (Next.js):**
  - `GET /api/config?version=...&form=impact|community` → returns math constants
  - `GET /api/lookup/population?state=..&county=..` → TODO: call real Census; right now returns `population: null`
  - `POST /api/submit/impact`
  - `POST /api/submit/community`
  - These 2 POST endpoints are where we will integrate **Go High Level** and the **Referral Tool** securely (server-side secrets).

- **Hosting target:** **Vercel**  
  Next.js → Vercel; static loader bundles live in `public/cdn/*` so they’re served from the same domain.

---

## 3. Local dev setup (current)

Run:

```bash
npm install
npm run dev
# http://localhost:3000
# http://localhost:3000/demo
```

- `npm run dev` runs **Next.js** **and** two **Rollup watchers** (for the two loaders).
- The demo page (`/demo`) simulates a customer website. It loads:

```html
<script src="/cdn/leadcalc-impact.min.js"></script>
<div id="impact-mount"></div>
<script>
  OFACalculator.init('impact-mount', {
    apiBase: window.location.origin,
    iframeBase: window.location.origin,
    configVersion: 'dev',
    theme: 'light',
    referralCookie: 'referral'
  });
</script>
```

If `/cdn/leadcalc-impact.min.js` 404s → Rollup didn’t build. See section 5.

---

## 4. Important files / dirs

- **app/embed/impact/page.tsx**  
  React UI for the employer calculator.  
  - Sends `OFA_CALCULATOR_READY`  
  - Waits for `OFA_CALCULATOR_BOOT`  
  - Renders the form, runs the math, then posts to `/api/submit/impact`

- **app/embed/community/page.tsx**  
  React UI for the county/community calculator.  
  - Same postMessage pattern  
  - Calls `/api/lookup/population` (currently stubbed)

- **app/api/config/route.ts**  
  Returns the canonical math config (see §6). This is because the client said: **all math constants must be configurable via API**.

- **app/api/submit/impact/route.ts** and **app/api/submit/community/route.ts**  
  Stubs right now. This is where we will add the real **GHL** and **Referral Tool** calls.

- **packages/loader-impact/src/index.ts** and **packages/loader-community/src/index.ts**  
  The UMD loaders. These:
  - read the **referral cookie** on the host page,
  - create the **iframe** pointing to `/embed/impact` or `/embed/community`,
  - `postMessage` a boot payload into the iframe,
  - listen for resize events from the iframe.

- **public/cdn/**  
  Where Rollup writes built loader bundles. Next.js serves this statically → becomes `/cdn/...`.

---

## 5. Rollup build notes (we hit this already)

We saw:

> RollupError: Could not resolve entry module "src/index.ts".

This happened because the Rollup config was using a **relative** input and we were running Rollup from the repo root.

**Fix we applied:**

1. Installed the TS plugin:

```bash
npm i -D @rollup/plugin-typescript
```

2. Updated **both** Rollup configs to use **absolute paths** and **TS plugin**:

```js
import path from 'node:path';
import { fileURLToPath } from 'node:url';
import terser from '@rollup/plugin-terser';
import typescript from '@rollup/plugin-typescript';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default {
  input: path.join(__dirname, 'src/index.ts'),
  output: {
    file: path.join(__dirname, '../../public/cdn/leadcalc-impact.min.js'),
    format: 'umd',
    name: 'OFACalculator',
    sourcemap: true
  },
  plugins: [
    typescript({ tsconfig: path.join(__dirname, '../../tsconfig.json') }),
    terser()
  ]
};
```

And the same for **packages/loader-community/rollup.config.mjs**, except the output file is:

```text
../../public/cdn/leadcalc-community.min.js
```

**We also switched plugins:**

- ❌ rollup-plugin-terser@7 (wanted Rollup 2)
- ✅ @rollup/plugin-terser@^0.4.4 (works with Rollup 4)

After this, `npm run dev` should produce:

- `public/cdn/leadcalc-impact.min.js`
- `public/cdn/leadcalc-community.min.js`

Then `/demo` should **not** 404.

---

## 6. Math model (do not hard-code in loaders)

Client wants to be able to tweak the assumptions later. That means loaders and iframes **should not** freeze the numbers. Source of truth is `/api/config`.

Here's an example from a real impact analysis that the client delivered recently for Palm Beach Employee Health Plan:

INPUT: Employees Members: 25,000
MATH CASCADING LOGIC:
 - Members with Rx (rx_rate) = 25,000 * 0.5 = 12,500
 - Members with Opioid Rx = 12,500 * 0.24 = 3,000 (this example they tweaked the opioid_rx_rate from 0.2 to 0.24)
 - Cost/member with ORx = $7,500 - ballpark idea / hardcoded
 - Net cost/member with ORx = $4,000 - TODO: need to learn from client what this is
 - Average Medical/Rx Claim cost / member = $3,500
 - Average Care Managed / ORx Claim Cost = $4,500 ($7,500 from Cost/member with ORx - $4,500 Average Care Managed = $3,000 savings)
OUTPUT - "Financial Impact of Opioids"
 - Net cost/member with ORx * Members with ORx ($4,000 * 3,000) = $12,000,000
OUTPUT - "Targeted Savings"
 - The "savings" in Average Care Managed * 3,000 = $9,000,000
OUTPUT - "Targeted Savings %"
 - Targeted Savings / Financial Impact (12,000,000 / 9,000,000) = 75%


Current defaults:

```json
{
  "impact_analysis": {
    "avg_dependents_per_employee": 2.5,
    "rx_rate": 0.5,
    "opioid_rx_rate": 0.2,
    "at_risk_rate": 0.3,
    "prescriber_non_cdc_rate": 1.4,
    "avg_med_claim_usd": 4000
  },
  "return_on_community": {
    "rx_rate": 0.5,
    "opioid_rx_rate": 0.2,
    "at_risk_rate": 0.3,
    "prescriber_non_cdc_rate": 1.4
  }
}
```

Also from the scope:  
**“Average Medical Claim at $4,000. Please add to the scope that we will identify final actual cost here during the project.”**  
→ so this is expected to change and the API is the right place.

---

## 7. Integrations (to be implemented)

### 7.1 Go High Level (GHL)

To be implemented in **server** routes:

- `app/api/submit/impact/route.ts`
- `app/api/submit/community/route.ts`

What to send:

- Standard contact:
  - first_name
  - last_name
  - email
  - phone
  - company
  - title
- Custom fields:
  - `form_type` → `"impact"` or `"community"`
  - `input_employees` or `county_population`
  - `rx_count`, `orx_count`, `at_risk_count`, `prescribers_identified`
  - `referral_token`
  - `calc_version`
- Tags:
  - `leadcalc_impact`
  - `leadcalc_community`

### 7.2 Referral Tool

Also server-side, same routes:

- POST on submit with:
  - referral token
  - form type
  - timestamp
  - (optional) GHL contact id

We keep this server-side to avoid leaking tokens.

---

## 8. Security / embed decisions

These are **on purpose** — don’t undo them:

1. **Use iframe** to isolate CSS/JS and work well inside GHL/HubSpot.
2. **Read referral cookie in the loader** (host page), not in the iframe.
3. **Pass referral to iframe via postMessage**, then API calls include it.
4. **No secrets in client** — GHL + referral tool calls happen in API routes.
5. **Later**: restrict postMessage by origin when domain is final.

---

## 9. Deployment (Vercel)

After deployment to Vercel, static files are available at:

```text
https://YOUR_DOMAIN/cdn/leadcalc-impact.min.js
https://YOUR_DOMAIN/cdn/leadcalc-community.min.js
```

**Impact embed (final form):**

```html
<script src="https://YOUR_DOMAIN/cdn/leadcalc-impact.min.js"></script>
<div id="ofa-impact"></div>
<script>
  OFACalculator.init('ofa-impact', {
    apiBase: 'https://YOUR_DOMAIN',
    iframeBase: 'https://YOUR_DOMAIN',
    configVersion: '1.0.0',
    theme: 'light',
    referralCookie: 'referral'
  });
</script>
```

**Community embed (final form):**

```html
<script src="https://YOUR_DOMAIN/cdn/leadcalc-community.min.js"></script>
<div id="ofa-community"></div>
<script>
  OFACalculator.init('ofa-community', {
    apiBase: 'https://YOUR_DOMAIN',
    iframeBase: 'https://YOUR_DOMAIN',
    configVersion: '1.0.0',
    theme: 'light',
    referralCookie: 'referral'
  });
</script>
```

---

## 10. TODOs for Cursor agent

1. Implement real GHL calls in both submit routes (server-side).
2. Implement Referral Tool POST in both submit routes.
3. Implement real population lookup (Census, cached).
4. Harden the loader’s postMessage origin checks.
5. Remove the `experimental.appDir` warning in `next.config.mjs`.
6. Improve iframe styling (inputs, spacing, headings).
7. Add UAT test cases that match the scope doc.

---

## 11. Do NOT refactor these away

- Two separate loader scripts
- postMessage handshake (`OFA_CALCULATOR_READY` → `OFA_CALCULATOR_BOOT`)
- Host-page cookie read
- Server-driven math config
- API submit endpoints

---

## 12. Scope background

- Fixed fee: **$2,500**
  - $1,000 start
  - $1,000 on “embed delivered”
  - $500 on completion
- No admin UI in v1
- Target before Thanksgiving
- Biggest lift: Return-on-Community API calls + GHL integration

---

## 13. Style notes

- User is technical → don’t over-explain React
- Prefer explicit file edits
- Keep secrets server-side
- If in doubt: iframe > script-only embed

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lxndrtsh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
