## swingbylite

> Build an internship portfolio project from the SwingBy Lite prepaid-pickup specification. Prioritize a usable demo, understandable engineering decisions, honest evidence of testing, and the owner's ability to discuss the work in interviews.

# SwingBy Lite working agreement

## Project goal

Build an internship portfolio project from the SwingBy Lite prepaid-pickup specification. Prioritize a usable demo, understandable engineering decisions, honest evidence of testing, and the owner's ability to discuss the work in interviews.

## Project context

- The owner wants `plans/` kept local. Keep it ignored; do not force-add its files or copy their full contents into public documentation. Public documentation should link to GitHub issues instead.
- `plans/spec.md`, when available locally, is the staged implementation plan and current milestone tracker. On a fresh clone without local plans, use GitHub issues and public documentation for scope.
- `plans/product-spec.md` preserves the original detailed requirements.
- `CONTRIBUTING.md` records the owner's requested GitHub development practice.
- Confirmed: full-stack internships, a GitHub Pages portfolio linking to a separately hosted app, and a simulated-payment demo before payment-provider test mode.
- The owner asks for a popular, internship-relevant stack and can spend at least eight hours, with more as needed. The plan interprets this as eight or more hours per week; a deadline and existing skills remain unspecified.
- `plans/stack-recommendation.md` records the recommended React/Next.js, TypeScript, Express, and PostgreSQL stack and evidence. Initial task details remain open; do not treat draft defaults as owner approval.
- Confirmed hosting approach: $0 during development/early previews, then review a possible paid upgrade when the demo is ready for applications. No paid service or monthly charge has been selected. Provider selection remains Stage 1 work, tracked in https://github.com/YuntongGuo/SwingByLite/issues/2; `plans/hosting-issue.md` retains the draft.
- The owner subsequently requested AWS hosting. Target AWS for the app and keep GitHub Pages for the portfolio. A single EC2 server with containers is proposed; account eligibility, credits, size, region, and full costs remain to be checked. Do not assume AWS will remain free or that a paid-plan upgrade has been authorized.
- The owner has no AWS account. Recommend setup near the first deployment; do not block local application foundation work on account creation.

## Collaboration preferences

- Discuss scope before application implementation. Progress one issue at a time after kickoff.
- Before a new feature, help the owner create an issue with acceptance criteria and choose a small implementation slice.
- Before fixing a newly discovered bug, investigate it and help the owner create a reproducible bug report. Existing issue/PR corrections do not need duplicate issues.
- Let the owner practice GitHub issue creation and PR review/merge unless they explicitly delegate those steps. Honor delegation already given; do not ask repeatedly.
- Prepare concrete issue/PR drafts, and explain the next action instead of silently doing the whole project in one pass.
- Keep issue drafts concise and natural: a short explanation and a few observable completion criteria. Put detailed implementation guidance in the plan rather than repeating it in each issue.
- Complete the agreed issue and its checks, then demonstrate it before starting unrelated work.
- Use real issue numbers, commits, test results, and review history. Never invent activity or claim future features are implemented.
- Update the plan with accepted decisions and completed milestones so future sessions retain context. The owner's later instructions take precedence over this agreement.

## Implementation expectations

Preserve server authorization, explicit state transitions, integer money values, balanced append-only ledger entries, and idempotent actions as their stages are built. External payment simulation must be labeled and use the same domain services as later provider integration. Test mode is not a claim of live payment readiness.

---
> Source: [YuntongGuo/SwingByLite](https://github.com/YuntongGuo/SwingByLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
