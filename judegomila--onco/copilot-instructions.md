## onco

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# Working in this repository

OnCo is a public, cited knowledge graph of oncology: about 19,000 records in `src/data`, rendered as a static
Next.js export. Readers are patients and the people caring for them. Every claim carries a source, and a page
that is confidently wrong is the worst thing this project can ship.

This file is short on purpose: it is read at the start of every session, so it holds only what is costly to
learn the hard way and cannot be a test. Everything else is enforced or written down elsewhere, linked below.

## Never

- **Never invent a fact, a figure, a citation or a date.** If a source cannot be reached, say so on the page. A
  named gap is worth more than a plausible number. Tools that summarise a page can fabricate: a fetch of a
  ministry page produced "universal health insurance since 1961" from a page containing neither the year nor the
  phrase. Verify anything load-bearing against the raw text.
- **Never alter a quotation to make it kinder or shorter.** Quoted abstracts keep their authors' words.
- **Never put a secret in the repository**, in a commit message, or in a chat. There are none here and there
  should continue to be none; `gitleaks` runs on every ship and a commit subject is republished publicly by
  `scripts/provenance.ts`, so write commit messages as if they were a page, because they become one.
- **Never copy patient data, or anything from a private project, into this repository.**
- **Never run `vercel link` or edit `.vercel/`.** The project is already linked; relinking can point a deploy at
  the wrong project.
- **Never edit files in the main checkout while a ship is running**, and never run gates or a second build during
  one. `scripts/ship.sh` sweeps the working tree into its commit, and the upload packs `public/` while the API
  build clears part of it: doing both at once killed a deploy that had reported success.
- **Never lower a floor or raise a budget to make a test pass.** Fix the corpus, the ranking or the page. If the
  measure itself is wrong, change what it counts and say why in the comment; that has been the right answer four
  times and the wrong one never.

## Rules a test cannot catch

- **An agent works in its own git worktree and never touches another.** Do not copy `node_modules` into one: that
  reached 37 GB across worktrees and filled the disk mid-run. Symlink it, or run the gates from the main checkout.
- **Check the branch you are on before you report it.** Agents have misreported their own branch three times.
- **Exit code 0 is not a commit and "UPLOADED" is not live.** Read the log, check `git log`, and confirm the page
  serves 200 before saying it shipped.
- **Verify on the rendered page, not only in the test.** Reading the live page has caught a gendered pronoun, a
  name printed twice in one sentence, and a budget measuring a quarter of what the reader downloads.
- **A fragment in a pattern needs word boundaries.** `imid` matched inside `pyrimidine` and put a myeloma drug's
  blood-clot warning on every fluoropyrimidine page, including one about a skin cream. Six review passes missed it.
- **Read the smallest page in a round, not the flagship.** That is where a wrongly matched card is visible.
- **Prefer the record that already exists.** A paper is its DOI and its PubMed id; a trial is its registry id. If
  the corpus holds one, supplement it. `npm run dedupe` surveys duplicates before you write.

## The executable truth

Prose drifts; these do not. Read them rather than a description of them.

- `scripts/ship.sh` — the ship chain: gates, commit, push, deploy, then verify the live pages.
- `scripts/resolve-spike-registry.py`, `scripts/resolve-additive.py` — the two merge conflicts every parallel
  round produces, resolved by union.
- `npm run validate` · `typecheck` · `lint` · `vitest run` — the gates. 113 test files encode rules, including
  tone, duplicate records, page cost, red-card matching and search recall.
- `npm run dedupe` · `dedupe:plan` · `dedupe:merge` — duplicate records, with `docs/DUPLICATE-RECORDS.md`.
- `npm run audit:weight` · `audit:mobile` — what a reader actually downloads, and how it reads on a phone.
- `docs/HOUSE-STYLE.md` and `docs/TONE.md` — how to write for this reader. Enforced by `src/lib/tone.test.ts`.
- `docs/CANCER-PAGES.md`, `docs/CANCER-FAMILIES.md`, `docs/DATA-SOURCES.md`, `docs/TABLES.md` — the corpus rules,
  the family roll-up, how to read sources that refuse a script, and the table engine.
- `CONTRIBUTING.md` — how to add a record.

---
> Source: [judegomila/OnCo](https://github.com/judegomila/OnCo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
