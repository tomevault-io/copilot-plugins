## arabista-site

> This workspace contains a unified omnichannel architecture divided into distinct decoupled systems, managed by a central Master Router:

# ARABISTA PROJECT - AI BEHAVIOR RULES

## Context
This workspace contains a unified omnichannel architecture divided into distinct decoupled systems, managed by a central Master Router:
1. Master Webhook Router (Central point of entry, routing triage, & Command Center)
2. Retail Engine (Direct E-Commerce / Pos Laju / SenangPay)
3. Alteration Service (Gig-Tailor Network / Lalamove / Hub Portal)
4. Support CRM (Telegram 1-on-1 Auto-Topic Triage)
5. Production ERP (BOM / Planning / Procurement / AI Ops Lounge)

## ⚠️ MANDATORY DEVOPS & ARCHITECTURE RULES
- **Single-File Architecture (Google Apps Script):** To bypass Google Apps Script's aggressive caching and ghost-file merging bugs, frontend HTML/UI must be embedded directly as string-returning functions (e.g., `getDashboardUI()`) inside the main `.gs` file. Do NOT create separate `.html` files or use global `const` variables for HTML templates to avoid redeclaration errors.
- **Telegram CRM Standard:** All outbound bot messages to customers from a Telegram CRM environment MUST require the `/c` command prefix to enforce muscle-memory consistency across the team.

## ⚠️ MANDATORY DOCUMENTATION RULE
We maintain three single-source-of-truth architecture documents in the root folder:
- `Arabista_Retail_Master_Doc.md` — storefront, checkout, Retail GAS, Master Router
- `Arabista_Alteration_Master_Doc.md` — alteration / hub / gig network
- `Arabista_Production_Master_Doc.md` — Production ERP (`production-dashboard.html`, `production-app.js`, `Production_Engine.gs`)

Whenever you (the AI) are asked to write new code, modify logic, alter an API payload, change a URL, or adjust the database schema, you MUST adhere to the following rule:
You must proactively update the corresponding Master Document (.md file) to reflect the new changes so the documentation never drifts from the live codebase. Maintain the existing Markdown hierarchy, structure, and tone when making your updates. Do not ask for permission to update the doc; just include the document updates in your proposed code changes. (Retail: `Arabista_Retail_Master_Doc.md` **v3.4**+; Production: `Arabista_Production_Master_Doc.md` **v5.1**+.)

## ⚠️ FRONTEND MASTER TEMPLATE PROTOCOL
To prevent code drift and maintain a unified architecture, we strictly enforce a Master Template system for all product pages.

**1. The Master Template**  
`product-z01-staging.html` is the absolute **Master Reference Template**. Whenever you are asked to create or update a product page (e.g., `product-m01-staging.html`, `product-d02-staging.html`), you MUST strictly clone the HTML layout, CSS classes, and JavaScript logic of the Master Template.

**2. Allowed Modifications for New Products**  
When cloning the Master Template for a new product, you are ONLY permitted to change the following data points:
* Model codes (e.g., `Z01` to `M01`) and Series Names (e.g., `Zahra` to `Maraya`).
* Image file paths and the CSV Review data payload.
* Alteration ID keys (e.g., `Z01_ALT_LENGTH` to `M01_ALT_LENGTH`).
* **THE DAHLIA EXCEPTION:** If the product belongs to the "Dahlia" series (e.g., D01, D02), you MUST change the feature bullet wording from `"Cutting Ala Cardigan"` to `"Cutting 1-Lapis"`. For all other series (Zahra, Maraya, etc.), it must remain `"Cutting Ala Cardigan"`. Do not alter any other structural HTML.

**3. Staging vs. Production Synchronization**  
Staging files (e.g., `product-z01-staging.html`) and Production files (e.g., `product-z01.html`) MUST remain structurally, logically, and aesthetically 100% identical at all times.  
The ONLY permitted differences between a staging file and its production counterpart are environment variables:
1. **Title Tag:** Staging includes `(STAGING)`.
2. **Internal Routing:** Staging uses relative paths (e.g., `href="index-staging.html"`). Production uses live root paths (e.g., `href="/"`).
3. **API URL:** Staging connects to the Staging Google Apps Script URL. Production connects to the Live Apps Script URL.  
Never introduce a layout or logic change to Production without also mirroring it in Staging (and vice versa).

