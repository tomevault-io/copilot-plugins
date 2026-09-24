## commonplace

> A Next.js litigation workspace: matters, a paginated document editor, reusable templates,

# Working on this codebase

A Next.js litigation workspace: matters, a paginated document editor, reusable templates,
connected legal research, Word export, and optional AI drafting. Local-first, deployable to
Vercel with a hosted libSQL database.

If you have just forked this to set it up for a firm, run `/firm-setup` rather than editing files
directly. It interviews the lawyer and writes the pack for you.

## The one thing to understand first

Everything jurisdiction-specific lives in **`app/jurisdictions/`**. The rest of the app knows
about "a court system", "a forum", and "a practice area" — never about Florida.

```
app/jurisdictions/types.ts   the pack contract; read this before changing a pack
app/jurisdictions/fl.ts      the Florida pack — the reference example
app/jurisdictions/index.ts   the active pack + the indexes built from it
app/catalogs/civil.ts        jurisdiction-NEUTRAL civil documents, with {{token}} placeholders
app/catalogs/index.ts        composes core + pack, and fills tokens
```

To serve a different jurisdiction you write one new pack file and change one import in
`app/jurisdictions/index.ts`. You should not need to touch `legal-data.ts`, `court-resources.ts`,
or `research-config.ts` — if you find yourself wanting to, something belongs in the pack that
isn't there yet, and that is worth fixing properly.

`docs/JURISDICTION_PACKS.md` explains the design and why it is shaped this way.

## Rules that will bite you

**Document keys are a persistence contract.** A catalog row's `key` becomes the document id and
is stored in saved practice settings and template links. Never change or reuse one. Add a new row
with a fresh key rather than renumbering neighbours. `tests/catalog-ids.test.mjs` pins the set.

**Forum ids and court system ids are persisted too.** `Matter.forum` and `Matter.jurisdiction`
store these strings verbatim. Renaming one orphans every matter that used it.

**Catalog order is behaviour.** Composition is `[...core rows, ...pack additions]`, and that is
the order documents appear in a matter. Reordering rows reorders existing matters' files.

**A wrong caption renders perfectly.** This is the failure mode the whole design guards against.
Nothing crashes, nothing looks broken, and the mistake reaches a filing. Prefer failing a test or
showing a visible `{{token}}` over silently producing plausible output.

**Never invent legal content.** Rule numbers, statutory cites, deadlines, court names, case-number
formats. If you don't know a jurisdiction's real value, leave a bracketed placeholder — the app's
own convention — and tell the user it needs checking. A confident wrong citation is worse than an
obvious gap.

## Verification

```bash
npm test        # builds, then runs the suite (~106 tests)
npm run lint
npx tsc --noEmit
```

Pack work is covered by `tests/forum-table.test.mjs` and `tests/catalog-terms.test.mjs`. Most of
their assertions are written against the pack's own declarations rather than Florida values, so a
new pack runs them unchanged — they are the grader for a generated pack.

When changing anything that generates documents, diff the output before and after rather than
trusting a read-through. Every phase of the pack extraction was verified this way and each came
out byte-identical; that is the standard to hold.

## Conventions

- TypeScript throughout, `strict`. No `any` without a comment saying why.
- Comments explain *why*, not what. Match the density of the file you're in.
- Modules under `app/` that the test suite loads directly need explicit `.ts` import extensions —
  Node's ESM resolver takes neither directory imports nor extensionless files.
- Bracketed `[instructions to the drafter]` are the house convention inside document bodies.
- Don't run `next build` while a dev server is live on the same directory; it pollutes `.next`
  and the dev server then serves stale CSS.

---
> Source: [samharden/commonplace](https://github.com/samharden/commonplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
