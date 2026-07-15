## agent-context-template

> YOUR APP NAME is a Brazilian edtech product that turns free YouTube courses into verifiable, paid certificates. It targets two core buyers: university students who need to log "horas complementares" (mandatory extracurricular hours) and early-career professionals trying to strengthen their LinkedIn and résumé.

# AGENTS.md — Your APP Name

## Product Overview

YOUR APP NAME is a Brazilian edtech product that turns free YouTube courses into verifiable, paid certificates. It targets two core buyers: university students who need to log "horas complementares" (mandatory extracurricular hours) and early-career professionals trying to strengthen their LinkedIn and résumé.

**Value proposition:** learning is free and abundant — proof of learning isn't. KipperDev bridges that gap. Students watch curated free courses, pay a small fee in credits to take a timed exam, and walk away with a certificate that has a public verification URL they can share with universities, recruiters, or on LinkedIn.

**Monetization:** credit-based microtransactions via Stripe (card) and AbacatePay (Pix, the dominant Brazilian payment method). Each exam attempt costs credits, so revenue scales with engagement rather than locking value behind a subscription.

**Why it works in this market:** Brazilian undergrads face real regulatory pressure to accumulate extracurricular hours, and juniors face a brutal entry-level job market where a populated LinkedIn is table stakes. Competing platforms either gatekeep the learning (paid courses) or the credential (expensive bootcamps). KipperDev's anti-gatekeeping bet is: keep the course free, charge only for the credential, and make the credential trustworthy enough to matter.

## Documentation Map

Detailed documentation lives in `docs/`; see the [README index](./README.md#-documenta%C3%A7%C3%A3o) for the full categorized list. The most load-bearing references for an agent are:

- `.agents/GLOSSARY.md` — domain glossary
- `.agents/ARCHITECTURE.md` and `docs/API_DOCUMENTATION.md` — system shape and endpoints.


# ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in .agents/PLANS.md) from design to implementation.

---
> Source: [Fernanda-Kipper/Agent-Context-Template](https://github.com/Fernanda-Kipper/Agent-Context-Template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
