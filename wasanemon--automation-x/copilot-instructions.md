## automation-x

> - Do not put secrets in code, tests, fixtures, docs examples, or logs.

# AGENTS.md

## Project Rules

- Do not put secrets in code, tests, fixtures, docs examples, or logs.
- Use environment variables for all credentials and service URLs.
- Never implement automated replies, mentions, likes, follows, retweets, DMs, or keyword-triggered outreach.
- X API usage in this MVP is read-only and limited to owned post lookup and metrics collection.
- X metrics collection in this MVP is public metrics only; private/non-public metrics are future work.
- X post ID reconciliation uses normalized text similarity plus scheduled/created time proximity.
- X API credentials may be missing; app startup and Postiz scheduling must continue to work.
- Mock Postiz, X API, and any other external HTTP calls in tests.
- Keep retries bounded and timeouts explicit.
- `GET /health` may stay unauthenticated; protect other endpoints with `GROWTH_AGENT_API_KEY` unless `TESTING=true`.
- Default scheduling to dry-run. Only call Postiz when `SCHEDULING_DRY_RUN=false`.
- Treat `POSTIZ_BASE_URL` as the complete Public API base URL; do not append `/api/public/v1` or `/public/v1`.
- Before scheduling, enforce duplicate or near-duplicate prevention.
- Do not schedule the same draft twice.
- URL-bearing drafts and posts must set `has_url=true`.
- If `OWNED_DOMAINS` is empty, URL-bearing drafts require human approval.
- External URLs, shortened URLs, pricing/legal language, and strong claims require human approval.
- High-risk drafts must require human approval before scheduling.
- X metrics credentials may be missing during dry-run and Postiz test scheduling; metrics collection should skip or fail clearly, not break startup.
- Do not collect private/non-public metrics in this MVP; future implementations must not collect them for posts older than 30 days.
- Run `pytest` and `ruff check .` before handing off changes when the environment allows it.

---
> Source: [wasanemon/automation_x](https://github.com/wasanemon/automation_x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
