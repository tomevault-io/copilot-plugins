## insta-checkout

> Follow these rules unless the user explicitly instructs otherwise.

# Insta Checkout — Agent Instructions

Follow these rules unless the user explicitly instructs otherwise.

## Critical Rules

- **Do NOT commit** any work until the user explicitly asks you to.
- **Do NOT deploy** any work until the user explicitly asks you to.
- **Do NOT push** to remote until the user explicitly asks you to.
- **Do NOT merge main** into the current branch unless the user explicitly asks you to.
- **Do NOT access any production data store** without explicit, in-the-moment user approval. This covers — at minimum — the **production MongoDB cluster**, the **production Firebase project** (Auth, Storage, Firestore, Realtime Database), and any other prod-side data store the project adds later. Read, write, delete, query, admin-SDK calls, CLI commands, scripts, and one-off connections all count as "access". Working against dev/staging/PR-env data stores is fine and does not need approval. When in doubt about whether a connection string, project ID, bucket name, or service account belongs to prod, **stop and ask** — never guess. If a task seems to require touching prod, propose the action and wait for an explicit "yes, go ahead with prod" before proceeding.

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions).
- If something goes sideways, STOP and re-plan immediately — don't keep pushing.
- Use plan mode for verification steps, not just building.
- Write detailed specs upfront to reduce ambiguity.
- **Exception**: Skip plan mode during Night Shift — execute tasks directly without entering plan mode.

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean.
- Offload research, exploration, and parallel analysis to subagents.
- For complex problems, throw more compute at it via subagents.
- One task per subagent for focused execution.

### 3. Self-Improvement Loop
- After ANY correction from the user: update memory files with the pattern.
- Write rules for yourself that prevent the same mistake.
- Review lessons at session start for relevant project.

### 4. Verification Before Done
- Never mark a task complete without proving it works.
- Diff behavior between main and your changes when relevant.
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness.

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution."
- Skip this for simple, obvious fixes — don't over-engineer.
- Challenge your own work before presenting it.

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding.
- Point at logs, errors, failing tests — then resolve them.
- Zero context switching required from the user.
- Go fix failing CI tests without being told how.

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Minimal code impact.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.
- **Follow Existing Patterns**: Match the codebase's existing structures and conventions.

## Brand Voice (MANDATORY)

All user-facing text in the **landing**, **checkout**, and **admin** apps — in **both Arabic and English** — MUST follow the Brand Voice Guide. It is the single source of truth for how Insta Checkout communicates:

➡️ https://app.notion.com/p/karimtamer/Brand-Voice-Guide-31cc92f98d9c81acbb1ae8cfd0e217b4

**Scope:** any copy a user reads — UI labels, buttons, headings, body copy, onboarding, empty/error/success states, tooltips, emails, and the i18n strings in `packages/i18n/messages/*.json` (both `en.json` and `ar.json`). This applies whenever you add or change text, not just at commit time.

**Before writing or changing any user-facing string, fetch the page** (Notion connector) so you are working from the current guidance. If Notion is unreachable (e.g. headless / night-shift runs), fall back to the distilled rules below — but the live page wins on any conflict.

### Voice in one line
Write like a **trusted local expert**: trustworthy, clear and direct, locally aware (built for Egypt / InstaPay), warm and human. The voice is fixed; only the tone shifts by channel — product UI is clear and neutral, onboarding is warm and encouraging, error messages are empathetic and never blame the user.

### Always
- Plain, short sentences. Every sentence must earn its place.
- Sentence case for headings ("Create a payment link").
- Oxford comma; `%` symbol; spell out one–nine, numerals for 10+.
- Contractions in conversational copy; at most one exclamation mark per piece, only when earned.
- Bold for emphasis — never ALL CAPS.

### Never
- Hype words: "revolutionary", "game-changing", "disrupting", "seamless", "easy", or a loose "empower".
- "solution" when you mean product/tool; "user" / "merchant" / "vendor" when you mean **seller**.

### Preferred terminology
payment link (not paylink) · seller (not user/merchant/vendor) · dashboard · sign up / log in (verbs) · InstaPay · Insta Checkout · funds (not money/cash in UI).

