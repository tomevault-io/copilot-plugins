## repo-starter

> 이 저장소에서 코드를 작성하는 모든 에이전트(Codex, Claude Code 등)가 따르는 규칙이다.

# AGENTS.md

이 저장소에서 코드를 작성하는 모든 에이전트(Codex, Claude Code 등)가 따르는 규칙이다.

상세 규칙은 `.claude/docs/` 에 있다. **파일을 수정하기 전에 해당 문서를 읽는다.**
(`.claude/` 라는 경로명이지만 Claude 전용이 아니다 — 도구 중립적인 아키텍처 규칙 문서다.)

| 상황 | 읽을 문서 |
|---|---|
| 작업 순서·커밋·PR | `.claude/docs/01-WORKFLOW.md` |
| 도메인 모델링, 항상 | `.claude/docs/10-DDD-CORE.md` |
| NestJS 백엔드 | `.claude/docs/20-BACKEND-NESTJS.md` |
| FastAPI/Python 백엔드 | `.claude/docs/21-BACKEND-PYTHON.md` |
| Next.js 프론트 | `.claude/docs/30-FRONTEND-FSD.md` |
| 테스트 작성 | `.claude/docs/40-TDD.md` |
| **구현 계획 수립** | `.claude/docs/50-PLAN-CHECKLIST.md` |

---

## 스택

| 영역 | 스택 | 아키텍처 |
|---|---|---|
| 프론트엔드 | Next.js (App Router) + TypeScript | **FSD** (Feature-Sliced Design) |
| 백엔드 (Node) | NestJS + TypeScript | **DDD + Ports & Adapters + Facade** |
| 백엔드 (Python) | FastAPI + SQLAlchemy | **DDD + Ports & Adapters + Facade** |
| 개발 방식 | 전 영역 | **TDD** (Red → Green → Refactor) |

---

## 절대 규칙 (위반 시 머지 불가)

### 백엔드

1. **도메인 레이어는 프레임워크를 모른다.** `@nestjs/*`, `typeorm`, `sqlalchemy`, `pydantic`,
   `fastapi` 를 `domain/` 에서 import 하지 않는다.
2. **컨트롤러/라우터는 Facade 만 호출한다.** UseCase·리포지토리·`AsyncSession` 을 직접 주입받지 않는다.
3. **트랜잭션은 Facade 메서드에서 연다.** UseCase 안에서 열지 않는다. 중첩 금지.
4. **한 트랜잭션에 애그리거트 하나.** 넘으면 도메인 이벤트로 분리한다.
5. **ORM 모델 ≠ 도메인 엔티티.** 항상 별개 클래스 + 매퍼를 둔다.
6. **애그리거트는 다른 애그리거트를 ID 로만 참조**한다. 객체 참조 금지.
7. **시간·랜덤·ID 는 주입받는다.** 도메인 안에서 `new Date()` / `datetime.now()` / `uuid()` 호출 금지.
8. **모듈 `exports` 에는 Facade 만.** 컨텍스트 내부를 밖으로 내보내지 않는다.
9. **다른 컨텍스트는 Facade 또는 이벤트로만** 접근한다. 테이블·리포지토리 직접 접근 금지.

### 프론트엔드

10. **의존은 한 방향뿐이다**: `app → views → widgets → features → entities → shared`.
    역방향, 같은 레이어 슬라이스 간 import 금지.
11. **슬라이스는 `index.ts` 공개 API 로만** 들어간다. 깊은 import 금지.
12. **`shared/` 에 도메인 단어가 있으면 위반**이다.
13. **서버 DTO 는 `entities/*/api/mapper.ts` 를 거쳐** 도메인 모델로 변환한 뒤 사용한다.
14. **쿼리 키는 `entities/*/api/queries.ts` 에 집중**한다. 인라인 정의 금지.
15. `app/` 의 `page.tsx` / `layout.tsx` 는 **10줄 이하**, 뷰 위임만.

### 공통

16. **실패하는 테스트 없이 프로덕션 코드를 쓰지 않는다.**
17. **`skip` / `only` 가 남은 채로 커밋하지 않는다.**
18. 이름은 `docs/domain/<context>/ubiquitous-language.md` 의 식별자를 쓴다.
    `Manager` / `Processor` / `Util` / `Helper` / `data` / `info` 금지.
19. 로그에 **traceId / userId** 등 요청 컨텍스트를 싣는다. 도메인 레이어는 로깅하지 않는다.

---

## 작업 절차

Claude Code 에는 이 절차를 자동화한 스킬(`/ddd-plan` 등)과 서브에이전트가 있다
(`.claude/skills/`, `.claude/agents/`). **그 기능이 없는 도구에서는 아래를 수동으로 수행한다.**

### 1. 계획 (코드 작성 전)

`.claude/docs/50-PLAN-CHECKLIST.md` 의 A~F 항목에 **답을 문장으로 적는다.**
최소한 다음이 계획서에 있어야 한다.

- 어느 바운디드 컨텍스트인가
- 애그리거트와 **불변식**, 상태 전이표
- 한 트랜잭션에 애그리거트 하나인가? 아니면 어떤 이벤트로 나누나
- 레이어별 변경 파일 목록
- Facade 메서드 시그니처
- 새 포트가 있다면 **DI 배선 지점 파일명**
- 프론트라면 어느 레이어의 일인지 + 판단 근거
- **테스트 목록 (구현 목록보다 위에)**

