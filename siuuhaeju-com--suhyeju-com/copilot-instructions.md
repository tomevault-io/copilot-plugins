## suhyeju-com

> > Agent 페르소나 **Single Source**. Cursor·Claude·Codex 모두 이 파일(`AGENTS.md`)을 따른다.

> Agent 페르소나 **Single Source**. Cursor·Claude·Codex 모두 이 파일(`AGENTS.md`)을 따른다.

## 프로젝트 개요

**수혜주.com** (테오 AI 스프린트 6팀) — 사용자가 **뉴스 링크를 붙여넣으면**, AI가 그 이슈가 어느 산업으로 번지는지 **1→2→3차 파급 경로를 노드 그래프(섹터별 영향도 포함)로 시각화**해 주는 **뉴스 기반 산업 파급 분석 서비스**다. 예: _"AI 투자 확대"_ 뉴스 → `GPU → HBM → 반도체 장비 → 유리기판 → 특수가스 → 데이터센터 → 전력설비` 같은 연쇄 파급 사슬과 **다음 수혜 산업**을 짚어 준다.

- **핵심 가치:** 정보 과부하 해소 · 뉴스 근거 기반 신뢰성 · 시각적 가독성 — 초보도 "이 뉴스가 다음엔 어디로 번질까"를 스스로 그려 보게 한다.
- **화면 3개:**
  - `메인 페이지` — 뉴스 링크 입력 + 인기 뉴스·섹터 현황·최근 분석 내역
  - `로딩 페이지` — 단순 스피너 금지, "뉴스 읽기 → 이슈 추출 → 호재/악재 비교 → 그래프 → 영향도·근거 정리" 단계 표시
  - `분석 페이지` — 요약 카드, 1·2·3차 파급 그래프(섹터별 영향도·부정 파급 포함, F-16 #86에서 히트맵 통합), 호재/우려 여론 비교, 산업 지식그래프
- **⚠️ 색상 규칙(코드 작업 시 주의):** **긍정/상승 = 레드(핑크 계열), 부정/하락 = 블루** 로 통일한다(국내 증시 관례). 서구식 초록/빨강과 **다르므로** 범례를 반드시 병기한다. (섹터 등락·신호 카드·그래프 영향도 공통)

## 디자인 작업

UI 컴포넌트·페이지·스타일을 만들거나 수정하는 **모든 디자인 관련 작업**은 아래를 따른다.

- **작업 전:** **`apps/web/DESIGN.md`** (디자인 시스템 정본)와 `apps/web/PRODUCT.md`(전략 컨텍스트)를 먼저 읽는다. DESIGN.md의 **frontmatter 토큰이 규범**이고, 본문은 적용 맥락이다. 시각 원본이 더 필요하면 `docs/design/mockup/`을 참고한다.
- **작업 중:** 색·radius·타이포는 토큰/shadcn CSS 변수로만 쓴다(hex 하드코딩 금지). 특히 의미색(긍정=레드/부정=블루)·파급 단계색·`ink-dim` 텍스트 금지 등 **Colors·Do's and Don'ts 섹션을 위반하지 않는다.**
- **작업 후:** DESIGN.md의 **Do's and Don'ts와 대조**하고, `npx impeccable detect <변경한 파일·디렉터리>`를 실행해 안티패턴·대비 위반이 없는지 확인한다. 발견 항목은 머지 전에 해소한다.
- **문서 동기화:** 작업 중 새 토큰·컴포넌트 규칙이 확정되면 DESIGN.md에 반영한다. 코드와 문서가 어긋난 채로 두지 않는다.

> 디자인 품질 작업(크래프트·크리틱·폴리시 등)에는 `/impeccable` 스킬을 활용한다 — 자동으로 PRODUCT.md·DESIGN.md를 읽고 시작한다.

## 데이터/API 작업

뉴스 분석(analyze)·시세·데이터 가공 관련 작업은 **`docs/data-pipeline.md`**(가공 파이프라인 단일 기준)를 먼저 읽고 따른다. 무엇을 GPT가 만들고·서버가 채우고·FE가 그리는지, `AnalysisResult` 필드 소유권, GPT 스키마 무결성 규칙, 시세 join 방법이 정리돼 있다.

## 프론트엔드 코드 컨벤션 (apps/web)

Next.js 코드 작성 시 아래 네이밍 규칙을 따른다.

| 대상                | 규칙                                                                                                                            | 예                                        |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| 컴포넌트 파일(.tsx) | **PascalCase** — 예외: Next 예약 파일(`page.tsx`·`layout.tsx`·`error.tsx`)과 shadcn CLI 생성물(`components/ui/*`)은 소문자 유지 | `SpreadGraph.tsx`, `SiteHeader.tsx`       |
| 일반 모듈(.ts)·폴더 | **kebab-case**                                                                                                                  | `mock-data.ts`, `components/analysis/`    |
| 컴포넌트            | **PascalCase**, named export (파일명과 일치)                                                                                    | `SpreadGraph`, `SiteHeader`               |
| 훅                  | `use` + PascalCase                                                                                                              | `useAnalysis`, `useSectorOverview`        |
| 함수                | camelCase 동사 시작                                                                                                             | `buildCloud()`, `getAnalysis()`           |
| 이벤트 핸들러       | 내부 정의 `handle*` / props 전달 `on*`                                                                                          | `handleSubmit`, `onSelect`                |
| boolean             | `is/has/can/should` 접두                                                                                                        | `isLoading`, `hasError`                   |
| 상수                | **UPPER_SNAKE_CASE** (모듈 상단)                                                                                                | `STEP_INTERVAL_MS`, `TIER_COLOR`          |
| 타입·인터페이스     | PascalCase, 접두사 `I`/`T` 금지                                                                                                 | `AnalysisResult`, `SpreadNode`            |
| 쿼리 키             | 배열 리터럴 `[도메인, 식별자]`                                                                                                  | `['analysis', id]`, `['news', 'popular']` |
| 환경변수            | `NEXT_PUBLIC_` (클라이언트 노출 시) + UPPER_SNAKE                                                                               | `NEXT_PUBLIC_API_URL`                     |

- import 경로는 상대경로 대신 **`@/` alias**를 쓴다 (`@/components/ui/button`).
- 클래스는 지양하고 함수형으로 작성한다 — 도메인 로직은 `src/lib/`에 순수 함수로.

## 이슈 생성

"이런 이슈 만들어줘" 같은 요청을 받으면:

1. 제목·라벨·본문 규칙은 **`scripts/new-issue.sh`와 `.github/ISSUE_TEMPLATE/`가 강제**한다(제목 `[ID] 명사형`, 유형별 템플릿 자동). 팀원 안내는 `docs/issue-open-guide.md`.
2. 요청에서 **유형·ID·제목·순위·페이지·참고·완료조건·체크리스트**를 뽑아 **`scripts/new-issue.sh`** 로 등록한다. **직접 `gh issue create`를 손으로 구성하지 않는다**(형식이 흔들린다). 카탈로그(`docs/proposal/14`·`docs/issues-list.md`)에 있으면 그 내용을 재사용한다.
3. 먼저 `--dry-run`으로 조립 결과(제목·라벨·본문)를 사용자에게 보여주고, 확인되면 실제 등록한다.
4. 등록 후 **생성된 번호·제목·라벨**을 보고한다.
5. 라벨이 없으면 스크립트가 자동 생성한다(최초 1회는 `scripts/init-labels.sh`).

> gh CLI 설치(`brew install gh`)·인증(`gh auth login`)이 선행되어야 실제 등록된다.

## 브랜치 전략

**Git Flow 기반**으로 브랜치를 운영한다.

| 브랜치                  | 역할                                              |
| ----------------------- | ------------------------------------------------- |
| `prd`                   | **운영** 브랜치                                   |
| `dev`                   | 운영 전 **병합 후 확인**용 (별도 dev 서버는 없음) |
| `feat/#이슈번호-기능명` | **작업(기능) 브랜치** — 이슈 단위로 생성          |

**흐름:** `feat/#이슈번호-기능명` → `dev` → `prd`

- 작업은 반드시 **`feat/#이슈번호-기능명`** 브랜치에서 한다. (예: `feat/#12-login-form`)
- 기능 완료 시 **`dev`** 에 병합해 통합 상태를 확인한 뒤, **`prd`** 로 올려 운영에 반영한다.
- `prd`·`dev`에 **직접 커밋하지 않는다**. 브랜치명의 `#이슈번호`는 관련 GitHub Issue 번호와 일치시킨다.

---
> Source: [siuuhaeju-com/suhyeju-com](https://github.com/siuuhaeju-com/suhyeju-com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