### Arabic
- Natural Egyptian dialect, not formal فصحى (unless the context is legal / formal B2B). Don't directly translate the English — make it read natively.
- Prefer: دوس (not اضغط) · تحويل (not دفعة) · من فضلك / فعل أمر مباشر (not يرجى / الرجاء) · اللي تحت (not أدناه).
- Default to gender-neutral phrasing.

Run the Brand Voice Checklist (section 8 of the guide) before treating any copy as done.

## Project-Specific Setup

### Worktree Setup
- After creating a worktree, always run `npx pnpm install` before attempting to build or run any app. This is required because workspace dependencies (like `@insta-checkout/i18n`) are not automatically linked in new worktrees.

### Worktree Naming
- Once you understand the task, rename the current worktree branch to a short, descriptive kebab-case name reflecting the task (e.g., `claude/fix-login-bug`, `claude/add-dark-mode`). Use `git branch -m <new-name>` to rename.

### Pulling Main
- At the start of each new session, always pull main (`git pull origin main`) to ensure you have the latest changes before starting any work.

### Dev Server Management
- At the start of each new conversation, close any active dev server sessions using `preview_stop` before starting a new one.
- When asked to run apps without preview, provide manual commands for the user to run in their terminal:
  - Landing: `npm run dev:landing` (port 3000)
  - Checkout: `npm run dev:checkout` (port 3001)
  - Backend: `npm run dev:backend` (port 4000)

### Notion
- When asked to view or check something in Notion, use the Notion connector and navigate to the Insta Checkout workspace: https://www.notion.so/karimtamer/Insta-Checkout-f555b3bf0c434947a1a37613eda62c1b

## File Uploads

**Uploads always go to Firebase Storage, never to the backend's local disk.** The backend runs on Railway, whose container filesystem is ephemeral — anything written to `process.cwd()/uploads/` is wiped on every redeploy/restart, leaving DB rows pointing at 404s.

### Convention
- **Bucket**: `instacheckout-a4141.firebasestorage.app` (set via `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET`).
- **Path layout**: `{resource}/{ownerOrLinkId}/{timestamp}_{shortId}.{ext}`.
- **Upload from the client** using the Firebase Web SDK (`firebase/storage`); the backend only persists the resulting download URL.
- Per-resource access is controlled by `storage.rules` at the repo root.

### Existing helpers
| Resource | Helper | Path |
|---|---|---|
| Seller logos (onboarding) | `apps/landing/lib/firebase.ts:uploadSellerLogo` | `logos/{firebaseUid}/...` |
| Product images | `apps/landing/lib/firebase.ts:uploadProductImage` | `products/{firebaseUid}/...` |
| Branding (logo + cover photo) | `apps/landing/lib/firebase.ts:uploadSellerBranding` | `branding/{firebaseUid}/...` |
| Customer payment screenshots | `apps/checkout/lib/firebase.ts:uploadCheckoutScreenshot` | `screenshots/{paymentLinkId}/...` |

### When adding a new upload
1. Add a helper in the relevant app's `lib/firebase.ts` that mirrors the existing pattern (size + MIME validation, namespaced path, returns `getDownloadURL` result).
2. Add a matching `match /...` block to `storage.rules` with size + MIME guards. Authenticated paths gate by `request.auth.uid`; anonymous paths (e.g. customer screenshots) gate by path prefix only.
3. Deploy rules with `firebase deploy --only storage` (config in `firebase.json`).
4. Backend should validate the URL belongs to our bucket before persisting via `isValidFirebaseStorageUrl` from `apps/backend/src/services/firebaseStorage.ts`. Never accept arbitrary URLs.

## Internationalisation (i18n)

**All user-facing text in the landing and checkout apps MUST live in the shared translation files — never hardcoded inline.**

