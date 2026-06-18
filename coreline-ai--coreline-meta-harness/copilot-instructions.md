## coreline-meta-harness

> 작업을 해석하고, 필요한 도메인 팩과 에이전트/스킬 구성을 결정한 뒤, 스킬을 먼저 설계·바인딩하고, 실제 구현 에이전트 팀 runtime으로 handoff 가능한 Codex용 메타 하네스 패키지를 정의하는 메타 하네스 스킬.


# Coreline Harness

범용 요청을 **판단·계약·패키지 정의** 구조로 정리하는 **Codex용 문서형 메타 하네스**.

## Purpose

- 입력 작업에 맞는 메타 하네스 패키지를 설계한다.
- 생성 대상 하네스의 도메인 팩과 에이전트 구성을 결정한다.
- 각 에이전트가 사용할 스킬 세트를 먼저 정의하고 바인딩한다.
- 실제 구현 에이전트 팀 runtime으로 handoff 가능한 문서와 샘플 패키지를 만든다.

## Core Model

메타 하네스는 아래 3층으로 동작한다.

1. **입력**
   - 자유 사용자 요청
   - `refs/*` 계약 문서
   - fixture (`test-run/*`, `generated/*`)
2. **판단**
   - request normalization
   - domain pack 선택
   - generic role 선택
   - concrete agent 구체화
   - skill_builder 선택
   - extension/orchestrator binding
3. **출력**
   - binding map
   - artifact contract
   - validation contract
   - completion state 판단 근거
   - handoff 가능한 문서 패키지

## Input Boundary

이 스킬이 읽는 핵심 입력은 아래다.

- 사용자 요청
- `refs/domain_packs/*`
- `refs/agents/*`
- `refs/skill_builders/*`
- `refs/skills/*`
- 필요 시 fixture 문서 (`test-run/*`, `generated/*`)

이 단계에서는 runtime 상태나 실행 로그를 핵심 입력으로 취급하지 않는다.

## Decision Boundary

이 스킬이 직접 결정하는 것은 아래다.

- 어떤 domain pack 인터페이스가 맞는가
- 어떤 generic role이 필요한가
- 각 generic role이 어떤 concrete agent로 구체화되는가
- 어떤 skill_builder와 extension skill이 필요한가
- 어떤 artifact와 validation contract가 필수인가

즉, 이 스킬의 핵심은 **실행이 아니라 판단**이다.

## Output Boundary

이 스킬의 직접 출력은 아래다.

- normalized request 해석
- selected domain pack
- generic role / concrete agent 구조
- skill binding 결과
- artifact contract
- validation contract
- completion state를 판정할 수 있는 문서 패키지

실제 코드 실행, worker 상태 수렴, runtime 제어는 직접 출력이 아니다.

## Runtime Boundary

이 스킬은 **실행 엔진**이 아니다.

- `coreline-harness` 는 메타 하네스다.
- 실제 구현 에이전트 팀의 생성/실행/상태 관리는 **`https://github.com/coreline-ai/coreline-agent-team`** 이 담당한다.
- 이 스킬의 목적은 runtime을 다시 만드는 것이 아니라, **어떤 팀을 어떤 스킬로 구성해야 하는지 정의하고 handoff 문서를 준비하는 것**이다.

## Design Stance

1. 진입점은 단일 `SKILL.md` 이다.
2. 상세 규칙은 `refs/` 문서를 참조한다.
3. 최종 목표는 Codex용 메타 하네스 패키지이며, 현재 `.claude/` 구조는 참고용 샘플/호환 레이어로 취급한다.
4. 에이전트 생성보다 스킬 생성과 바인딩이 먼저다.
5. 실제 구현 에이전트 팀은 `coreline-agent-team` 기반 Codex runtime으로 handoff 한다.

## Generic Role vs Concrete Agent

- **generic role**: 도메인에 독립적인 추상 역할  
  예: spec, architecture, implementation, review, operations
- **concrete agent**: 특정 domain pack이 선택한 실제 역할 인스턴스  
  예: `spec-analyst`, `solution-architect`, `frontend-builder`

메타 하네스는 먼저 generic role을 판단하고, 그 다음 domain pack 규칙에 따라 concrete agent를 구체화한다.

## Input Interpretation

사용자 요청에서 아래를 추출한다.

- domain
- goal
- complexity
- desired_outputs
- collaboration_need
- validation_need

## Target Outputs

이 스킬이 직접 책임지는 출력은 아래다.

- domain pack 선택 결과
- generic role / concrete agent 설계
- skill builder 선택 결과
- extension skill / orchestrator skill 정의
- binding map
- artifact contract
- validation contract
- completion state 판단 근거
- runtime handoff용 문서와 샘플 패키지

## Domain Pack Selection Rules

도메인별 구체 규칙은 `refs/domain_packs/domain_pack_interface.md` 와 `refs/domain_packs/*.md` 를 함께 참조한다.

- 현재 기본 포함 예시는 `fullstack-webapp-domain-pack.md`, `seo-optimization-domain-pack.md` 다.
- domain pack은 특정 도메인 구현물이 아니라 **플러그인 인터페이스**를 따르는 규칙 묶음으로 본다.
- 상위 카테고리는 문제 영역을 넓게 묶는 분류일 뿐이다.
- 실제로 선택하고 바인딩하는 단위는 concrete domain pack이다.
- 따라서 같은 상위 카테고리 안에도 여러 concrete domain pack이 공존할 수 있다.

## Agent Selection Rules

에이전트는 `refs/agents/*.md` 를 참조하여 선택한다.

- 요구 정리 필요 → `spec-agent.md`
- 구조 설계 필요 → `architecture-agent.md`
- 구현/생성 필요 → `implementation-agent.md`
- 검증/리뷰 필요 → `review-agent.md`
- 배포/운영 필요 → `operations-agent.md`