같은 문서의 "계획을 반려하는 신호" 10개에 해당하면 계획을 고친 뒤 시작한다.

### 2. 구현 (TDD)

```
RED    실패 테스트를 먼저 쓰고, 실제로 실패하는 것을 확인한다
GREEN  domain → application(UseCase → Facade) → infrastructure → presentation → 프론트
       단계마다 테스트 실행
REFACTOR  초록을 유지하며 구조 정리
```

- 리포지토리는 **목이 아니라 인메모리 Fake** 로 대체한다 (`.claude/docs/40-TDD.md` §2).
- 목은 외부 경계(결제·메일)에서 "보냈는지" 확인할 때만 쓴다.
- **DI 배선을 잊지 않는다** — NestJS `providers`/`exports`, Python `container.py`. 가장 자주 빠진다.

### 3. 커밋 전 검사 (직접 실행하고 출력을 확인한다)

```bash
# NestJS
npm test && npx depcruise src --config .dependency-cruiser.js

# Python
pytest -m "not e2e" && lint-imports && mypy app

# Next.js
npm run test && npx steiger ./src && npm run lint
```

린터 설정이 아직 없으면 그 사실을 보고한다. **눈으로 본 것은 검증이 아니다.**

그다음 위 "절대 규칙" 19개를 변경분에 대해 직접 점검한다.
위반이 남아 있으면 커밋하지 않는다.

### 4. 문서 동기화

- API 변경 → `docs/api-spec/<context>.md` + `docs/api-spec/changelog/<context>/log_<날짜>.md`
  (breaking change 는 ⚠️ + 영향 범위 + 마이그레이션 방법)
- 엔드포인트 추가/변경 → `http/<context>.http` (성공 + 400/401/403/404/409 케이스)
- 새 용어 → `docs/domain/<context>/ubiquitous-language.md`

---

## 커밋 / PR

### 커밋 메시지

**Conventional Commits** 접두사로 시작한다. 접두사는 영어 소문자, 설명은 한국어.

```
<type>(<scope>): <무엇을 했는지 한 줄>

상황: 바꾸기 전 상태 (사실만)
원인: 그게 왜 문제인지 — 재현 조건, 깨진 가정
조치: 무엇을 어떻게 바꿨는지 + 버린 대안이 있으면 그 이유
결과: 검증한 사실 (테스트 통과 수, 린터 출력)
```

- 라벨(`상황:` 등)은 생각의 순서용이고 실제로는 쓰지 않는다 — 문단으로 푼다.
- type: `feat` `fix` `refactor` `docs` `style` `test` `chore` `perf`
- 제목은 72자 이내, 마침표 없음.
- `style` 을 제외한 모든 커밋은 본문에 **원인**을 남긴다. diff 를 보면 "무엇"은 알 수 있다.
  메시지가 남겨야 하는 건 **diff 에 안 보이는 것**이다.
- 가장 중요한 한 문장은 `**볼드**`. 근거는 요약하지 말고 **원문 인용**.
- 추측·사과 금지. 검증 안 한 건 "미검증"이라고 쓴다.
- **커밋과 PR 은 사용자가 요청할 때만 생성한다.**

### PR

제목도 동일하게 접두사로 시작한다 (squash merge 시 `(#21)` 자동 부착).
본문 순서:

```
## 요약
## 배경 / 원인          ← 반드시 채운다
## 변경사항
## 검증                 ← 미검증 항목은 "미검증"이라고 명시한다
## 영향 범위 · 롤백
## 리뷰 포인트
```

**본문은 한 화면(약 40줄)을 넘기지 않는다.** 섹션 구조는 유지하되 각 섹션은 1~4줄이다.

- **변경 파일 목록·디렉터리 트리·표를 넣지 않는다.** diff 가 이미 보여 준다.
- 코드 블록을 붙이지 않는다. 필요하면 `파일:줄` 로 가리킨다.
- 불릿은 섹션당 3개 이하. 나머지는 문장으로.
- 길어야만 하는 근거(벤치마크, 긴 로그)는 `<details>` 로 접는다.

남길 것은 **diff 에 안 보이는 것**뿐이다 — 왜 이렇게 했는지, 무엇을 검증했는지, 무엇이 미검증인지.
리뷰어는 "무엇이 바뀌었나"를 코드에서 읽는다. 본문에 그걸 복사하면 읽히지 않는 PR 이 된다.

`.github/pull_request_template.md` 가 있으면 그 구조를 우선한다.
squash merge 는 커밋 본문을 그대로 머지 커밋에 넣는다 → **커밋 본문 품질이 곧 영구 기록**이다.

---

## 하지 않는 것

- 계획 없이 프로덕션 코드 쓰기
- 테스트를 느슨하게 고쳐서 통과시키기 (테스트가 깨졌다면 동작이 바뀐 것이다)
- 요청받지 않은 리팩터링·성능 최적화 끼워 넣기 — 발견하면 **보고하고 별도 작업으로 남긴다**
- 검증하지 않은 것을 검증한 것처럼 보고하기
- 사용자 요청 없이 커밋·푸시·PR 생성

---
> Source: [sung-young-fnf/repo_starter](https://github.com/sung-young-fnf/repo_starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