- The single source of truth is `packages/i18n/messages/en.json` and `ar.json` (the `@insta-checkout/i18n` workspace package).
- The two files must always have **identical key sets** (English and Arabic in lockstep). Add every new key to **both**.
- In components, read copy via `useTranslations()` from `@/lib/locale-provider`: `t("namespace.key")` for strings (supports `{var}` interpolation, e.g. `t("dashboard.products.imageTooLarge", { size })`), and `get("namespace.key")` for arrays/objects.
- **Do NOT** write inline strings such as `locale === "ar" ? "..." : "..."`, hardcoded Arabic/English literals in JSX, or per-locale ternaries. If you catch yourself typing user-facing text in a component, stop and add a key instead.
- Keys are organised by feature namespace (`common`, `landing`, `checkout`, `dashboard`, `onboard`, `branding`, …). Reuse an existing key (e.g. `common.egpShort` for the EGP suffix, `common.listSeparator`) before adding a new one.

**Documented exemptions** (these may legitimately hold non-translation-file text):
- **SEO metadata** — static Next.js `metadata` exports (page `title`/`description`) are intentionally bilingual and cannot use the client hook.
- **Intentional dual-locale / demo content** — e.g. `CheckoutLocalePreview` (shows both locales at once), landing demo seed data, the Arabic-first splash page. These show fixed text regardless of the active locale by design.
- **Number formatting** — Arabic-Indic digit conversion logic (e.g. `story-dashboard`) is formatting code, not copy.
- **Backend** (`apps/backend`) — emails, the WhatsApp bot, and role labels are **not yet wired** to `@insta-checkout/i18n`. They currently hold inline strings; centralising them is a tracked follow-up (it requires importing `MESSAGES`/`getNested` server-side and adding `email`/`whatsapp` namespaces). Until then, keep backend copy consistent across the two languages.

## Skills

- **Project-scoped only**: When creating any new skill, always create it under `.claude/skills/` in this project — never in the global `~/.claude/skills/` directory. All skills for Insta Checkout must be project-specific.

## Night Shift Mode

When running in night shift mode (triggered via `/night-shift` skill):
- The agent operates autonomously following `.claude/skills/night-shift/SKILL.md`
- Commits and pushes to feature branches ARE allowed (single branch per session: `claude/night-shift-YYYY-MM-DD`)
- Creating PRs (not draft) IS allowed at session end
- Notion task status updates ARE allowed (only to "In Progress" or "🔍 To Be Reviewed")
- Writing Agent Reports on Notion task cards IS allowed
- All other Critical Rules still apply: **no merge, no deploy**
- The agent MUST leave a comment or Agent Report on every Notion task it touches
- The agent MUST NOT modify tasks it didn't work on

## Git & Commits

### Pre-commit Hook Strategy

When committing changes:

1. **Fix errors in your own code first** — ESLint, Prettier, TS errors in files you modified.

### Feature Documentation Gate (MANDATORY)

Every user-facing feature in the **landing** and **checkout** apps must be listed on the **Product Features** Notion page **before it is committed**:

➡️ https://app.notion.com/p/37bc92f98d9c8171ad39dbe27933e464

**Before every commit:**

1. Check whether the staged change adds or alters a user-facing capability in `apps/landing/` or `apps/checkout/` — a new page, component, action, state, field, toggle, validation, copy, empty/error state, etc.
2. If it does, open the Product Features page and confirm that capability is documented there.
3. **If the feature is not yet on the page, STOP — do not commit. Update the page in the same change-set, then commit.** An undocumented user-facing feature must hard-cancel the commit, not slip through.
4. If the change touches no user-facing feature (refactor, bug fix, dependency bump, test-only, internal admin- or backend-only change), it is exempt.

This is the feature-side equivalent of the backend **API Documentation Sync** rule — the Product Features page is the single source of truth for what the product can do.

**Enforcement (git hook):** A `pre-commit` hook at `.githooks/pre-commit` hard-blocks any commit that stages user-facing source in `apps/landing/` or `apps/checkout/` (under `app/`, `components/`, `pages/`, `lib/`, or `src/`; generic `components/ui/` primitives and `*.test`/`*.spec` files are exempt). It is activated automatically on `npm`/`pnpm install` via the root `prepare` script, which runs `git config core.hooksPath .githooks`. After confirming the page is up to date (or the change is exempt), let the commit through with the per-commit acknowledgement variable:

```
FEATURES_DOCUMENTED=1 git commit ...
```

Use the per-commit variable — never disable the hook globally — so the check stays honest.

---
> Source: [Insta-Checkout/insta-checkout](https://github.com/Insta-Checkout/insta-checkout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
