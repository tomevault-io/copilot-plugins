## x-notion-sync

> This project syncs the accounts a user follows on X into a Notion database. The main user path should stay simple: install dependencies, run `npm run setup`, review the 10-profile test sync, then run `npm run backfill` for the full import.

# x-notion-sync

This project syncs the accounts a user follows on X into a Notion database. The main user path should stay simple: install dependencies, run `npm run setup`, review the 10-profile test sync, then run `npm run backfill` for the full import.

## Key Files

- `src/index.ts` — Notion Worker entrypoint, sync schedule, X fetch loop, Notion page creation, audit rows.
- `src/x/` — X API client and OAuth 1.0a signing.
- `src/notion/` — Notion schema, page property builders, and table views.
- `src/classify/` — optional Anthropic classifier.
- `src/cost.ts` — cost estimates shown during setup and classification.
- `scripts/setup.ts` — first-run setup wizard.
- `scripts/backfill.ts` — switches from test import to full import, or restarts an import.
- `scripts/classification.ts` — enables, disables, or backfills AI tags.
- `scripts/inspect.ts` — builds the worker and prints its manifest before deploy.

## Setup Flow

`npm run setup` should remain the primary setup path. It:

1. Checks for the Notion CLI and logs in if needed.
2. Validates the Notion token.
3. Validates the four X OAuth 1.0a values against `/users/me`.
4. Lets the user choose basic sync or AI tagging.
5. Shows a cost estimate and defaults to a 10-profile test import.
6. Creates or reuses the `X Contacts` database.
7. Reconciles missing schema or views on rerun.
8. Deploys the worker and triggers `xFollowingSync`.

If setup fails after creating a database, rerunning setup should reuse the saved database ID from `.env`.

## Sync Behavior

- The first run imports `/following` pages until X runs out of pages, unless test mode caps it at 10 profiles.
- Later runs check the newest follows first and stop when they reach profiles already in Notion.
- The sync is adds-only. Unfollows are not removed from Notion.
- The dedupe key is the `X User ID` rich text property.
- X profile images are rewritten from `_normal` to `_400x400` for sharper Notion icons.
- Select and multi-select option names must replace commas with semicolons and stay under 100 characters.

## AI Tagging

AI tagging is on only when `ANTHROPIC_API_KEY` is present. New profiles are classified during sync. Existing untagged rows are handled by `npm run classification`.

Do not reclassify every row during the weekly sync. That would surprise users and add cost.

## Rules

- Do not hardcode user-specific values in `src/` or `scripts/`. Use env vars.
- Do not commit `.env`.
- Keep the X API pacer. X bills per profile returned, and accidental loops are expensive.
- Do not reuse a worker capability key across databases, pacers, syncs, tools, or webhooks.
- Avoid new `as any` casts. If a Notion SDK type forces one, keep it tightly scoped and explain why.
- When adding a Notion property, update schema, property builders, view payloads, and tests together.
- Keep README instructions short and user-focused. Prefer concrete commands over architecture detail.

## Verification

Run before shipping changes:

```bash
npm run check
npm test
```

Run this before deploy-related changes:

```bash
npm run inspect
```

End-to-end tests are opt-in because they use a real Notion workspace:

```bash
RUN_E2E=1 npm run test:e2e
```

For integration tokens, `E2E_PARENT_PAGE_ID` must point to a page the integration can access. Personal access tokens can create the sandbox database at the workspace root.

---
> Source: [brianlovin/x-notion-sync](https://github.com/brianlovin/x-notion-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
