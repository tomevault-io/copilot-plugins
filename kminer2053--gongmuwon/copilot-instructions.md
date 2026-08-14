## gongmuwon

> 이 파일은 이 프로젝트에서 작업하는 에이전트를 위한 기본 운영 가이드다.

# AGENTS.md

## 목적

이 파일은 이 프로젝트에서 작업하는 에이전트를 위한 기본 운영 가이드다.
Claude Code의 `CLAUDE.md`와 같은 개념으로 사용하며, 이 저장소에서의 작업 방식,
역할 분담, 도구 선택 기준, 실행 규칙을 정의한다.

## 설치된 기능

### Plugins

- `Superpowers`: 브레인스토밍, 계획 수립, TDD, 디버깅, 리뷰, 검증,
  브랜치 마무리까지 포함한 실행 규율 계층
- `GitHub`: PR/이슈 확인, 리뷰 대응, CI 점검, 커밋/푸시/PR 흐름 지원

### MCP / Docs / Research

- `Context7`: 최신 라이브러리 및 프레임워크 문서 조회
- `OpenAI Developer Docs`: OpenAI 제품 및 API 공식 문서 조회

## 기본 작업 모델

사용자가 다른 순서를 명시하지 않았다면 아래 흐름을 기본 워크플로로 사용한다.

`Think -> Plan -> Build -> Review -> Test -> Ship -> Reflect`

이 흐름에서 `gstack`은 전체 진행의 주도권과 의사결정 프레임을 담당한다.
`Superpowers`는 각 단계 안에서 코드 품질, 실행 규율, 검증 강도를 높이는 역할을
담당한다.

## 단계별 역할 분담

### 1. Think

주요 담당: `gstack`

이 단계에서 수행할 일:

- 제품 가정 검증
- 범위와 리스크 점검
- 사용자 흐름과 성공 기준 정리

`Superpowers`의 역할:

- 이 단계에서는 보통 개입이 적다
- 다만 요구사항이 모호하거나 탐색이 필요한 경우
  `superpowers:brainstorming`을 먼저 사용해 설계 전 사고를 정리한다

### 2. Plan

주요 담당: `gstack`

이 단계에서 수행할 일:

- 아키텍처 설계
- ERD 및 도메인 모델링
- API 및 연동 구조 설계
- 테스트 전략 정의

`Superpowers`의 역할:

- 승인된 설계를 실행 가능한 작업 계획으로 바꾸기 위해
  `superpowers:writing-plans`를 사용한다
- 계획 문서 안에 TDD 기대치, 리팩터 체크포인트, 의존성 규칙,
  검증 게이트를 명시적으로 추가한다

### 3. Build

주요 담당: `gstack`

이 단계에서 수행할 일:

- 역할 기반 구현 진행
- 기능 개발
- 코드 생성과 진행 리포트 정리

`Superpowers`의 역할:

- 기능 추가나 버그 수정 전 `superpowers:test-driven-development`를 우선 적용한다
- 동작이 불명확하거나 버그 원인이 흔들릴 때는
  `superpowers:systematic-debugging`을 먼저 사용한다
- 이미 작성된 계획을 따라 구현할 때는
  `superpowers:executing-plans`를 사용한다
- 완료 주장 전에 `superpowers:verification-before-completion`으로 검증한다

### 4. Review

주요 담당: `gstack`

이 단계에서 수행할 일:

- 코드 리뷰
- 회귀 위험 점검
- 버그 및 리팩터 제안 검토

`Superpowers`의 역할:

- 중요한 작업을 마무리하기 전에
  `superpowers:requesting-code-review`를 사용한다
- 리뷰 피드백을 반영할 때는
  `superpowers:receiving-code-review`를 사용한다
- 단순 문법 수준이 아니라 유지보수성, 구조적 리팩터 여지,
  규칙 준수 여부까지 깊게 점검한다

`GitHub`의 역할:

- PR 리뷰, 코멘트, requested changes, CI 상태 확인에는
  `GitHub` plugin 도구를 우선 사용한다

### 5. Test

주요 담당: `gstack`

