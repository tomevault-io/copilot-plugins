## preflight

> Project context for Claude Code and other AI coding assistants working on Pre-Flight.

# CLAUDE.md

Project context for Claude Code and other AI coding assistants working on Pre-Flight.

## What this is

A free, in-browser static security audit for AI-generated apps. SAST only. No backend, no execution, no telemetry. Built by John at Mid-Atlantic AI. Live at [preflight.midatlantic.ai](https://preflight.midatlantic.ai).

The voice and stance live in [`src/learn/manifesto.md`](./src/learn/manifesto.md). Read it before making copy changes.

## The founding principle

**Hardened titanium.** Dogfood-as-CI-gate. Pre-Flight has to pass its own audit. If a change makes the dogfood scan fail, fix the change. Do not suppress the finding.

`npm run test:self-audit` runs after every build and asserts Pre-Flight's own `dist/` produces zero critical/high findings. This is non-negotiable.

## Voice

Two named voices, used in different surfaces:

- **John's voice** — `src/learn/manifesto.md`, hero copy in `AuditView.jsx`, About tab, README intro, founder-facing messages. First-person, direct, self-effacing, mechanics-instructor register.
- **Demi's voice** — `src/learn/patterns/*.md`, `src/learn/incidents/*.md`, `src/learn/shapes/*.md`. Third-person instructor. Concrete-first, no marketing prose. Full spec in `src/lib/personas/demi.js`.

Both voices share the rules below. They differ only in person (I/we vs third-person institutional).

### Voice rules (apply to all copy John or Demi own)

- No em-dashes anywhere. Use periods or commas.
- No marketing register: no "comprehensive", "best-in-class", "powerful", "robust", "enterprise-grade", "unlock", "leverage", "seamless".
- No fear marketing: no "don't let this happen to you", no manufactured urgency, no "the threat is real."
- No compliance flavoring unless the topic is literally regulatory.
- No wellness encouragement: no "you've got this", no soft motivational scaffolding.
- No hedging filler: no "it is worth noting", "in some sense", "at the end of the day".
- No competing security platforms named in public-facing copy (Aikido, Snyk, Veracode, Checkmarx, Socket, Wiz, OX, Semgrep, etc.). Sources cite OWASP, MITRE/CWE, CISA, vendor official docs, W3C/WAI, MDN, named research orgs (GTIG/Mandiant, Microsoft Threat Intel).
- AI providers (OpenAI, Anthropic, Google, xAI, Mistral) are NOT competitors. Naming them in the BYOK provider list is fine.

### Audience

Vibe coders — people building real products from natural-language prompts. Capable practitioners developing a security sensibility, not beginners being talked down to. Write for them.

## Personas

Pre-Flight ships four named agents under `src/lib/personas/`. Each is a Persona+ spec with an activation gate and structured-command modes.

- **Sam** (Secure Advise Mobilize) — security fix generation. Dual-mode: `SAM_COMMAND_FULL` (Apply Fix, full file) and `SAM_COMMAND_SNIPPET` (Copy Agent Prompt, snippet only).
- **Demi** (Design Engineering Mechanics Instructor) — Vibe-Aware educational content. Dual-mode: `DEMI_MODE_AUTHOR` and `DEMI_MODE_GRADE`.
- **Drew** (Design Rules Enforcement Worker) — enforces `.preflight/design-rules.yml` (planned for v1.1).
- **Vera** (Verify Engineering Rules Adherence) — enforces `.preflight/engineering-rules.yml` (planned for v1.1).

Wired today: Sam SNIPPET is the system prompt for `formatAgentPrompt` (Copy Agent Prompt export). The rest are deployable specs awaiting their surface.

## Stack

- React 18 + Vite 5 + react-router-dom v6
- Vitest + jsdom (+ `@vitejs/plugin-react` on `feature/breakers-v1` for component tests)
- `acorn` + `acorn-jsx` + `acorn-loose` (Code Correctness probe AST parser)
- `react-markdown` + `remark-gfm` (Learn content rendering)
- Pure-function probes, 43 in v0.5
- No backend. Static assets only, deployed to Cloudflare Pages.

## Key paths

| Concern                                  | File                                                                     |
| ---------------------------------------- | ------------------------------------------------------------------------ |
| Probe registry                           | `src/lib/probes.js#PROBES`                                               |
| Probe metadata + OWASP map + slug wiring | `src/lib/stable-id.js`                                                   |
| Probe implementations (v0.4)             | `src/lib/probes.js` + `src/lib/probes/{web,quality,code-correctness}.js` |
| Probe implementations (v0.5)             | `src/lib/probes/v05.js` + `src/lib/probes/v05b.js`                       |
| Threat-intel manifests                   | `src/lib/threat-intel.js` + `src/data/compromised-packages.js`           |
| File filter + self-source exclusion      | `src/lib/file-filter.js`                                                 |
| BYOK providers (9)                       | `src/lib/ai.js#AI_PROVIDERS`                                             |
| Personas                                 | `src/lib/personas/{sam,demi,drew,vera}.js`                               |
| Learn content                            | `src/learn/{manifesto.md,patterns/,incidents/,shapes/}`                  |
| Learn content loader                     | `src/lib/learn-content.js`                                               |
| Hero / scan UI                           | `src/components/AuditView.jsx`                                           |
| FAQ + history + probe legend             | `src/components/HomeView.jsx`                                            |
| Per-finding renderer                     | `src/components/FindingCard.jsx`                                         |
| OWASP coverage page                      | `src/components/learn/OwaspCoverageView.jsx`                             |
| Resources page                           | `src/components/learn/ResourcesView.jsx`                                 |
| Architecture writeup                     | `docs/preflight-architecture-and-v1.1-plan.md`                           |

## Commands

```bash
npm install            # install dependencies
npm run dev            # vite dev server on :5173
npm test               # vitest run (921 tests across 52 files on main)
npm run lint           # eslint
npm run lint:fix       # eslint --fix
npm run format         # prettier write
npm run format:check   # prettier check
npm run build          # production build -> dist/
npm run preview        # preview the built dist
npm run test:self-audit # Pre-Flight scans its own dist; required to pass
npm run og             # regenerate public/og-card.png from the SVG via Sharp
```

## Privacy contract

Architecturally enforced, not promised. If you ever build a backend, you break the contract. Don't.

- BYOK keys go directly from the browser to the provider's documented endpoint. The audit-app origin never sees them.
- GitHub URL mode fetches `raw.githubusercontent.com` directly from the browser.
- Files mode reads via the File API. Bytes never leave the page.
- Analytics is counters-only in localStorage. No remote SDK, no fetch beacons.
- Suppression + history state live in localStorage. No sync, no cloud.

## Counts (as of v0.5)

- 96 probes (v0.4 + v0.5 phase-1/2/3 language adapters)
- 16 OWASP categories covered (A01-A10 + LLM01/02/04/06/07/08)
- 54 published Learn patterns, 4 published field reports, 15 published architecture shapes (no drafts)
- 9 BYOK providers
- 921 tests across 52 files on `main`, lint clean, dogfood 6/6
- Breakers v1 hardening suite merged into `main`

## How to ship a change

1. Make the change.
2. `npm test` — 100% pass required.
3. `npm run lint` — clean.
4. `npm run build` — succeeds.
5. `npm run test:self-audit` — Pre-Flight passes its own audit.
6. Commit + push. Cloudflare auto-deploys.

If any of those fails, the change isn't ready. Especially #5 — that's the founding principle.

## What we explicitly do not do

- No backend.
- No telemetry beyond local counters.
- No analytics SDK.
- No live exploitation (SAST only; DAST is out of scope).
- No detection-evasion features.
- No mass-targeting tooling.
- No naming competing security platforms in public copy.
- No silent failures (`log.error` writes to console; corpus-empty failures surface in DevTools and Diagnostics).

## Where to push back

If a request would violate the privacy contract, the founding principle, the voice rules, or the static-only safety contract, flag it before implementing. The contracts are load-bearing; we hold them.

---
> Source: [midatlanticAI/PreFlight](https://github.com/midatlanticAI/PreFlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