## 🛒 OMNICHANNEL CART PORTING PROTOCOL (V2)
All product pages MUST use the unified `localStorage` Cart Engine. `product-z01-staging.html` is the Master Template. When creating or updating a product page, you MUST adhere to the following strict porting rules:

**1. No In-Memory State:** NEVER use `let cartState = null;`. The cart MUST be globally driven by the `CartManager` object interacting with `localStorage.getItem('arabista_cart')`.

**2. Universal Engine Parity:**
The following JavaScript functions MUST be exact 1:1 copies from the Z01 Master Template. Do NOT alter their internal logic per page:
- `CartManager` object (including the `nav-cart-badge` UI animation)
- `renderCart()`
- `updateGrandTotal()`
- The `#btn-calc-ship` and `#btn-checkout` listeners (which execute "Hard Reject" reloads and dynamic array building for Google Apps Script).

**3. Product Identity Mapping:**
When building the `add[MODEL]ToLocalCartAndOpenDrawer()` payload, you MUST map the following properties to match the specific HTML page:
- `id`: Must prefix with the model (e.g., `D01-${Date.now()}`).
- `series`: The correct product family (e.g., 'Dian Series').
- `image`: The correct hero thumbnail path for the cart drawer.
- `alterations`: Ensure the `appConfig` keys match the exact model (e.g., `D01_ALT_LENGTH`).
- Ensure `unitWeight: activeWeightKg` is included in the payload.

**4. UI Strictness:**
- Do NOT use the legacy `#cart-notification` red banner for sold-out states. Rely on the Hard Reject `alert()` + `window.location.reload()` built into the Master checkout logic.

## ⚠️ REVIEW INGESTION PROTOCOL
Whenever you are instructed to inject customer reviews from a CSV file into a product staging page, you MUST strictly follow this exact 5-step process to prevent syntax errors and maintain data integrity:

**1. Column Mapping:**  
Read the provided CSV and map the data to an array of JSON objects using EXACTLY these 4 keys: `username`, `size`, `rating`, `original`.
* `username` <- Mapped from the `Reviewer` column.
* `rating` <- Mapped from the `Rating` column (parsed as an integer).
* `original` <- Mapped from the `Text` column.
* `size` <- Mapped from the `Variant` column.

**2. Strict Data Sanitization (Syntax Error Prevention):**
* **The Newline Rule:** You MUST replace ANY literal newlines (`\n` or `\r`) inside the `Text` column with a single space. The entire review string must sit on a single line to prevent JavaScript `Uncaught SyntaxError` crashes.
* **The Size Extraction Rule:** You MUST extract ONLY the size string from the `Variant` column. Strip out any extra product details. (e.g., `"No Shawl, S"` becomes `"S"`; `"Shawl, XXL"` becomes `"XXL"`; `"XS"` remains `"XS"`).

**3. JSON Injection Formatting:**  
Locate `const FULL = [...];` inside the `// --- Review Grid Logic ---` section of the HTML file. Replace it entirely with the newly mapped array.
* **Format:** The injected JSON MUST be formatted as a minified, single-line string or tightly packed array to guarantee zero line-break syntax errors.

**4. Sync HTML Review Counts:**  
Calculate the exact total number of reviews injected (the length of the `FULL` array) and update the hardcoded numbers in the HTML to match:
* Update the Header count: `<span class="text-white/50 text-[10px] uppercase tracking-widest">{TOTAL} Reviews</span>`
* Update the Grid count: `<div class="text-gray-400 text-sm">{TOTAL} reviews</div>`

**5. Verification:**  
Do NOT alter the `reviewCard(r)` function or any surrounding layout/logic during this process.

---
> Source: [aqmalsalleh/arabista-site](https://github.com/aqmalsalleh/arabista-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