## Skill Builder Selection Rules

각 에이전트에 대해 대응되는 `refs/skill_builders/*.md` 를 먼저 선택한다.

- spec-agent → `spec_skill_builder.md`
- architecture-agent → `architecture_skill_builder.md`
- implementation-agent → `implementation_skill_builder.md`
- review-agent → `review_skill_builder.md`
- operations-agent → `operations_skill_builder.md`

중요:
- skill_builder의 `Required Skill Categories`는 **internal decomposition**이다.
- 기본적으로 여러 category는 **하나의 composite extension skill**로 조립된다.
- 즉 category 개수와 runtime-visible extension skill 개수는 기본적으로 1:1이 아니다.
- 예외 분리 기준은 `refs/skill_builders/skill_builder_mapping_rules.md`를 따른다.

## Skill Generation Rules

개별 스킬 정의는 `refs/skills/*.md` 를 따른다.

- 공통 계약: `skill_definition_contract.md`
- 오케스트레이터 스킬 계약: `orchestrator_skill_contract.md`
- 에이전트 확장 스킬 계약: `agent_extension_skill_contract.md`

## Binding Rules

1. 에이전트를 만들기 전에 해당 에이전트용 스킬 세트를 만든다.
2. 생성된 스킬의 입력/출력 흐름을 확인한다.
3. 바인딩 가능한 스킬만 에이전트 정의에 연결한다.
4. 바인딩이 완료된 뒤 에이전트 정의와 샘플 패키지 파일을 생성한다.

표현 주의:
- `BINDING_MAP.md`는 가독성을 위해 agent 중심 컬럼 순서를 사용할 수 있다.
- 하지만 메타 하네스의 결정 순서는 **skill_builder 선택 → skill generation / binding → agent definition** 이 먼저다.

## Runtime Handoff Rules

실제 구현 에이전트 팀 runtime은 아래 기준을 따른다.

- Codex에서 하네스 부분의 실제 구현 에이전트 팀은 **`coreline-agent-team`** 을 사용한다.
- runtime 관련 운영 흐름은 `doctor`, `app`, `run`, `attach`, `watch`, `tui` 관점을 따른다.
- 메타 하네스는 runtime 입력 계약과 팀 구성 규칙을 준비하고, 실행 자체는 runtime에 위임한다.

## Packaging Rules

생성 산출물은 `generated/<harness-slug>/` 아래에 정리한다.

현재 v1 단계에서 패키지는 아래 층을 가질 수 있다.

- 루트 설명 문서 (`README.md`, `BINDING_MAP.md`, runtime notes)
- 샘플 결과 구조
- 필요 시 `.claude/` 하위의 참고용 에이전트/스킬 패키지

중요:

- `.claude/` 구조는 현재 **참고용 샘플/호환 레이어**다.
- 최종 핵심은 Codex 사용자와 `coreline-agent-team` runtime 사이의 handoff 문서 정합성이다.
- 샘플 패키지와 fixture는 canonical 정의 문서가 아니라 **계약 검증용 예시**다.

## Execution Flow

1. 작업 해석
2. request normalization
3. domain pack 선택
4. generic role 선택
5. concrete agent 구성 정의
6. 각 에이전트용 skill_builder 선택
7. 스킬 세트 생성
8. 생성된 스킬 바인딩
9. orchestrator / extension skill 정의
10. artifact / validation contract 구성
11. fixture 또는 handoff 문서 패키지 구성
12. completion state 검토

## Completion Criteria

- 메타 하네스의 입력/판단/출력 경계가 명확하다.
- 선택된 domain pack과 에이전트 구성이 명확하다.
- 선택된 모든 에이전트에 스킬 세트가 존재한다.
- 모든 스킬 입출력 흐름이 끊기지 않는다.
- 문서 패키지가 binding map, artifact contract, validation contract를 포함한다.
- 실제 구현 에이전트 팀 runtime이 `coreline-agent-team` 임이 문서상 명시된다.

## Not In Scope

- runtime 엔진 자체 구현
- direct API 기반 별도 런타임 설계
- 외부 저장소 수정
- 다중 도메인 동시 완성

## Primary References

- 상위 목적 문서: `dev-plan/PRD.qmd`
- 종료된 canonical workstream: `dev-plan/implement_20260406_220119.md`
- 확장 규칙 보강 supplement: `dev-plan/implement_20260411_111211.md`
- 전체 구조: `docs/harness-architecture.md`
- 문서 계약: `docs/meta-harness-document-contracts.md`
- completion state 모델: `docs/meta-harness-completion-state-model.md`
- execution backlog: `docs/meta-harness-execution-backlog.md`
- Codex 사용 흐름: `docs/codex-cli.qmd`
- 도메인 규칙: `refs/domain_packs/`
- domain pack 인터페이스: `refs/domain_packs/domain_pack_interface.md`
- 에이전트 기준: `refs/agents/`
- 스킬 세트 기준: `refs/skill_builders/`
- 개별 스킬 기준: `refs/skills/`
- 샘플 결과물: `generated/sample-fullstack-webapp/`, `generated/sample-seo-optimization/`

참고:
- canonical contract 판단은 `PRD.qmd` → `implement_20260406_220119.md` → `implement_20260411_111211.md` → `SKILL.md` → `refs/*` 순서를 우선한다.
- `README.md`, `AGENTS.md`는 중요한 entry 문서지만 contract source of truth 자체보다는 안내/운영 가이드 성격으로 읽는다.

---
> Source: [coreline-ai/coreline-meta-harness](https://github.com/coreline-ai/coreline-meta-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
