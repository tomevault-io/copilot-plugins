## shieldx

> - License: Apache 2.0

# ShieldX — LLM Prompt Injection Defense System

## Project
- npm: @shieldx/core
- License: Apache 2.0
- Stack: TypeScript strict, Node.js 20+, PostgreSQL 17 + pgvector, Vitest
- Architecture: 10-layer defense pipeline + self-evolution engine
- Philosophy: Local-first, zero mandatory cloud, self-evolving

## Commands
- `npm run build` — Build with tsup (CJS + ESM + DTS)
- `npm run dev` — Watch mode build
- `npm test` — Run tests with vitest
- `npm run test:coverage` — Coverage report (target: 80%+)
- `npm run typecheck` — Type checking
- `npm run db:migrate` — Run database migrations
- `npm run db:seed` — Seed initial patterns (500+)
- `npm run benchmark` — Performance benchmarks
- `npm run self-test` — Red team self-testing

## Code Style
- TypeScript strict mode, no `any` except explicitly marked with `// eslint-disable-next-line`
- Immutable data patterns — return new objects, never mutate
- All async operations must have proper error handling
- All public methods must have JSDoc documentation
- Files < 400 lines, functions < 50 lines
- No raw input stored in database — always SHA-256 hashed

## Architecture
- 10 defense layers (L0-L10), each independently toggleable
- Kill chain mapping: Schneier 2026 Promptware Kill Chain (7 phases)
- Self-evolution: GAN red team, drift detection, active learning, federated sync
- Compliance: MITRE ATLAS, OWASP LLM Top 10 2025, EU AI Act

## Performance Targets
- L0 (Preprocessing): <0.5ms
- L1 (Rules): <2ms
- L2 (Classifier): <10ms
- Full pipeline (L0-L9): <50ms
- Embedding scan: <200ms (Ollama local)

## Testing
- Vitest with v8 coverage
- Attack corpus: 13 JSON files, 500+ patterns each
- Benchmarks: ASR, latency, PINT, AgentDojo, false-positive rate
- Coverage target: 80%+ global

## Git
- Gitea: gitea.context-x.org/rene/shieldx
- Conventional commits: feat, fix, refactor, docs, test, chore, perf
- No Co-Authored-By headers

---
> Source: [renefichtmueller/ShieldX](https://github.com/renefichtmueller/ShieldX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
