## second-o-pi-nion

> Guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.)

# Agent guidelines — second-o-pi-nion

Guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.)
working on this repo. Humans should also skim this; nothing here is
agent-specific in spirit.

## What this is

A TypeScript extension for the third-party open-source CLI
[`@mariozechner/pi-coding-agent`](https://github.com/mariozechner/pi).
It registers a single tool, `second_opinion`, that the active LLM can
call when stuck or when wanting a sanity check. The reviewer model
runs **tool-less, single-shot** with only the supplied `problem` and
`context` strings.

Peer-deps only on `@mariozechner/pi-ai`,
`@mariozechner/pi-coding-agent`, and `typebox`. No Adobe-internal
dependencies, no servers.

## Build / test / typecheck

```bash
npm install        # if node_modules is missing
npm test           # runs the full test suite (currently 47 tests)
npm run typecheck  # tsc --noEmit against tsconfig.check.json
```

`npm test` must stay green on every PR. Don't merge red.

The `evals/` directory contains a separate golden-eval harness for
periodically re-evaluating the reviewer-model candidate list. It is
not part of `npm test`; run it manually when changing ranking logic.

## Packaging & runtime (read before touching `engines` or the loader)

- pi loads `.ts` extensions through its bundled TypeScript loader
  (`@mariozechner/jiti`), **not** Node's native `--experimental-strip-types`.
  So the published package's `engines.node` must mirror the host CLI
  (`@mariozechner/pi-coding-agent`, currently `>=20.6.0`), **not** the Node
  version the `npm test` / `eval` scripts happen to need. Do not bump
  `engines.node` to 22.x just because those scripts pass
  `--experimental-strip-types`; that flag is a dev/test-only concern and is
  irrelevant to how the shipped extension is loaded at runtime.
- pi has first-class npm support: `pi install npm:@adobe/second-o-pi-nion`
  is the primary install path. Keep the published tarball loadable as-is
  (no build step) — ship the `.ts` sources, not compiled JS.
- The `files` whitelist in `package.json` is what ends up in the npm tarball
  (`index.ts`, `lib.ts`, `logo.png` for the README image, license/docs). If
  you add a runtime file the extension needs, add it to `files` too, or it
  won't ship. `evals/` and `test/` are intentionally excluded. Verify with
  `npm pack --dry-run` before publishing.
- General lesson worth keeping: don't assert a dependency's internal
  mechanism (loader, engine requirement, transport) without checking its
  actual code/manifest first. This note exists because that check was once
  skipped.

## Code style

- TypeScript, `strict: true`. No `any` without a written justification
  in a comment.
- Source lives in `index.ts` (extension surface) and `lib.ts` (logic).
  Keep the split — `lib.ts` is what the tests exercise directly.
- No new runtime dependencies without discussion.
- Apache-2.0 header on every new `.ts` file, copyright
  `2026 Adobe`. Match the format of the existing source.

## Commit / PR hygiene

- Conventional-commits style commit messages.
- Squash PR branches to a single descriptive commit before merging.
- Don't bundle unrequested work. Surface adjacent fixes in the PR
  description and ask — don't silently include them.

## Trust model

Read the **"Trust model & security notes"** section of `README.md`
before changing anything in the prompt-construction path, the
secret-redaction pass, or the candidate-ranking heuristic. The threat
model is documented there in full.

## Security invariants — don't weaken these

These were tightened during the open-source review. Don't relax them
without an explicit discussion on the PR:

- **Secret redaction on the outgoing prompt is damage reduction, not
  protection.** Don't sell it as security in docs, comments, or commit
  messages — the README is deliberately blunt about this and must stay
  that way. The redactor is a best-effort regex layer that catches a
  list of known token shapes (see `compileRules()` in `lib.ts`); it
  will miss novel, re-encoded, whitespace-split, or non-standard
  secrets. The visible-on-redact notice must remain so the caller can
  see when redaction fired. Adding new token shapes is a clear win and
  welcome — but don't reorder existing rules without understanding the
  regex-replace consumption order documented at the top of
  `compileRules()`.
- **`BLOCK_ON_SECRETS` opt-in.** When set, the call must refuse rather
  than redact. Don't quietly downgrade to "redact and continue".
- **Reviewer model is tool-less and single-shot.** It does not get
  file access, shell access, or any other tool. Don't grant tools "for
  convenience" — the whole point of the second opinion is that it
  cannot read your files or run your commands.
- **Same-family skip in candidate ranking.** Never recommend the same
  model family that's calling. The skip is what makes the second
  opinion *second*.
- **Candidate list is built from configured providers.** Don't bake
  provider names or specific model IDs into runtime defaults. The user's
  configured pi-ai providers are the source of truth; an env var
  (`PI_SECOND_OPINION_MODEL`) is the only override.
- **No new network calls** beyond the LLM call itself. No telemetry,
  no usage analytics, no crash reporters.

---
> Source: [adobe/second-o-pi-nion](https://github.com/adobe/second-o-pi-nion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
