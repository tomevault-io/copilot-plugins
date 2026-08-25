## free-account-tool

> This repository is a Chrome Manifest V3 extension for CDK redeem and account status workflows.

# Repository Guidelines

## Project Structure & Module Organization

This repository is a Chrome Manifest V3 extension for CDK redeem and account status workflows.

- `manifest.json` defines permissions, content scripts, side panel, and the background service worker.
- `background.js` wires the service worker; feature code lives under `background/`, especially `background/steps/` and `background/membership/`.
- `content/` contains page automation scripts injected into OpenAI/Auth/iCloud pages.
- `sidepanel/` contains the extension UI, view models, and managers.
- `shared/` contains reusable parsing, API, and state helpers used by background and side panel code.
- `scripts/` contains smoke audits, Node tests, release helpers, and small maintenance utilities.
- `icons/`, `docs/`, `data/`, and `release-artifacts/` hold extension assets, documentation, fixture data, and packaged releases.

## Build, Test, and Development Commands

Use the repository package scripts for the complete gates:

```powershell
npm run syntax
npm test
npm run docs:check
npm run audit
npm run check
```

Use `chrome://extensions` with Developer Mode to load this directory as an unpacked extension. After background changes, reload the extension so the MV3 service worker uses the new code.

## Coding Style & Naming Conventions

Use plain JavaScript and existing browser-extension patterns. Keep indentation at two spaces, prefer `const`/`let`, and use `camelCase` for functions and variables. Browser modules expose namespaces such as `self.MultiPage...`; keep new exports consistent with nearby files. Avoid broad rewrites in large files unless they are part of an explicit refactor.

## Testing Guidelines

Tests use Node’s built-in `node:test` and `node:assert/strict`. Put focused unit tests in `scripts/test-*.cjs`; put static integration checks in `scripts/audit-smoke-tests.mjs`. For feature changes, run the relevant `node --check` files plus at least the matching test script. Add regression coverage for import/export formats, redeem state transitions, and trial eligibility decisions.

### Browser E2E Environment

All automated MV3 browser tests must run in an isolated test browser. Use Puppeteer with its downloaded, version-pinned Chrome for Testing and a fresh temporary profile. On Windows, launch with `pipe: true`; the repository harness may use `--no-sandbox` only for empty-profile local extension pages and must not navigate external sites in that mode. Keep a small bounded retry around browser startup, and report an exhausted startup failure as test-infrastructure failure rather than an extension pass or business failure.

Never connect automated tests to the user's installed Google Chrome, default `User Data`, signed-in profile, cookies, history, or installed extensions. Do not silently fall back to Microsoft Edge, system Chrome, or another browser. Playwright Chromium, Edge, Google Chrome, and Chrome for Testing are distinct targets: print the actual product/version, executable source, and profile type in every E2E result, and name the result accordingly.

The minimum browser E2E must verify the MV3 service worker, Side Panel rendering, expected account/task controls, runtime messaging, diagnostic clipboard export using a page-local clipboard stub, and zero uncaught page errors. Use only fake or empty test data. Close the browser in `finally`, confirm that no test-browser process remains, and never package the downloaded browser with the extension.

Keep the browser harness in repository-owned test code once adopted; do not rely on undocumented one-off commands as release evidence. The evaluated tool choice, compatibility boundaries, and a verified isolated run are recorded in `docs/DEVELOPMENT.md#浏览器-e2e`.

## Issue and Fix Archive

Every confirmed user-reported defect must have a dated section in the current monthly `docs/audit/issue-fix-archive-YYYY-MM.md` and an entry in `docs/audit/issue-fix-index.md`. Use a stable HTML anchor for each section and keep the record in the same commit as the fix whenever practical. Do not create a new Markdown file for each defect.

Each record must include the observed symptom and diagnostic evidence, root cause, affected workflow, implementation changes, safety or compatibility boundaries, regression coverage, actual verification results, and commit or release impact. Record only completed work and real command results; do not leave placeholders or planned test counts.

Redact real emails, passwords, access tokens, verification codes, cookies, API keys, CDKs, proxies, phone numbers, and sensitive URL parameters. Preserve old records as historical evidence; when a later fix supersedes one, add a new record and link both entries instead of silently rewriting the history.

## Documentation Lifecycle

Treat `README.md`, `docs/USER_GUIDE.md`, `docs/DEVELOPMENT.md`, `CHANGELOG.md`, and `docs/audit/issue-fix-index.md` as living entry points. Update the affected entry in the same commit as the behavior change. Do not create a standalone Markdown file for a new proposal, implementation plan, release note, or defect when an existing current document or dated archive owns that responsibility.

Completed design and implementation material belongs in the existing monthly files under `docs/history/`. Confirmed defects belong in the monthly issue archive plus the issue index. Historical files are append-only evidence and must never be edited to masquerade as the current specification.

Run `npm run docs:check` before completion. The audit rejects unexpected Markdown files, deprecated document paths, broken current-document links, version drift, missing history disclaimers, and issue index/archive anchor drift. If a genuinely new documentation category is required, update the audit and this lifecycle rule deliberately in the same commit.

## Commit & Pull Request Guidelines

Recent history uses concise conventional prefixes such as `fix:`, `refactor:`, and `docs:`, plus release commits like `Prepare v1.0.7 release`. Keep commits small and behavior-focused.

Pull requests should include a short behavior summary, test commands run, screenshots for side panel UI changes, and any configuration or release impact. Never commit real emails, passwords, API keys, access tokens, cookies, proxies, phone numbers, or local runtime logs.

## Security & Configuration Tips

Treat `cha.nerver.cc`, Assurivo/iCloud URLs, CDK pools, and access tokens as sensitive operational data. Keep sample values fake, and check `git diff` before committing generated exports or local backup files.

---
> Source: [kui123456789/free-account-tool](https://github.com/kui123456789/free-account-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
