## factory

> 독립적으로 호출 가능한 9개 스킬. 순서 없이 필요한 스킬을 바로 실행.

# Factory — Software Development Toolbox

독립적으로 호출 가능한 9개 스킬. 순서 없이 필요한 스킬을 바로 실행.

## Quick Start

```
/factory-research    # 시장/기술 리서치
/factory-prd         # 요구사항 정의
/factory-analysis    # 코드/시스템 분석
/factory-architect   # 기술 설계
/factory-ui-design   # UI/UX 디자인
/factory-tdd         # TDD (Red-Green-Refactor)
/factory-develop     # 구현
/factory-e2e         # E2E 테스트
/factory-code-review # 코드 리뷰 + 수용 검증
```

## Skills

| Skill | Description | Output |
|-------|-------------|--------|
| factory-research | 시장/기술 리서치 (단일/멀티 토픽) | `factory/research/` |
| factory-prd | 인터뷰 기반 요구사항 정의 (EARS) | `factory/prd/prd.md` |
| factory-analysis | 코드/시스템 분석, 갭 분석, 3옵션 평가 | `factory/analysis/analysis.md` |
| factory-architect | 기술 설계, ADR, 수용 기준, GO/NO-GO | `factory/architect/architect.md` |
| factory-ui-design | UI/UX 디자인 (Pencil MCP / HTML 폴백) | `factory/ui-design/ui-design.md` |
| factory-tdd | TDD — Red → Green → Refactor | `factory/tdd/` |
| factory-develop | 구현 가이드 — 에스컬레이션, 중복 판정 | `factory/develop/` |
| factory-e2e | E2E 테스트 — 수용 기준 기반 검증 | `factory/e2e/` |
| factory-code-review | 정량 채점 + 90/70/fail 판정 | `factory/code-review/code-review.md` |

## Recommended Flow (Not Enforced)

```
research → prd → analysis → architect → ui-design → tdd → develop → e2e → code-review
```

각 스킬은 독립적이지만, 위 순서로 실행하면 이전 스킬의 산출물을 자연스럽게 참고합니다.

## Artifacts

모든 산출물은 `factory/` 디렉토리에 저장:

```
factory/
  research/     # 리서치 보고서 + 토픽별 서브리포트 + decisions.md
  prd/          # 요구사항 정의서 + decisions.md
  analysis/     # 분석 보고서 + decisions.md
  architect/    # 기술 설계 문서 + 리서치 버퍼 + decisions.md
  ui-design/    # UI 디자인 명세 + decisions.md
  tdd/          # 테스트 플랜, 사이클 기록 + decisions.md
  develop/      # 구현 노트 + decisions.md
  e2e/          # E2E 테스트 리포트 + decisions.md
  code-review/  # 채점 매트릭스 + 판정 + decisions.md
```

## Settings

- `settings/rules/` — EARS format, gap analysis framework, design review checklist
- `settings/templates/` — factory-analysis, factory-architect, decision-agenda 템플릿

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JongHyunLee84)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JongHyunLee84)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
