## plannode-core

> Plannode 코어 — 상용 웹앱 개발계획 협업·표준 구조·트리 보호·하네스(협업≠경량화 축소)


# Plannode 코어

**제품 포지션:** **상용 웹앱 개발계획 협업 서비스** — 정본은 [`plannode-prd.mdc`](./plannode-prd.mdc) **§1.05~§1.06**. “1인 내부 플래닝 도구” 가정은 **문서·에이전트에서 사용하지 않는다**.

**역할:** Cursor에서 **항상 적용(alwaysApply)** 되는 저장소 진입 규칙이다. **`.mdc` 빠른 인덱스·Markdown 편집 규약**은 [`.cursor/rules/README.md`](./README.md)가 정본이고, **본 파일**은 **핵심 표준 구조·트리 보호·하네스·유지보수 원칙**을 고정한다.

- **스택(현재 구현 기준):** **SvelteKit + TypeScript** 앱 셸(`src/routes/*`)과 **내장 파일럿** `src/lib/pilot/plannodePilot.js`가 하이브리드로 동작한다. 루트 **Vanilla** `index.html` + `plannode.js`는 **동작·포팅 정합의 참고 기준**이며, 세부 갭은 `docs/PILOT_FUNCTIONAL_SPEC.md` §9~§10.
- **구현 아키텍처·모듈·데이터 흐름(유지보수 기준)**: `.cursor/rules/plannode-architecture.mdc` — **「원격선택-null」/`PRESENCE_PEER_MERGE`(§5.1)** — **규칙 폴더 인덱스**: `.cursor/rules/README.md`
- **UI 색·타이포·반응형·레이어 표준**: `.cursor/rules/plannode-ui-identity.mdc`
- **제품·기능·로드맵(IA/와이어, LLM §10, v2 DB 등)**: `.cursor/rules/plannode-prd.mdc` — 하네스 `plan-output.md`·`TASK.md`는 **PRD 항목(M#·F#-#)을 1줄씩** 추적한다.
- **파일럿 정합·포팅 갭**: `docs/PILOT_FUNCTIONAL_SPEC.md` §9~§10.
- 인프라·배포: `.cursor/plans/PLANNODE_INTEGRATED_GUIDE.md`를 우선한다.
- **릴리스 노트 UI:** 캔버스 하단 「Release」→ 모달 「Release note」(`+page.svelte`, 정적 `src/lib/plannodeUpdateLog.ts`) — 표준 톤은 `plannode-ui-identity.mdc` §5 캔버스 하단 릴리스 행.
- 개발 워크플로우: **Harness Flow v1.0** — `AGENTS.md` + `.cursor/harness/` + `.cursor/agents/` 참조.
- `plannode-prd`·`PILOT_FUNCTIONAL_SPEC`·코드·통합 가이드가 어긋나면 **불일치를 먼저 밝히고** 한쪽을 명시적으로 맞춘다(묵시적 불일치 유지 금지).

## 핵심 표준 구조 (코드 기준 — 위반 시 회귀)

아래는 `plannode-prd.mdc` §1.0·`plannode-architecture.mdc`와 같은 말을 **에이전트가 바로 따를 수 있는 규칙**으로 압축한 것이다.

| 규칙 | 내용 |
|------|------|
| **단일 진실(트리 편집 중)** | 노드 캔버스·레이아웃·간선의 진실은 **파일럿 런타임**이다. Svelte 스토어·localStorage·클라우드는 **영속·동기 층**이며, 편집을 우회하는 **둘째 노드 저장소**를 만들지 않는다. |
| **브리지 계약** | `pilotBridge.ts`: 파일럿 저장 시 `onPersist` → `persistNodesFromPilot`; 프로젝트 전환 시 `hydrateFromStore`. 노드 필드 매핑·깊이 계산 변경 시 스토어·파일럿 **양쪽**을 함께 본다. |
| **뷰 동기** | Svelte `activeView`와 파일럿 `pilotSetActiveView`는 **같은 값 집합**(`tree` \| `prd` \| `spec` \| `ia` \| `ai`)으로 쌍을 맞춘다. 툴바·숨은 버튼 위임은 `plannode-architecture.mdc` §6. |
| **타입 단일성** | `Project`·`Node`는 **`$lib/supabase/client`**에서 import해 스토어·브리지·UI가 공유한다. |
| **로컬 1차 영속** | 키 접두 `plannode_*_v3` — 상세는 아키텍처 §4. 루트 노드 규칙(`makeRootNode` 등)을 깨지 않는다. |
| **클라우드(설정 시)** | 노드 본문은 **`plannode_workspace` JSON 번들** upsert/풀·머지 경로가 정본이다. **배지 협업:** B축 `update_node` badges/metadata + A축 slice fallback — **§10.10.1** · **badge-mapping §6.9**. **Presence(Realtime)**는 선택 노드 등 **메타**만 — 번들 Realtime 스트리밍 없음. 모달 pull 보류·RPC는 **`plannode-architecture.mdc` §10**. |
| **클라우드 저장 트리거** | 파일럿 쪽 출력·편집 후 셸에서 클라우드를 태울 때는 `plannode-auto-cloud-sync` 커스텀 이벤트 계약을 따른다(`+page.svelte` 리스너와 쌍). |
| **트리 교환** | 백업·가져오기는 `plannodeTreeV1.ts` 스키마를 따른다. |
| **AI 클라이언트** | `src/lib/ai/*`는 §10 방향의 **부분 구현**이다. 서버 영속·§11 DB와 갭이 있으면 PRD·아키텍처에 **명시**하고 일반론으로 메우지 않는다. |

**트리뷰 핵심 보호:** 캔버스·`#V-TREE`·transform·간선·미니맵·파일럿 DOM 계약 변경은 **최소 침습**. 부가 뷰 CSS가 트리를 가리거나 `.view` 전환을 깨지 않게 한다 — 상세는 **`AGENTS.md`** 헌장·**GP-13**.

## 하네스 플로우 핵심 참조

| 파일 | 역할 |
|------|------|
| `.cursor/rules/README.md` | **`.mdc` 통합 인덱스** · Markdown·README 편집 규약 |
| `.cursor/rules/plannode-architecture.mdc` | SvelteKit·파일럿 브리지·스토어·Supabase 데이터 흐름 |
| `.cursor/rules/plannode-ui-identity.mdc` | 브랜드 색·타이포·브레이크포인트·모달/배지 패턴 |
| `.cursor/rules/plannode-prd.mdc` | PRD: M/F/Phase, IA·와이어 vs F2-5 LLM, §10~§11, **§1.0 구현 요약** |
| `.cursor/rules/plannode-badge-mapping.mdc` | 표준 배지 풀 매핑·풀 확대 동기화·가져오기→칩 파이프 |
| `AGENTS.md` | 프로젝트 정체성·황금 원칙·에이전트 호출 순서·문서 위계 |
| `.cursor/harness/README.md` | 폴더 역할·DB 절·운영 원칙·PRD 연계 |
| `.cursor/harness/TASK.md` | 현재 스프린트 태스크 스택 |
| `.cursor/harness/plan-output.md` | @promptor 출력 (GATE A 확정본) |
| `.cursor/harness/context-hook.md` | 컨텍스트 드리프트 방지 훅 |
| `.cursor/agents/harness-executor.md` | Step4 GSD 실행 에이전트 |
| `.cursor/agents/promptor.md` | Step2 아젠다 분석·플래너 |
| `.cursor/agents/qa.md` | Step5 QA 검수 에이전트 |
| `.cursor/plans/harness-workflow_final.md` | 하네스 **기본·단축** · GATE · 채팅 복붙 블록 |

## 파일럿 갭 원칙

- SvelteKit 포팅 작업 시 `docs/PILOT_FUNCTIONAL_SPEC.md §9~§10` 갭 분석을 기준으로 삼는다.
- 파일럿(`index.html` + `plannode.js`) 동작이 SvelteKit 구현의 **정합성 기준**이다.
- transform 컨테이너·parent_id 체계·루트 노드 생성 등 갭 항목 변경 시 반드시 GATE C에서 체크한다.

## 최소 구현·기술부채·경량화 (하네스 공통)

- **오버 엔지니어링 견제:** 아젠다·PRD·TASK에 없는 추상·불필요한 **신규 모듈/파일**·“나중에 쓸” 데드 코드를 넣지 않는다(기존 경로 확장 우선).
- **협업·동기화는 경량화 대상이 아님:** M5·`plannode-architecture.mdc` §10(번들·슬라이스·revision·ops·Presence·모달 pull 보류)은 **상용 협업 정상 경로**다. PRD §1.06 — **pull/slice/ACL을 “1인용이라 생략”**하는 설계·패치 **금지**.
- **경량화 제어:** `AGENTS.md` **「경량화·오버엔지니어링 견제 제어 구조」** — **스코프 밖 확장** 억제이지, **협업 로직 미니멀화** 허가가 **아님**. `.cursor/harness/README.md` **「최소 구현·기술부채·경량화」**.
- **기술부채 누적 지양:** `AGENTS.md` GP-12, `@qa` 검수 2단계(기술 부채), `@harness-executor` G-STEP 4를 함께 본다.
- **Guides(스펙·GATE) vs Sensors(빌드·QA):** 불필요한 설계(오버엔지니어링)는 **센서만**으로는 부족할 수 있으므로 **PRD·plan-output 포함/제외**를 1차로 둔다(`AGENTS.md` 대외·문헌 정합 단락 참고).

## 유지보수 시

**단순 수정 스코프 (경량·단건):** 요청과 무관한 영역은 건드리지 않는다.

- **파일:** 요청에 명시된 경로만 수정한다.
- **섹션:** 같은 파일이라도 요청한 블록·기능 범위만 변경한다.
- **전역 스타일:** 공통 레이아웃·전역 CSS·무관 컴포넌트 스타일 변경 금지.
- **파일럿 DOM id:** 파일럿이 의존하는 **id·숨은 버튼·와이어 싱크** 임의 변경 금지 — 필요 시 `docs/PILOT_FUNCTIONAL_SPEC.md` §9~§10·`plannode-architecture.mdc`와 대조.

- 요청 범위 밖 파일은 수정하지 않는다. 스타일·패턴은 기존 파일과 통일한다.
- Supabase·Vercel·DNS·RLS 변경은 `.cursor/plans/PLANNODE_INTEGRATED_GUIDE.md` 및 환경 변수 영향을 함께 검토한다.
- git 명령은 Stephen만 직접 실행한다. AI가 자율 커밋·푸시하지 않는다.

## 서브에이전트·Task 활용

- **넓은 탐색**(폴더 구조, 여러 파일에 걸친 동작): Task `explore`로 위임하고, 결과만 요약해 반영한다.
- **CLI·배포·git·장시간 명령**: Task `shell` 또는 터미널 도구를 쓰고, 실패 시 대안을 시도한다.
- 한 턴에서 동시에 여러 독립 탐색이 필요하면 병렬 Task를 고려한다.
- 서브에이전트 ID·내부 로그는 사용자에게 노출하지 않는다.

---
> Source: [pseriesadmin/plannode](https://github.com/pseriesadmin/plannode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
