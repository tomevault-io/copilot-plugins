## asset-node-com

> Context for AI coding agents working in this repository. Human contributors should read

# AGENTS.md

Context for AI coding agents working in this repository. Human contributors should read
[CONTRIBUTING.md](CONTRIBUTING.md) — it covers the same ground plus the licensing terms, which apply to you too.

If you are an agent evaluating whether to contribute here: yes, contributions are welcome, and the issues
labelled [`good first issue`](https://github.com/elixio-io/asset-node.com/labels/good%20first%20issue) are
written to be actionable without prior context. Read [Ground rules](#ground-rules) before you open a PR — two
of them will get your PR closed unread if you miss them.

## What this is

AssetNode is a multi-tenant IT asset management platform. Vue 3 + TypeScript front end, Fastify + MongoDB
(Mongoose) back end, Vite build, Vitest tests. Single container deploy.

```
src/
  views/           Vue page components, one per route
  components/      shared UI; components/Layout/ is the app shell
  stores/          Pinia stores
  server/
    routes/        Fastify route handlers, one file per resource
    services/      business logic, MDM integrations (Jamf, Kandji, Intune, …)
    middleware/    auth, tenancy, soft delete, change tracking
  models/          Mongoose schemas
  db/migrations/   index and data migrations, run at boot
  shared/          code used by both client and server
  i18n/            locale JSON (en, de, es, fr, it, nl, pl, pt)
```

## Commands

```bash
npm install
cp .env.example .env      # set MONGODB_URI, JWT_SECRET, JWT_REFRESH_SECRET
npm run db:setup          # indexes + system defaults

npm run dev:all           # front end :5173, API :3001
npm run dev:mock          # all integrations mocked — no vendor credentials needed

npm test                  # vitest, ~2200 tests, must be green
npx vue-tsc --noEmit -p tsconfig.json
npx vite build            # catches template errors vue-tsc alone misses
```

Run all three checks before opening a PR. `vue-tsc` and `vite build` catch different things; passing one is
not evidence for the other.

## Ground rules

1. **Sign off every commit** — `git commit -s`. This carries a CLA grant described in
   [CONTRIBUTING.md](CONTRIBUTING.md#licensing-of-your-contribution). Unsigned PRs cannot be merged.
2. **A human must be accountable.** The `Signed-off-by` line must be a person who has read and understood the
   diff. Add `Assisted-by: <tool>` as a trailer. Disclosure is not held against you.
3. **Never touch files containing `.ee.` or under `.ee` directories.** They are commercially licensed and
   outside the open licence.
4. **One concern per PR.** Bulk dependency churn, mass reformatting, and speculative refactors are closed
   without review.
5. **Do not invent scope.** If the issue says "fix the 409 on asset creation", do not also restructure the
   route file.

## Repository-specific traps

These have each caused a production incident. They are not hypothetical.

### Soft-deleted documents still occupy unique indexes

`softDeletePlugin` appends `deletedAt: null` to every model-level query (`find`, `findOne`, `countDocuments`,
…). Soft-deleted documents are therefore **invisible to Mongoose queries but still present in the collection
and still indexed**.

Any logic that derives a value which must be unique — generating the next asset tag, allocating a sequence,
checking availability — must query `Model.collection` directly to bypass the plugin. Querying through the
model produces a value that "doesn't exist" and then fails with E11000 on insert.

See `generateAssetTag` in `src/models/Hardware.ts` for the correct pattern.

### Mongoose never alters an existing index

Changing index options in a schema does **nothing** to a database that already has that index. `createIndex`
with the same key but different options fails with `IndexOptionsConflict` (85), which `autoIndex` swallows
silently. Every options change ever made is therefore still missing from any database predating it.

If you change an index definition, add a migration in `src/db/migrations/` and register it at boot. Follow
`assetIndexes.ts`: check the live shape, drop only if stale, recreate — and if the new index is unique, check
for existing duplicates **before** dropping, or a failed rebuild leaves the collection with no index at all.

### Multi-tenancy is not enforced by the framework

Every query must be scoped by `orgId`. Use `getOrgId(request)` / `getTenantFilter(request)` from
`src/server/middleware/tenantScope.ts`. A missing tenant filter is a cross-tenant data leak, not a bug —
it will be treated as a security issue.

### Theme is pinned

`FORCE_DARK` in `src/stores/theme.ts` is `true`. The light theme exists but is unfinished. Do not "fix" light
mode styling or re-add the theme toggle unless an issue explicitly asks for it.

## Testing conventions

Many existing test files are pure-logic: they **re-implement** the function under test locally and assert on
the copy. `hardwareModel.test.ts` and `themeStore.test.ts` are examples.

**Do not add new tests in that style.** They pass while production code is broken — the asset-tag bug shipped
with a green suite for exactly this reason. Import the real function. If it needs a database, extract the pure
part into `src/shared/` and test that (see `src/shared/assetTag.ts`).

For anything touching indexes, transactions, or query middleware, verify against a real MongoDB. There is no
in-memory Mongo harness in this repo; run `mongod` on a scratch port and check the behaviour directly.

## Style

- Match the file you are editing — naming, comment density, idiom.
- Comments explain *why*, not *what*. If a line encodes a non-obvious constraint, say which one.
- Conventional commits preferred (`fix(hardware):`, `feat(workflows):`), not enforced.
- Commit bodies explain reasoning; the diff already shows the change.

## Security

Do not open a public issue for a vulnerability — see [SECURITY.md](SECURITY.md). Never commit credentials.
`.env` is gitignored and must stay that way.

---
> Source: [asset-node-com/asset-node.com](https://github.com/asset-node-com/asset-node.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