이 단계에서 수행할 일:

- QA와 시나리오 검증
- 브라우저 자동화 및 E2E 확인
- 엣지 케이스 점검

`Superpowers`의 역할:

- 테스트 설계와 커버리지 기준을 보강한다
- 빠진 엣지 케이스, 약한 assertion, 불충분한 회귀 테스트를 확인한다
- 최종 증거 게이트로 `superpowers:verification-before-completion`을 사용한다

### 6. Ship

주요 담당: `gstack`

이 단계에서 수행할 일:

- PR 준비
- 배포 및 릴리스 절차 수행
- 릴리스 노트 및 핸드오프 정리

`Superpowers`의 역할:

- 배포 직전 규칙 준수와 준비 상태를 다시 확인한다
- 구현이 끝난 뒤 가장 안전한 다음 통합 단계를 정할 때
  `superpowers:finishing-a-development-branch`를 사용한다

`GitHub`의 역할:

- 커밋, 푸시, PR 생성 흐름이 필요하면 `github:yeet`를 사용한다
- GitHub Actions 실패 때문에 출고가 막히면 `github:gh-fix-ci`를 사용한다

### 7. Reflect

주요 담당: `gstack`

이 단계에서 수행할 일:

- 회고
- 배포 이슈와 전달 과정 분석
- 다음 스프린트를 위한 개선점 정리

`Superpowers`의 역할:

- 반복되는 실패를 코드 규칙, 리뷰 체크포인트, 테스트 기대치로 환원한다
- 회고 결과를 다음 계획과 구현 규율에 반영한다

## 도구 선택 규칙

### `Context7`를 사용할 때

- 라이브러리나 프레임워크 정보가 최신 문서에 의존할 수 있을 때
- 현재 버전 기준 사용 예제가 필요할 때
- 구현이 버전별 차이에 민감할 때

### `OpenAI Developer Docs`를 사용할 때

- OpenAI API, 모델, SDK, 도구, 제품 동작이 관련된 작업일 때
- 공식 문서 근거가 필요할 때

### `GitHub` plugin을 사용할 때

- PR, 이슈, 리뷰 스레드, CI 실패를 기준으로 작업할 때
- 커밋, 푸시, PR 생성이 필요할 때
- 리뷰 코멘트나 워크플로 로그를 확인해야 할 때

### `Superpowers`를 기본 규율 계층으로 사용할 때

- 동작 추가나 변경 전 설계 정리가 필요하면 `brainstorming`
- 설계 승인 후 실행 계획이 필요하면 `writing-plans`
- 구현 전 테스트 주도 흐름을 강제하려면 `test-driven-development`
- 추측성 수정 대신 원인 기반 디버깅이 필요하면 `systematic-debugging`
- 완료 주장 전 실제 검증이 필요하면 `verification-before-completion`

## 실행 규칙

- 아이디어에서 바로 코드로 점프하지 않는다
- 여러 단계가 있는 작업은 명시적인 계획을 선호한다
- 기능 개발과 버그 수정에는 가능하면 TDD를 적용한다
- 검증 근거 없이 완료를 선언하지 않는다
- `gstack`은 흐름 설계와 의사결정 프레이밍을 담당한다
- `Superpowers`는 품질 게이트와 실행 규율을 담당한다
- 둘 다 필요한 경우 `gstack`으로 방향을 잡고 `Superpowers`로 규율을 강제한다

## 짧은 운영 휴리스틱

헷갈릴 때는 아래 순서를 기본값으로 사용한다.

1. `gstack`으로 작업의 흐름과 결정을 정리한다.
2. `Superpowers`로 계획, TDD, 디버깅, 리뷰, 검증 규율을 적용한다.
3. 최신 라이브러리 문서는 `Context7`로 확인한다.
4. PR, 이슈, CI, 릴리스 작업은 `GitHub`로 처리한다.
5. `Reflect`에서 나온 문제는 다음 규칙 강화로 이어지게 만든다.

---
> Source: [Kminer2053/gongmuwon](https://github.com/Kminer2053/gongmuwon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
