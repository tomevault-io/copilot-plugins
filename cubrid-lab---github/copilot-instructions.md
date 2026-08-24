## github

> This repository holds organization-wide community health files for `cubrid-labs`.

# AGENTS.md

## Purpose
This repository holds organization-wide community health files for `cubrid-labs`.

## Read First
- `README.md`
- `CONTRIBUTING.md`
- `SECURITY.md`
- `SUPPORT.md`
- `docs/agent-playbook.md`

## Working Rules
- Treat this repo as policy and template infrastructure, not product code.
- Keep contributor-facing guidance consistent across files.
- When behavior or policy changes, update every affected document in the same change.
- Prefer small edits that preserve existing wording unless a policy change is intentional.

## Development Workflow (cubrid-labs org standard)

All non-trivial work across cubrid-labs repositories MUST follow this 4-phase cycle:

1. **Oracle Design Review** — Consult Oracle before implementation to validate architecture, API surface, and approach. Raise concerns early.
2. **Implementation** — Build the feature/fix with tests. Follow existing codebase patterns.
3. **Documentation Update** — Update ALL affected docs (README, CHANGELOG, ROADMAP, API docs, SUPPORT_MATRIX, PRD, etc.) in the same PR or as an immediate follow-up. Code without doc updates is incomplete.
4. **Oracle Post-Implementation Review** — Consult Oracle to review the completed work for correctness, edge cases, and consistency before merging.

Skipping any phase requires explicit justification. Trivial changes (typos, single-line fixes) may skip phases 1 and 4.

## Validation
- Manually review Markdown rendering and link targets.
- If examples or command snippets change, verify they still match current org conventions.

## Project Context — Performance Loop System

> **Board**: [CUBRID Ecosystem Roadmap](https://github.com/orgs/cubrid-labs/projects/2)

### What This Is

A continuous **Performance Loop** across the CUBRID ecosystem:
benchmark → profile → optimize driver/ORM → re-benchmark → CI-verified regression prevention.

### Core Repos

| Repo | Role | Current State |
|------|------|---------------|
| `cubrid-benchmark` | Benchmark suite (Python/TS/Go × CUBRID/MySQL) | Tier 0+1+Extended, CI, GitHub Pages — **most mature** |
| `pycubrid` | Python driver (PEP 249) | v1.5.0, published PyPI — **stable 1.x, async TLS + STARTTLS hardening, transport contract lock, 1.x compat-check CI gate; 2 perf cycles complete, 19% fetch improvement** |
| `sqlalchemy-cubrid` | SQLAlchemy 2.0–2.2 dialect | v1.5.0, published PyPI — **stable 1.x, SQLAlchemy 2.1/2.2 forward-compat shims, async integration stability; query compilation optimized** |
| `cubrid-cookbook` | Runnable examples across all languages | Broad coverage — **expected outputs not standardized** |

### Supporting Repos

`cubrid-client` (TS, v1.1.0), `cubrid-go`, `cubrid-rs`, `drizzle-cubrid`, `gorm-cubrid`, `sea-orm-cubrid`

### Execution Phases

| Phase | Period | Focus |
|-------|--------|-------|
| R0: Baseline Reset | Week 1-2 | Scope reset, baseline recording, plan rewrite |
| R1: Measurement | Week 3-5 | Tier 2 ORM benchmarks, reproducibility policy |
| R2: Python Optimization | Week 6-10 | pycubrid profiling, serialization + cursor fetch optimization |
| R3: ORM Validation | Week 11-14 | sqlalchemy-cubrid optimization, 2nd pycubrid cycle |
| R4: Stability | Week 15-18 | cubrid-client resilience (nice-to-have) |
| R5: CI & Regression | Week 19-22 | Compare script, cookbook standardization |
| R6: Finalization | Week 23-32 | Evidence pack, demo, documentation |

### Hero Metric

**Python driver MySQL 대비 성능 갭 감소** — before/after 수치가 핵심 스토리.

### Decision Gate (Week 8)

pycubrid 최적화 결과 +10% 이상이면 계속, 미만이면 Go/TS 성과 중심으로 내러티브 피벗.

### Narrative

"Build Once, Improve Continuously" — 매일 밤 자동 벤치마크, PR마다 회귀 감지, 재현 가능한 one-command 데모.

### Must-Have Deliverables

1. pycubrid before/after 성능 수치
2. Tier 2 ORM 벤치마크
3. Performance Loop 2회 이상 완주 증명
4. one-command 재현 데모 (`docker compose up && make demo`)
5. 증거팩 (차트, 보고서)

## Team Coordination (2-Person Team)

### Rules

1. **Issue assignment is mandatory before starting work.** GitHub issue에 본인을 assign한 뒤 작업 시작. assign 안 된 이슈는 누구도 작업 중이 아님.
2. **한 이슈에 한 사람.** 같은 이슈에 두 사람이 동시에 작업하지 않는다.
3. **같은 파일 수정 금지.** 서로 다른 이슈더라도 같은 파일을 동시에 수정하면 충돌. 의심되면 먼저 PR 올린 사람이 우선.
4. **Branch naming**: `{username}/{issue-number}-short-description` (예: `jay/19-profiling`)
5. **PR 리뷰 필수.** 본인 PR은 상대방이 리뷰. self-merge 금지.
6. **Phase 순서 존중.** Backlog에 있는 이슈는 해당 Phase의 선행 이슈가 Done이 될 때까지 착수하지 않는다.

### Parallel Work Guidelines

동시 작업 가능한 조합 (충돌 없음):
- Person A: `cubrid-benchmark` repo / Person B: `pycubrid` repo
- Person A: R1 measurement 이슈 / Person B: R2 profiling 준비 (단, R2 본격 최적화는 R1 완료 후)
- Person A: 벤치마크 인프라 (#6, #9) / Person B: profiling 인프라 (#14, #15)

동시 작업 **금지** 조합:
- 같은 repo의 같은 모듈 (예: 두 사람이 동시에 pycubrid/protocol.py 수정)
- upstream/downstream 동시 수정 (예: pycubrid API 변경 + sqlalchemy-cubrid에서 해당 API 사용)

### Daily Sync

- 작업 시작 전 GitHub Project Board 확인 → In Progress 상태 업데이트
- 작업 완료 시 즉시 PR 생성 + Done 이동

---
> Source: [cubrid-lab/.github](https://github.com/cubrid-lab/.github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
