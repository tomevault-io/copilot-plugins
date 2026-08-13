## powerbi-gantt-chart

> **Do not create, restore, or enable GitHub Actions workflows or any other hosted CI/CD in this repository. This rule has no exceptions.**

# Repository instructions for coding agents

## Never add CI/CD to this repository

**Do not create, restore, or enable GitHub Actions workflows or any other hosted CI/CD in this repository. This rule has no exceptions.**

This account has no GitHub Actions credits. A workflow file starts billable runs the moment it is pushed, and pushes happen on every branch update, so adding one costs the owner money immediately and silently.

Concretely, do not:

- create or edit anything under `.github/workflows/`
- add `.gitlab-ci.yml`, `azure-pipelines.yml`, `.circleci/config.yml`, `Jenkinsfile`, `.travis.yml`, `appveyor.yml`, or any equivalent
- add `on: push`, `on: pull_request`, or scheduled automation of any kind
- enable GitHub Actions in repository settings, or recommend enabling it
- add a status badge that implies hosted CI

This applies even when a task appears to require it. Requests such as "make validation reproducible", "gate the release", "prove the audit cannot be masked", or "add certification checks" must **not** be satisfied with hosted CI.

## Satisfy those requests locally instead

Use an npm script that runs the whole gate on the developer's machine, and document the exact command in the README. Local execution is the only supported validation mechanism here.

The validation entry point for this repository is:

```
npm run validate:certification
```

Run it from a clean `npm ci` before claiming any change is ready. If a gate needs to be un-maskable, enforce it with a test that shells out to the real command and asserts its exit status, not with a CI job.

## Power BI certification context

This repository ships a Power BI custom visual submitted for Microsoft's Power BI Certified badge, so also keep the following intact:

- Preserve the existing visual GUID. Never regenerate it.
- Advance the four-part version consistently across `package.json`, `pbiviz.json`, docs, and version assertions whenever packaged content changes.
- Keep `powerbi-visuals-api` and `powerbi-visuals-tools` pinned to exact versions, and keep `pbiviz.json` `apiVersion` in step with the installed API package.
- `npm audit` must report zero critical, high, and moderate advisories. Microsoft rejects certification otherwise.
- Keep `privileges` and `externalJS` empty, and keep the bundle free of `fetch`, `XMLHttpRequest`, `WebSocket`, `innerHTML`, `eval`, `new Function`, and dynamic `import()`.
- Do not commit generated output such as `dist/` or `.tmp/`.
- Note that `powerbi-visuals-api` exports a host contract of `major.minor.0`, so a packaged archive reporting a `.0` patch while the source pins a higher patch is expected and must not be "fixed".
- The lowercase `certification` branch is what Microsoft's reviewers read. Do not force-push or rewrite it without being asked.

---
> Source: [garrett-hamers/powerbi-gantt-chart](https://github.com/garrett-hamers/powerbi-gantt-chart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
