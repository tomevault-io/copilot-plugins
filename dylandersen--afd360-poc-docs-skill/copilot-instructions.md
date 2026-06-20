## afd360-poc-docs-skill

> Scaffolds a tailored Fumadocs documentation site for a Salesforce Solutions Engineer delivering a post-sale Agentforce, Salesforce, or Data 360 Proof-of-Concept. Use when the user runs /setup-docs, /update-docs, asks to "spin up customer docs", "create POC docs site", "scaffold Fumadocs for a customer", or "add a section to my POC docs". Generates a Next.js + Fumadocs site with sections for Overview, Architecture, Setup, Data Model, Agents & Flows, Handoff, and Troubleshooting, pre-populated with customer and POC details, and supports incremental updates to existing scaffolded sites.


# afd360-poc-docs-skill — Customer POC Documentation Site

Scaffolds a production-ready [Fumadocs](https://fumadocs.dev) site tailored for an SE handing off a Salesforce / Agentforce / Data 360 Proof-of-Concept to a customer. Also supports incremental updates to an already-scaffolded site.

## What it produces

A Next.js + Fumadocs site at a directory the SE chooses, with:

- Opinionated sidebar structure tuned for a POC handoff
- Pages pre-filled with customer + POC context and **starter content** (real tables, ASCII system/ERD sketches, example callouts) the SE can edit in place. Mermaid is intentionally **not** used in starter content because Fumadocs ships without a Mermaid renderer — SEs should drop a PNG/SVG into `public/` if they want a richer diagram.
- A single `site.config.ts` file that centralizes every knob an SE typically wants to change (customer, POC, theme colors, fonts, logo, personas, integrations, repo URL)
- **Auto-extracted customer branding.** If the SE provides the customer's public website during intake, the scaffolder runs `scripts/brand-extractor/index.mjs` against that URL and bakes the logo, favicon, OG hero, primary/secondary colors, and detected font family (with a Google-Fonts neighbor suggested for proprietary fonts) into `site.config.ts` + `public/brand/` + `data/brand-snapshot.json`. Respects `robots.txt`; always surfaces what was extracted for SE review; every value is overrideable in `site.config.ts`.
- Static client-side search (Orama), `llms-full.txt`, OG image route
- Tailwind v4, TypeScript strict, works with `pnpm` (preferred), `npm`, or `bun`

The template tracks **Fumadocs 16+** and requires **Node 22+**. The template's `package.json` is the source of truth for versions — do not hardcode versions anywhere in this skill's prose.

> **Status:** All scripts are implemented and the template passes `next build`. If `scripts/preflight.mjs` or `scripts/verify-placeholders.mjs` are ever missing on disk, **degrade gracefully** — for preflight, fall back to a bare `node --version` check (require ≥ 22) and confirm at least one of `pnpm`/`npm`/`bun` is on PATH; for verify-placeholders, do a `grep -rE '__[A-Z][A-Z0-9_]*__'` over the target. Do not bail or refuse intake — the SE shouldn't pay for a tooling gap.

## Invocation

Triggered by:

- `/setup-docs` — scaffold a new site (default)
- `/update-docs` — add or update sections in an existing scaffolded site
- Natural phrases: "spin up customer docs", "scaffold POC docs", "create docs site for \<customer\>", "add a \<section\> page to my POC docs"

## Workflow

Follow these phases **in order**. Phase 0 is non-negotiable — it catches 90% of "why didn't it work" issues before the SE has invested any time. Phase 1.5 is new: between intake and scaffold, offer the optional section library and let the SE skip anything they don't have answers for yet.

### Phase 0 — Preflight (always run first)

Run `node "<skill-dir>/scripts/preflight.mjs"`. It checks, in order:

1. Node version ≥ 22 (Fumadocs 16 minimum) — **hard requirement**
2. A package manager is available (`pnpm` preferred, `npm` fallback, `bun` acceptable) — **hard requirement**
3. `git` is on PATH (needed for Phase 4's `git init`) — **soft warning, do not block**
4. Network reachability to the npm registry (`https://registry.npmjs.org`) — **soft warning, do not block**

The script exits **0** on hard-requirement pass (with or without soft warnings) and **1** on hard-requirement fail. If it exits 1, **stop and print the script's `fix:` block** for the SE — it includes platform-specific install commands. Soft warnings should be surfaced but **do not stop intake** — the SE's network may flake or git may genuinely not be needed yet.

If `scripts/preflight.mjs` is missing for any reason (corrupted install, partial sync), **fall back to a manual check**: confirm `node --version` returns ≥ v22 and that `pnpm --version`, `npm --version`, or `bun --version` succeeds. **Never** stop intake just because the helper script is absent — the SE has not done anything wrong.

Resolve `<skill-dir>` from the absolute path of this `SKILL.md` file. Typical locations: `~/.cursor/skills/afd360-poc-docs-skill/`, `~/.claude/skills/afd360-poc-docs-skill/`, or `~/.agents/skills/afd360-poc-docs-skill/`.

### Phase 1 — Intake

**Required fields (4):**

1. **Customer name** — e.g. "Acme Corp"
2. **POC name** — e.g. "Service Cloud Agent POC"
3. **Product area** — one of: `Agentforce`, `Data 360`, `Agentforce + Data 360`, `Salesforce Platform`
4. **Target directory** — absolute path. Default: `~/Documents/cursor_orgs/<customer-slug>-docs`

**Optional fields (fill in after scaffold by editing `site.config.ts`):**

5. Primary personas — defaults to `Admin, End User`
6. Key integrations — defaults to `None`
7. Deploy target — defaults to `Not decided`
8. Repo URL — defaults to empty
9. SE name — defaults to the OS user (`os.userInfo().username`)
10. **Customer's public website** — URL. If provided, the scaffolder auto-extracts brand (logo, colors, font, favicon, OG hero). Empty → skip. Auto-prepend `https://` if scheme missing.

Ask for the 4 required fields one at a time. **Do not ask optional fields unless the SE volunteers them.** At the end, offer: "I've got what I need. Want to set personas / integrations / deploy target / repo / your name / **customer website** now, or fill those in later via `site.config.ts`?" The customer-website field is the single highest-impact optional — always mention it in the summary prompt.

Derive slugs from names: lowercase, hyphenate, strip punctuation. Confirm all values back in one compact summary before scaffolding.

Speak directly to the SE in second person. Keep the tone conversational.

### Phase 1.5 — Optional sections (skippable)

Between intake and scaffold, offer the optional section library:

```bash
node "<skill-dir>/scripts/scaffold.mjs" --list-sections
```

Present the slugs as a multi-select with one-line descriptions and ask:

> "Want any of these seeded now? Skip anything you don't have answers for yet — `/update-docs add-section <slug>` adds them later."

**Default behavior is to skip.** If the SE is unsure or short on time, do not push — these pages are easy to add later via `/update-docs`. Common picks for handoff: `security`, `faq`, `glossary`. Common picks for stakeholder demos: `demo-script`. Anything cost-related: `cost-model`.

If the SE picks any sections, pass them comma-separated to scaffold:

```bash
node "<skill-dir>/scripts/scaffold.mjs" --target ... --include-sections "security,faq,glossary"
```

#### Intake anti-duplication rules

- Ask only for the next unanswered required field.
- Never repeat the same prompt text after the user answers.
- Never restate the user's previous answer in the same message as the next question.
- If a field is already known from context (e.g., SE mentioned "Acme" in their first message), skip it and confirm it in the summary.
- Keep acknowledgments short: "Got it." Then ask the next question.
- If the current tool interface supports single-question prompts (like `AskQuestion`), submit one at a time — never a multi-field form.
- Do not echo the full summary until every required field is collected.

### Phase 2 — Scaffold

Call the Node scaffold script. Never re-implement copy/substitute logic inline.

```bash
node "<skill-dir>/scripts/scaffold.mjs" \
  --target "<ABSOLUTE_TARGET_DIR>" \
  --customer "<CUSTOMER_NAME>" \
  --customer-slug "<CUSTOMER_SLUG>" \
  --poc "<POC_NAME>" \
  --poc-slug "<POC_SLUG>" \
  --product-area "<PRODUCT_AREA>" \
  --personas "<PERSONAS_OR_DEFAULT>" \
  --integrations "<INTEGRATIONS_OR_DEFAULT>" \
  --deploy-target "<DEPLOY_TARGET_OR_DEFAULT>" \
  --repo-url "<REPO_URL_OR_EMPTY>" \
  --se-name "<SE_NAME_OR_OS_USER>" \
  [--customer-url "<CUSTOMER_WEBSITE_URL>"] \
  [--no-brand-extract] \
  [--brand-snapshot "<ABS_PATH_TO_EXISTING_SNAPSHOT>"]
```

Resolve `<skill-dir>` from the absolute path of this `SKILL.md` file.

When `--customer-url` is provided, the scaffolder invokes `scripts/brand-extractor/index.mjs` as a subprocess **before** copying the template. The extractor writes `<target>/data/brand-snapshot.json` and downloads assets into `<target>/public/brand/`. The scaffolder then emits matching theme placeholders into `site.config.ts`.

Pass `--no-brand-extract` to keep the customer URL in `site.config.ts` but skip the network calls (useful for offline / air-gapped machines, or when the SE already ran the extractor manually).

Pass `--brand-snapshot <path>` to reuse an existing snapshot (e.g. generated in a previous run or hand-edited). The scaffolder copies it plus a sibling `brand-assets/` directory into the target.

The scaffold script handles: directory creation, recursive copy, placeholder replacement across all text files, `.gitignore` generation, brand asset installation, and idempotent re-runs.

**After the script returns, run `scripts/verify-placeholders.mjs <target>`.** It scans the target for any leftover `__*__` tokens. If any remain, fail loudly — this means a template file was added without updating the REPLACEMENTS map.

### Phase 2.5 — Brand review (only when `--customer-url` was used)

When the scaffolder's output includes a `brand` key with `extractedFrom`, surface a scannable review to the SE **before** starting dev server. Example agent message:

```
Brand extracted from https://www.acme.com:

  Primary       #005fb2   (from <meta theme-color>)
  Secondary     #00a1e0   (from --brand-secondary CSS var)
  Font          "Salesforce Sans" → neighbor "Inter"
  Logo          public/brand/logo.svg
  Favicon       public/brand/favicon.png
  OG hero       public/brand/og-hero.png
  WCAG          primary vs. white: 4.63 (pass)   vs. dark: 3.83 (fail)

Anything to change before I boot the dev server? You can:
  • Accept as-is (recommended — everything is overrideable in site.config.ts later)
  • Edit a specific field now
  • Refresh from the same URL ("refresh-brand")
  • Skip the extracted branding entirely and fall back to defaults
```

If the extractor reported `robots-disallow` or `unreachable`, say so in plain English and offer to continue without extracted branding — do **not** stall the scaffold. If the primary color was nudged for WCAG contrast, show both the raw and nudged hex so the SE can see what changed and why.

### Phase 3 — Install & smoke test

1. `cd` to the target directory.
2. Ask once: "Install dependencies now? (recommended) [Y/n]". If yes, run `pnpm install` (or `npm install` / `bun install` based on what preflight found).
3. **Run `pnpm build` as a smoke test.** If build fails, print the first error and a note: "This is likely a template bug, not your fault. Please report to \<your team's skill maintainer\> with this error."
4. Start `pnpm dev` in the background and confirm `http://localhost:3000` responds.
5. Print a short summary: target path, dev URL, the 3 files the SE will edit first (`site.config.ts`, `content/docs/index.mdx`, `content/docs/architecture.mdx`), and suggested `git init` steps.

### Phase 4 — Next steps

End with a checklist the SE can hand to the customer:

- Where to edit each section (link `content/docs/*.mdx` paths)
- How to add new pages (update `meta.json`, drop `.mdx`)
- How to tune theme/title/personas (edit `site.config.ts` — one file, not ten)
- How to swap the logo (`public/logo.png`)
- How to deploy to the chosen target (or `/setup-docs deploy` in a future version)
- How to invoke `/update-docs` later to add sections

## `/update-docs` — incremental updates

Invoked when the SE already has a scaffolded site and wants to add or refresh a section.

**Required fields (2):**

1. Target directory (absolute path to existing scaffolded site)
2. Action — one of:
   - `add-section <slug>` — drop a new `.mdx` from the section library and update `meta.json`
   - `refresh-starter <slug>` — re-copy the starter content into an existing section (destructive; warns before overwrite)
   - `resync-config` — regenerate `site.config.ts` from intake (interactive)

The section library lives at `templates/fumadocs-poc/content/docs/_sections/`. `/update-docs` copies a single file, runs placeholder substitution, and appends the slug to `meta.json`. It does **not** touch unrelated files.

**Available sections** (canonical list — keep in sync with the library on disk):

| Slug | Purpose |
|------|---------|
| `security` | Threat model, sharing rules, secrets, perm audit, MFA, residency |
| `security-questionnaire` | Pre-answered SIG-Lite / CAIQ-Lite vendor-questionnaire responses |
| `observability` | Signals, review cadence, thresholds, SLO snapshot |
| `rollout-plan` | Phased rollout, comms, training, success metrics, rollback |
| `cost-model` | Flex Credit / volume-based cost projections + levers |
| `demo-script` | 5-min and 20-min demo scripts the customer can run themselves |
| `faq` | Customer-facing FAQ seeded with the questions admins ask in week 2 |
| `glossary` | Salesforce + product + customer-specific acronyms and terms |
| `runbook` | Standalone operational runbook (when `handoff.mdx` grows past ~300 lines) |
| `release-notes` | Keep-a-Changelog format with v0.1.0 stub and Unreleased bucket |

The library lives at `templates/sections/` (skill-level — **not** inside `templates/fumadocs-poc/` so the scaffolder doesn't accidentally render it as live Fumadocs pages). To discover what's actually on disk at runtime:

```bash
node scripts/scaffold.mjs --list-sections        # human format
node scripts/scaffold.mjs --list-sections --json # machine format
```

The table above is authoritative for the skill's prose; the script is authoritative for the filesystem. If they disagree, update whichever is wrong.

**How `add-section` works:**

```bash
node scripts/scaffold.mjs --add-section security --target /abs/path/to/site
```

The script reads intake values from `<target>/.poc-docs-meta.json` (written during initial scaffold), copies `templates/sections/<slug>.mdx` to `<target>/content/docs/<slug>.mdx` with placeholders substituted, and inserts `<slug>` into `meta.json`'s `pages` array **before** `troubleshooting` so Troubleshooting always stays last in the sidebar. Re-running on an already-added slug is a no-op unless `--force` is passed (then it overwrites the page contents but leaves `meta.json` as-is).

## Placeholder contract

All template files contain these literal, uppercase, double-underscored tokens. `scripts/scaffold.mjs` replaces them everywhere.

| Token | Source | Default if missing |
|-------|--------|--------------------|
| `__CUSTOMER_NAME__` | Customer display name | *(required)* |
| `__CUSTOMER_SLUG__` | Lowercased, hyphenated customer name | *(derived)* |
| `__POC_NAME__` | POC display name | *(required)* |
| `__POC_SLUG__` | Lowercased, hyphenated POC name | *(derived)* |
| `__PRODUCT_AREA__` | Product area string | *(required)* |
| `__INCLUDE_DATA_CLOUD__` | `true` if `__PRODUCT_AREA__` contains `Data 360`, else `false` | *(derived)* |
| `__INCLUDE_AGENTFORCE__` | `true` if `__PRODUCT_AREA__` contains `Agentforce`, else `false` | *(derived)* |
| `__DEPLOY_TARGET_PRIMARY__` | First value from deploy-target list | `Not decided` |
| `__DEPLOY_TARGETS__` | Full comma-separated deploy-target list | `Not decided` |
| `__PERSONAS__` | Comma-separated personas | `Admin, End User` |
| `__INTEGRATIONS__` | Comma-separated integrations | `None` |
| `__REPO_URL__` | Git remote URL | *(empty)* |
| `__SE_NAME__` | SE display name | OS username |
| `__YEAR__` | Current calendar year | Auto from script |
| `__CUSTOMER_URL__` | Raw customer website URL (string interpolation) | *(empty)* |
| `__CUSTOMER_URL_JS__` | JS-literal form of the URL for `site.config.ts` (`"..."` or `null`) | `null` |
| `__THEME_PRIMARY_HEX__` | JS literal: resolved primary hex or `null` | `null` |
| `__THEME_SECONDARY_HEX__` | JS literal: resolved secondary hex or `null` | `null` |
| `__THEME_FONT_SANS_NAME__` | JS literal: detected font family or `null` | `null` |
| `__THEME_FONT_SANS_PROVIDER__` | JS literal: `"google" \| "typekit" \| "custom" \| "system" \| null` | `null` |
| `__THEME_FONT_SANS_NEIGHBOR__` | JS literal: nearest Google Font for proprietary families or `null` | `null` |
| `__THEME_BRAND_LOGO_FILE__` | JS literal: filename within `public/brand/` or `null` | `null` |
| `__THEME_BRAND_LOGO_ALT__` | JS literal: alt text for logo `<img>` or `null` | `null` |
| `__THEME_FAVICON_FILE__` | JS literal: favicon filename within `public/brand/` or `null` | `null` |
| `__THEME_OG_HERO_FILE__` | JS literal: OG image filename within `public/brand/` or `null` | `null` |
| `__BRAND_EXTRACTED_FROM__` | JS literal: URL the snapshot came from or `null` | `null` |
| `__BRAND_EXTRACTED_AT__` | JS literal: ISO timestamp or `null` | `null` |
| `__BRAND_COMPANY_TAGLINE__` | JS literal: tagline pulled from og:description or `null` | `null` |

### Conditional rendering contract

Starter MDX uses the boolean `__INCLUDE_DATA_CLOUD__` / `__INCLUDE_AGENTFORCE__` tokens to gate product-specific content (e.g., the Data Cloud mapping table in `data-model.mdx` only renders when Data 360 is in scope). The scaffolder emits these as literal `true` / `false` into a generated `site.config.ts` export; MDX reads them via a `<ProductArea>` helper component shipped with the template. **Do not** do string-matching on `__PRODUCT_AREA__` in MDX — always branch on the booleans so typos in the product-area label don't silently disable sections.

To add a new placeholder: update `scripts/scaffold.mjs` `REPLACEMENTS` map, update this table, update `scripts/verify-placeholders.mjs` allowlist.

## Idempotency & safety

- `scripts/scaffold.mjs` refuses to write into a non-empty directory unless `--force` is passed. Always confirm with the SE before passing `--force`.
- `scripts/scaffold.mjs` never runs `rm -rf`. The script handles collisions by refusing, not by deleting.
- The template ships **no** `.env` files and no secrets. If an SE asks you to add credentials to the template, decline and point them at `site.config.ts` + runtime env vars.

## Common customizations (recipes)

Paste these verbatim when an SE asks "how do I…":

**Change the logo:**
Drop a new file into `public/brand/logo.svg` (or `.png`) and update `theme.brandLogoFile` in `site.config.ts` to match the filename. If the SE never provided a customer URL at scaffold time, `public/brand/` won't exist yet — just `mkdir -p public/brand && cp <logo> public/brand/` and set `theme.brandLogoFile = "logo.svg"`.

**Change the primary color:**
Edit `site.config.ts`, set `theme.primaryHex = "#005fb2"`. That value beats whatever the brand extractor put in `data/brand-snapshot.json`. Propagates to sidebar-active, link color, buttons, and Fumadocs `--fd-primary` through the CSS variables emitted in `app/layout.tsx`.

**Refresh brand from the customer's website:**
```bash
cd <target>
node <skill-dir>/scripts/brand-extractor/index.mjs \
  --url "https://www.<customer>.com" \
  --out ./data/brand-snapshot.json \
  --assets-out ./public/brand
```
Manual overrides in `site.config.ts` still win — only the snapshot-backed fields refresh.

**Opt out of brand extraction for this POC:**
Set the `theme.*` fields in `site.config.ts` back to `null` and delete `data/brand-snapshot.json`. The site falls back to clean Fumadocs defaults.

**Add a new top-level section called "Security":**
```
/update-docs add-section security
```
This drops `content/docs/security.mdx`, adds `"security"` to `content/docs/meta.json`, and runs placeholder substitution.

**Switch from Vercel to static export:**
Edit `next.config.mjs`, set `output: 'export'`. Run `pnpm build`. Deploy the `out/` directory anywhere.

## Extending the template

- To change what every future site looks like, edit `templates/fumadocs-poc/`.
- Keep placeholder tokens literal.
- After edits, dry-run: `node scripts/scaffold.mjs --target /tmp/dry-run-$(date +%s) --customer Test --poc Test --product-area Agentforce`, then run `pnpm build` in that directory. If build fails, your template change broke something.

## Reference files

- [`reference/intake.md`](reference/intake.md) — prompt wording + example answers
- [`reference/content-guide.md`](reference/content-guide.md) — what belongs in each POC section (for the SE, not for scaffold)
- [`reference/handoff-checklist.md`](reference/handoff-checklist.md) — what "done" looks like for a POC handoff
- [`scripts/preflight.mjs`](scripts/preflight.mjs) — environment check
- [`scripts/scaffold.mjs`](scripts/scaffold.mjs) — the one and only scaffolder
- [`scripts/verify-placeholders.mjs`](scripts/verify-placeholders.mjs) — post-scaffold sanity check
- [`scripts/brand-extractor/`](scripts/brand-extractor/) — zero-dependency brand extractor that reads a customer's public site, parses HTML/CSS, downloads logo/favicon/OG images, runs WCAG contrast checks, and writes `data/brand-snapshot.json`. Invoked automatically by `scaffold.mjs` when `--customer-url` is present; callable standalone for refreshes.

## Anti-patterns

- Don't generate MDX content inline in chat and write files one-by-one. Use the template.
- Don't hardcode customer values into template files. Always go through placeholders.
- Don't skip Phase 0. A broken Node install will eat 15 minutes of an SE's day otherwise.
- Don't skip the smoke test in Phase 3. A template that scaffolds but doesn't build is worse than no template.
- Don't pin `Node.js 16` / `Fumadocs 16` in prose anywhere. Cite minimums (`Node 22+`, `Fumadocs 16+`) and let `package.json` be canonical.
- Don't ask the SE all 9 intake fields up front. Four required, the rest deferred to `site.config.ts`.
- Don't use Mermaid (```` ```mermaid ````) in starter content. Fumadocs does not render it by default and an MDX page will appear as a raw code block. Use ASCII diagrams or drop images into `public/`.
- Don't add duplicate `# Heading` lines at the top of an MDX page — the Fumadocs `DocsTitle` already renders the frontmatter `title` as an `h1`. Starter MDX should begin with a short lede paragraph or the first real section (`## ...`).
- Don't put smart quotes (`"` `"` `'` `'`) or un-escaped apostrophes inside single-quoted JSX attribute values. MDX parses attributes as JSX; use double quotes, a template literal `title={\`…'…\`}`, or an expression container.

---
> Source: [dylandersen/afd360-poc-docs-skill](https://github.com/dylandersen/afd360-poc-docs-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
