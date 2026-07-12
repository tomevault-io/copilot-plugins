## congressintel

> Personal track. Single-user web app surfacing U.S. congressional stock disclosures with AI-driven insider-risk scoring.

# CongressIntel

Personal track. Single-user web app surfacing U.S. congressional stock disclosures with AI-driven insider-risk scoring.

Canonical spec: `context-brain/working/congressintel/SPEC_CongressIntel.md` (architecture, secrets, deployment, known gaps).

## Local conventions

- Package manager: **npm**. `congressintel/` and `worker/` are independent packages — install separately.
- Node: pinned to **20** in CI (GitHub Actions). Local Node version unpinned.
- Linter / formatter: **none**. No pre-commit hooks.
- Tests: **none**. Verification is manual smoke-test (see SPEC §7).
- Dev:
  - Frontend: `cd congressintel && npm run dev` → http://localhost:5173/congressintel/
  - Worker: `cd worker && npx wrangler dev` → http://localhost:8787
- Build (frontend only): `cd congressintel && npm run build`

## Deployment guardrails

- **Frontend:** `git push` to `main` → GitHub Actions builds + deploys to Pages. Do not bypass with a manual `gh-pages` push.
- **Worker:** `cd worker && npx wrangler deploy`. There is no `--env production` — single environment.
- **Before any worker deploy:** confirm `OPENAI_API_KEY` and `APP_TOKEN` are still set on the worker (`npx wrangler secret list`). Missing secrets cause 401s for every request.
- **After editing `worker/src/index.js`:** verify `git status` shows it modified before stepping away. If clean — see Known footguns §1.
- Never run `npm publish` from any package.
- Never re-introduce `VITE_APP_TOKEN` as a build-time env var (re-exposes token in the public bundle). See SPEC §5.
- Never embed a PAT in the git remote URL. Auth goes through `gh`'s credential helper.

## Repository conventions

- **Branch model:** trunk-based. Push to `main` triggers deploy. No feature branches in use.
- **Commit messages:** free-form descriptive subjects. See `git log --oneline` for prior examples.
- **Gitignored-but-required at runtime:**
  - `congressintel/.env` (local frontend dev): `VITE_WORKER_URL=http://localhost:8787`
  - `worker/.dev.vars` (only during `wrangler dev`): `APP_TOKEN=...`, `OPENAI_API_KEY=...`
- **Do not touch:**
  - `congressintel/dist/` — build output
  - `worker/.wrangler/` — wrangler cache. Exception: read `worker/.wrangler/tmp/dev-*/index.js` for sync-race recovery (see footguns §1).
  - `*/node_modules/`

## Known footguns

1. **Dropbox sync race (RESOLVED 2026-05 — historical).** Workspace was migrated out of Dropbox CloudStorage to `~/Code/congressintel/`; the sync-race that silently reverted `worker/src/index.js` (April–May 2026; full incident in SPEC §10) no longer applies. Recovery technique remains generally useful: if `git status` is unexpectedly clean after editing worker source, check `worker/.wrangler/tmp/dev-*/index.js` for the latest bundled version.
2. **Yahoo Finance User-Agent.** Worker fetch to `query1.finance.yahoo.com` hardcodes `User-Agent: Mozilla/5.0 (compatible; CongressIntel/1.0)`. Don't remove — Cloudflare Workers' default UA gets 401'd.
3. **Valuation-step prompt-sniff coupling.** Worker matches `/^Valuation and technical snapshot for ([A-Z0-9.\-]{1,10}):/` against the user prompt to inject live price data. Changing the frontend step-5 prompt wording in `congressintel/src/App.jsx` silently disables injection. Keep them in sync, or refactor to an explicit `step`/`ticker` body field. See SPEC §10.
4. **wrangler dev bundle cache.** When code changes don't appear live, check `worker/.wrangler/tmp/dev-*/index.js` — this cache is also authoritative and was the recovery path during the sync-race incident.
5. **Credential-logging defense-in-depth not implemented (low risk, header-only).** OpenAI key is transmitted via `Authorization: Bearer ${env.OPENAI_API_KEY}` header in `worker/src/index.js`, not via URL or query string, so the high-risk URL-leak pattern (Muninn 2026-05-19) does not apply here. However, per `context-brain/input/Protocols/PROTOCOL_credential_logging.md` §5, header-based credentials still warrant a defense-in-depth scrub filter — third-party libraries occasionally log full request objects. Flag if logging behavior changes, error-handling around the OpenAI fetch expands, or a new URL/query-credential API is added (e.g. Polygon).

## Escalate-to-Henry triggers

Stop and ask before doing any of the following:

- Adding any new dependency to either `package.json`.
- Anything touching the OpenAI key, `APP_TOKEN`, OpenAI usage caps, or production data.
- Schema changes (no DB exists today — surface if introducing one).
- `rm -rf` outside `node_modules/`, `dist/`, or `.wrangler/`.
- Re-introducing `VITE_APP_TOKEN` as a build env var.
- Force-push or history-rewrite on `main`.
- Worker deploy when `git status` is unexpectedly clean after recent edits (suspected Dropbox revert).

---
> Source: [gwaiblade/congressintel](https://github.com/gwaiblade/congressintel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
