## aitraffic

> Build `aitrafficanalytic.com` as a useful, source-backed publication with practical AI traffic tools. Read [README.md](README.md), [HANDOFF.md](HANDOFF.md), and [docs/delivery-plan.md](docs/delivery-plan.md) first. Before making product or routing decisions, read [docs/product-and-content.md](docs/product-and-content.md), [docs/architecture.md](docs/architecture.md), and [docs/seo-agent-playbook.md](docs/seo-agent-playbook.md). Tool work also requires [docs/tools.md](docs/tools.md).

# Agent instructions

## Mission and starting point

Build `aitrafficanalytic.com` as a useful, source-backed publication with practical AI traffic tools. Read [README.md](README.md), [HANDOFF.md](HANDOFF.md), and [docs/delivery-plan.md](docs/delivery-plan.md) first. Before making product or routing decisions, read [docs/product-and-content.md](docs/product-and-content.md), [docs/architecture.md](docs/architecture.md), and [docs/seo-agent-playbook.md](docs/seo-agent-playbook.md). Tool work also requires [docs/tools.md](docs/tools.md).

These are a self-contained starting specification. Record material changes and reasons in the research log; do not silently replace documented decisions. Current user instructions take precedence over this starting brief.

## Scope and safety

- Work inside this project. Treat `../multipass` as read-only reference material; do not deploy or alter that site.
- Inspect the working tree before editing; preserve any existing user changes. If this folder is not yet a Git repository, do not accidentally use the sibling repository.
- Never print, copy, or persist tokens, keys, credentials, raw private logs, or personal analytics data in code, examples, prompts, fixtures, reports, or commits. Use synthetic fixtures.
- Do not copy `.env`, Cloudflare bindings, account identifiers, Plausible script IDs, screenshots, or raw CSV exports from another project.
- Do not create remote repositories, change visibility, configure DNS, deploy, install global/system software, use paid research APIs, or enable scheduled external jobs without task-specific authorization. Local application dependencies and tests are normal implementation steps once the user starts the build.
- Do not run broad/destructive deletes. Do not launch subagents unless the current user request explicitly authorizes delegation.
- If launch credentials or owner choices are missing, finish local implementation and tests, document the narrow blocker, and hand back a launch checklist. Never claim a local preview is production.

## Product and evidence rules

- Use English. Be pragmatic and readable; avoid generic AI-generated marketing copy and unverifiable AEO promises.
- Prefer primary technical documentation and reproducible tests. Check current vendor behavior before making time-sensitive claims.
- Separate reported crawler identity, verified identity, request purpose, human referrals, and citations. Unknown is a valid result.
- Every public quantitative claim needs a measurement window, unit, denominator, method, and limitations. Synthetic examples must be labeled in the UI and downloads.
- Drafts and unreviewed claims must not appear as published authority. Do not fabricate author credentials, customer examples, keyword volumes, quotations, or results.
- User files remain local to the browser/CLI in the initial tools. Never send their content, paths, or query strings to analytics.

## Implementation rules

- Start static-first, with minimal client JavaScript and no operational datastore. Do not transplant MultiPass Rank's large matrix infrastructure.
- Use one content model and route registry for HTML, Markdown, JSON metadata, canonical links, redirects, and sitemap membership.
- Use real server-rendered links, meaningful headings, visible sources and review dates, and mobile-safe layouts.
- Keep explicit `.md` URLs working. Native Markdown negotiation must not leak Markdown into cached browser responses or HTML into Markdown requests.
- Unrecognized routes return genuine 404s. Use reviewed aliases, never broad fuzzy redirects or an SPA success response for unknown paths.
- Choose currently compatible dependency versions at implementation time; lock them. Prefer the smallest implementation that meets the acceptance tests.
- Use targeted patches, run relevant checks, and report actual results and remaining risks. Do not report planned npm scripts as existing until implemented and executed.

## Handoff discipline

At the end of a substantial pass, update the milestone state in [docs/delivery-plan.md](docs/delivery-plan.md) and append dated observations to [docs/research-log.md](docs/research-log.md). Report what works, what was tested, what is still a draft, and the next bounded task. This project must remain transferable without access to a conversation transcript.

---
> Source: [chatcode-lab/aitraffic](https://github.com/chatcode-lab/aitraffic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
