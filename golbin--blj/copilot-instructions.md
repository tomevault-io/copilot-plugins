## blj

> BLJ는 BOJ 번호 체계를 유지하면서, 문제 본문 아카이브와 로컬 채점으로 문제를 풀 수 있게 만드는 로컬 우선 웹앱이다.

# 저장소 작업 지침

## 개요

BLJ는 BOJ 번호 체계를 유지하면서, 문제 본문 아카이브와 로컬 채점으로 문제를 풀 수 있게 만드는 로컬 우선 웹앱이다.

가장 중요한 불변 조건:

- 사용자에게 보여주는 결과는 항상 로컬 기준이어야 하며, 공식 BOJ 판정이나 원문 서비스 가용성을 암묵적으로 전제하면 안 된다.
- BOJ 수집 기능은 운영 서버에 과도한 부하를 주지 않는 점진적 수집을 전제로 유지해야 한다.

패키지 매니저와 스크립트 실행은 `pnpm`만 사용한다. 다른 lockfile, 패키지 매니저, 중복 실행 체계를 추가하지 않는다.

핵심 스택:

- Vite + React 19 + TypeScript
- CodeMirror 기반 에디터
- 브라우저 워커 기반 Python / JavaScript / TypeScript 실행
- 로컬 Docker 사이드카 기반 C / C++ / Java 실행
- SQLite 카탈로그 + IndexedDB(Dexie) 기반 브라우저 로컬 작업공간
- Vitest / ESLint / Playwright 기반 검증 스크립트

## 구조

```text
src/app/               화면, 라우팅, 에디터, 상태 연결
src/judge/             로컬 채점 로직, 언어 런너, 어댑터, 사이드카 클라이언트
src/storage/           브라우저 로컬 저장소와 카탈로그 읽기 계층
src/lib/               문제팩, sanitize, 공통 유틸
src/types/             도메인 타입
scripts/               사이드카, 크롤링, 카탈로그 동기화, E2E/운영 스크립트
data/boj-crawl/        점진 수집 데이터, 상태, 로그, 정규화 결과
data/release/          릴리스 기준 SQLite 카탈로그 체크포인트
public/data/           실행 시 퍼블리시되는 카탈로그 자산
docker/sidecar/        C/C++/Java 채점용 컨테이너 정의
docs/                  제품 계약, 디자인, 운영 문서
```

## 핵심 문서

- `README.md`: 실행 방법, 주요 명령, 운영 메모
- `docs/product/spec.md`: 제품 요구사항과 사용자 경험 기준
- `docs/design/system.md`: 디자인 기준과 UI 원칙
- `docs/operations/runbook.md`: 설치, 운영, 검증, 복구 절차

이 파일은 짧게 유지한다. 세부 정책과 운영 절차는 위 문서들에 둔다.

## 명령

```bash
pnpm install
pnpm dev
pnpm dev:full
pnpm preview:full
pnpm sidecar
pnpm test:run
pnpm lint
pnpm build
pnpm smoke:e2e
pnpm judge:e2e
pnpm judge:e2e:preview
pnpm verify:release
pnpm crawl:boj help
pnpm crawl:boj status
pnpm crawl:sync-batch --fetch-limit 10 --verify-limit 10 --coverage-limit 20
pnpm sync:crawl-pack
pnpm judge:coverage
pnpm judge:scaffold --problem-id 1010 --dry-run
```

광범위하거나 비용이 큰 명령은 필요할 때만 실행한다. 변경 범위를 증명할 수 있는 가장 작은 검증부터 우선한다.

## 비협상 규칙

- 로컬 판정, 로컬 아카이브, 로컬 실행이라는 제품 약속을 깨지 않는다.
- 공식 BOJ 결과를 암시하는 문구, UI, 저장 필드를 추가하지 않는다.
- 크롤러는 반드시 정책 승인과 속도 제한을 존중해야 하며, 서버 부하를 키우는 공격적 기본값을 넣지 않는다.
- 입력 데이터, 크롤링 결과, 문제팩, 외부 응답은 경계에서 검증하고 정리한다.
- 비밀값, 토큰, 쿠키, 개인 정보, 원문 서비스 민감 정보를 로그에 남기지 않는다.
- 파일 시스템, 네트워크, Docker, 브라우저 워커, SQLite/IndexedDB 접근은 경계 모듈에 모은다.
- 관련 없는 dirty worktree 변경은 건드리지 않는다.

## 계획

다음 조건 중 하나라도 해당하면 구현 전에 짧은 1페이지 계획을 남긴다.

- 작업 범위가 넓다.
- 요구사항이 애매하다.
- 채점 경로, 저장 포맷, 크롤링 정책, 배포 동작을 바꾼다.
- 여러 언어 런타임이나 외부 통합에 영향을 준다.

포함할 항목:

- 배경
- 문제
- 목표
- 비목표
- 제약
- 구현 개요
- 검증 계획
- 되돌리기 또는 복구 메모

작고 명확한 수정은 주변 코드를 읽은 뒤 바로 진행해도 된다.

## 코드 품질

- 수정 전 관련 코드와 테스트를 먼저 읽는다.
- 숨은 마법보다 의도가 드러나는 코드를 선호한다.
- 플랫폼 의존 로직은 얇은 경계 모듈 뒤로 숨기고, 앱 코드는 명확한 API만 호출하게 유지한다.
- 실제 중복이 생기기 전에는 과도한 추상화를 만들지 않는다.
- 함수나 파일이 길어져 동작 파악이 어려워지면 분리한다.
- 경로, 인코딩, 시간, 프로세스 실행, 브라우저/컨테이너 차이를 항상 의식한다.
- 구조화된 API를 우선하고, ad hoc 문자열 결합으로 JSON, 셸 인수, 경로를 만들지 않는다.

권장 기준:

- 파일 300 LOC 안팎
- 함수 50 LOC 안팎
- 매개변수 5개 이하
- 복잡도 10 이하

기준을 넘겨야 한다면 이유를 계획 문서나 작업 설명에 남긴다.

## 검증

- 위험도에 맞춰 검증 수위를 맞춘다. 정적 검사, 단위 테스트, 통합 테스트, 브라우저 E2E, 수동 스모크 중 필요한 최소 조합을 선택한다.
- 외부 시스템을 직접 때리는 테스트보다 결정적인 fixture, 샘플, 로컬 팩, 계약형 테스트를 우선한다.
- 채점 변경은 가능하면 `pnpm test:run`, `pnpm judge:e2e`, `pnpm smoke:e2e` 중 적절한 조합으로 확인한다.
- 크롤링 변경은 상태 파일, 실패 로그, 증분 재개 흐름까지 같이 확인한다.
- 최종 보고에는 실행한 것, 실행하지 않은 것, 남은 위험을 분명히 적는다.
- 환경 변수 기본값과 운영 절차는 `docs/operations/runbook.md`와 `.env.example`을 기준으로 맞춘다.

## Git / PR

- 커밋은 작고 되돌리기 쉽게 유지한다.
- 현재 작업과 관련된 파일만 stage 한다.
- 명시적 요청 없이 히스토리 재작성, 태그 이동, force-push를 하지 않는다.
- PR에는 사용자 영향, 데이터 포맷 변화, 운영 영향, 검증 내용을 포함한다.

---
> Source: [golbin/blj](https://github.com/golbin/blj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
