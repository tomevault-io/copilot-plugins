## plannode

> 서비스명   : Plannode (plannode.pseries.net)

# AGENTS.md — Plannode
# Harness Flow v1.0 (Plannode 전용 적용)
# 역할: 프로젝트 정체성 + 도메인 규칙 선언

---

## 🏗️ 프로젝트 정체성

```
서비스명   : Plannode (plannode.pseries.net)
성격       : 상용 웹앱 개발계획 협업 서비스 (노드 트리·팀 동기화·PRD/명세/IA/와이어)
개발방식   : 1인 AI 에이전틱 코딩(운영) | Cursor AI | Harness Flow v1.0
             ※ 제품·시스템 설계는 상용 협업 기준 — plannode-prd.mdc §1.05~§1.06
스택       : Vanilla HTML/CSS/JS (index.html + plannode.js) → SvelteKit 이행 중
             Supabase (PostgreSQL + Auth + RLS) | Vercel | 가비아 DNS
현재 단계  : 파일럿(Vanilla) 기능 검증 완료 → SvelteKit 포팅·Supabase 연동 진행 중
```

---

## 🌳 트리뷰 핵심 보호 헌장 (비가역)

Plannode의 **1순위 핵심 기능**은 **트리뷰**에서의 **노드 작성·편집**과 **`parent_id` 기반 트리 구조**(루트·순환 방지·레이아웃 맵)의 **정확성·신뢰성**이다. PRD·IA·기능명세·내보내기·LLM 파이프라인은 **이 위에 얹는 부가 층**이며, 아래를 **모든 변경·에이전트·지침**에 강제한다.

| 원칙 | 규정 |
|------|------|
| **환경 보호** | `#V-TREE`, `#CW`, `#CV`, `#EG`, `#SG`, 파일럿 `render()`·줌·팬·간선·미니맵·노드 DOM 계약은 **최소 침습**으로만 수정한다. 부가 뷰(PRD/기능명세/IA/AI)의 CSS·전역 스타일·이벤트 위임은 **트리 영역을 가리거나** `.view` / `.view.active` 전환을 **깨는 특이도·z-index·overflow**를 두지 않는다. |
| **저해 요소 최소화** | 노드 데이터의 **단일 진실(SSoT)** 는 `nodes`·스토어·`treeText`·파일럿 상태와 `pilotBridge` 동기 경로다. 부가 기능은 **읽기/동기** 또는 명시된 `metadata` 확장으로만 붙이고, 트리 편집을 우회하는 **둘째 저장소**·암묵적 이중 계약을 만들지 않는다. |
| **연결 안정성** | Svelte `activeView` ↔ `pilotSetActiveView`, 툴바·뷰 메뉴, `onPersist` / `hydrateFromStore`, 숨은 버튼 id·와이어 싱크는 **쌍으로** 유지한다. DOM id·스토어 계약 변경 시 `docs/PILOT_FUNCTIONAL_SPEC.md` §9~§10·`plannode-architecture.mdc`와 **대조**한다. |
| **변경 시 재검증** | 트리·파일럿·브리지·스토어·`+page.svelte` 뷰 셸 중 **하나라도** 변경하면 `npm run build`와 **트리 기본 시나리오**(프로젝트 열기·노드 추가/이동·저장·트리↔타 뷰 전환)를 **수동 또는 TASK/GATE C**에 한 줄 이상 명시한다. 하네스 NOW에는 **회귀 범위**를 남긴다. |

상위 교차 참조: `.cursor/rules/plannode-core.mdc` **「트리뷰 핵심 보호」**, **GP-13** 아래.

---

## 📚 문서 위계 (에이전트·기획·개발 공통)

| 구분 | 문서 | 용도 |
|------|------|------|
| **제품 진실(단일 기준)** | `.cursor/rules/plannode-prd.mdc` | M1~M6, F#-#, Phase, IA≠AI, LLM(§10), v2 DB·4-레이어·성공기준(§7) — **“무엇·왜·어느 단계”** |
| **통합 개발 로드맵** | `.cursor/plans/plannode_integrated_milestone_v3.md` | M1~M5·H1~H4·M2-CORE **단계별 순서** · 하네스 §0·§7과 연동 |
| **파일럿 기준(동작·포팅)** | `docs/PILOT_FUNCTIONAL_SPEC.md` | `index.html`+`plannode.js` 분해, SvelteKit **정합·갭(§9~§10)** |
| **배포·인프라** | `.cursor/plans/PLANNODE_INTEGRATED_GUIDE.md` | Git, Supabase, Vercel, DNS |
| **절차(하네스)** | `AGENTS.md` + `.cursor/harness/*` + `.cursor/agents/*` | @promptor → TASK.md → @harness-executor → @qa, GATE — **“어떻게 작업·검수”** |
| **워크플로·복붙(가이드)** | `.cursor/plans/harness-workflow_final.md` | **기본·단축** 모드 분리 · GATE · 채팅 **복붙 블록**(G/R/W) — 상세 견제는 아래 표·`harness/README` |

**PRD vs 하네스:** `plannode-prd.mdc`는 제품·범위의 기준; 하네스는 **그 PRD에 맞는 작업을** 분해·승인·로그로 묶는다. `plan-output.md`·`TASK.md`에는 **PRD 연계**로 모듈·기능 ID·(해당 시) PRD 절(예: §3 F2-4, §10)을 적는다.

---

## ⚖️ 황금 원칙 (모든 에이전트 공통 적용)

```
GP-1.  git 명령어는 Stephen만 직접 실행 (add / commit / push 전부)
GP-2.  모든 GATE 전환은 Stephen의 명시적 승인 후에만 진행
GP-3.  AI는 제안·실행, 결정은 Stephen
GP-4.  DB 스키마 변경은 반드시 신규 마이그레이션 파일 추가 (기존 SQL 수정 금지)
GP-5.  30분 초과 GSD 태스크는 분해 후 재승인
GP-6.  불확실하면 즉시 멈추고 Stephen에게 질문
GP-7.  명시 범위 밖 기능 선제 구현 금지
GP-8.  컨텍스트 리셋 요청 시 TASK.md·plan-output.md 즉시 재로드
GP-9.  Supabase anon 키·서비스 키는 절대 클라이언트 코드에 직접 노출 금지
GP-10. 파일럿(Vanilla) 동작 기준을 SvelteKit 포팅 정합성 체크의 기준으로 삼는다
       → 기능·UX 불일치 발견 시 docs/PILOT_FUNCTIONAL_SPEC.md §9~§10 체크리스트 참조
GP-11. 제품 범위·로드맵·“IA(정보 구조) vs LLM(F2-5)” 구분은 .cursor/rules/plannode-prd.mdc 를 우선한다.
       아젠다·plan-output·TASK.md에는 PRD 추적용으로 M#·F#-# (및 필요 시 PRD 절)을 1줄씩 남긴다.
       PRD·파일럿·통합 가이드가 충돌하면 불일치를 먼저 밝히고 조정한다.
GP-12. 오버 엔지니어링·기술부채 누적을 지양한다. 아젠다·PRD·TASK에 없는 “미래용” 추상·래퍼·중복 계층·
       사용 불명 스크립트·**불필요한 신규 모듈/유틸**을 붙이지 않는다(기존 경로·파일 확장을 우선).
       TODO·debug 로그·any·무분별 의존성은 @qa 2단계(기술 부채) 기준에 맞출 것.
       “보상” 문장만 늘리지 말고 **반복 위반**은 **구조·이름·린트**로 고쳐 **AGENTS/규율**을 **얇게** 유지한다(하네스 **누적=과세** — 엔지니어링 문서 일반론).
GP-13. **트리뷰 핵심 보호:** 제품의 중심은 **트리뷰 노드 작성·트리 구조**다. 관련 로직·모듈·UI 파이프라인은
       **「트리뷰 핵심 보호 헌장」**·`plannode-core.mdc`를 따른다. 부가 메뉴·뷰·스타일이 캔버스·SSoT·뷰 전환을
       **저해하거나 가리는 변경**을 하지 않으며, 해당 영역을 수정할 때는 **재검증·회귀**를 TASK/GATE C·@qa에 남긴다.
GP-14. **상용 협업 포지션:** 제품은 **상용 웹앱 개발계획 협업 서비스** (`plannode-prd.mdc` §1.05). M5·`plannode-architecture.mdc` §10
       (번들·슬라이스·revision·structure_ops·Presence·충돌)을 **“1인용·미니멀·스킵 가능”**으로 설계·축소하지 않는다.
       하네스 경량화(GP-12)는 **PRD·TASK 밖 불필요 모듈** 억제이지 **협업 동기화 견고함** 억제가 아니다(§1.06).
```

### 경량화·오버엔지니어링 **견제** 제어 구조 (하네스)

하네스는 **스코프 밖 로직·모듈 증가**를 **GATE·스코프·검수**로 끊는다. **협업·동기화(M5)는 상용 핵심 경로**이므로 GP-12로 **축소·생략하지 않는다** (`plannode-prd.mdc` §1.06).

| 제어 층 | 수단 | 역할 |
|---------|------|------|
| **0 — 스코프** | `plan-output` **포함/제외**, `TASK` **PRD:** | PRD M#·F#·Phase 밖·“나중에”는 **쓰지 않음**으로 박는다. |
| **1 — 쪼개기** | `GP-5`·NOW **30분·한 파일** | 큰 뼈대·다중 모듈 **한 턴** 금지 → 쪼개서 GATE B. |
| **2 — 승인** | **GATE A/B** | 설계/태스크 목록 **사람 승인** 전까지 구현 확대 금지. |
| **3 — 구현** | `@harness-executor` G-STEP 4, **GP-12** | **확장 전** 동일 요구를 기존 파일·exports로 **만족시킬 수 있는지** 1문장 점검. |
| **4 — 검수** | `@qa` 1~2단계(범위·부채) | **범위 초과(오버엔지니어링 의심)**·불필요 파일/의존성·잔여 로그/TODO. |

> **새 `*.ts`/`lib/…` 하위 모듈** — PRD·TASK·plan-output **어느 줄에도 없으면** 먼저 `BACKLOG`·Stephen 확인 없이 **추가하지 않는다**.

> **대외·문헌 정합 (요지):** *Guides(사전 유도) + Sensors(사후 검출)* — [Martin Fowler, Harness engineering](https://martinfowler.com/articles/harness-engineering.html) — Plannode는 `plan-output`/PRD/GATE가 **가이드**, `@qa`·빌드·린트가 **센서**다. *오버엔지니어링*은 **테스트·센서만**으로 끝까지 잡기 어렵다는 점 → **0층 스코프·GATE**가 **우선**. *층·툴 최소* — [Kruczek, Fewer layers](https://matthewkruczek.ai/blog/agent-harnesses-fewer-layers). *YAGNI* — [Fowler YAGNI](https://martinfowler.com/bliki/Yagni.html). *하네스 = 목차형 AGENTS + 상세 분산* — [OpenAI Harness (요지)](https://www.engineering.fyi/article/harness-engineering-leveraging-codex-in-an-agent-first-world). 운영상 견제 표는 위 **「경량화·오버엔지니어링 견제 제어 구조」**와 `.cursor/harness/README.md` **「최소 구현·기술부채·경량화」**를 본다.

---

## 🔴 Plannode GSD 전용 도메인

Plannode 파일럿은 결제·동시성·권한 로직이 없으므로 **모든 태스크가 GSD 기본**이다.
아래 영역에 한해 주의 강화 체크를 적용한다.

```
캔버스·렌더링  : transform / panX / panY / scale / drawEdges / layoutEngine
노드 구조      : addChild / cDel / getDepth / parent_id / num 계산 / lm 재계산
Supabase 연동  : plan_projects / plan_nodes RLS 정책 · owner_id 일치 여부
SvelteKit 이행 : store 계약 / 컴포넌트 바인딩 / 파일럿 갭 분석(§9)
```

### 주의 강화 체크 (GATE C 포함)

```
캔버스 변경 시:
□ SVG(#EG)와 노드 DOM이 동일 transform 컨테이너 안에 있는가?
□ 줌·패닝 좌표계가 파일럿 applyTx() 방식과 일치하는가?
□ drawEdges / updMM 호출 타이밍이 파일럿과 동일한가?

노드 구조 변경 시:
□ addChild 후 편집 모달 자동 오픈 여부를 결정했는가?
□ parent_id 체인 순환 방지 로직이 있는가?
□ lm(레이아웃 맵)이 render() 마다 재계산되는가?

Supabase 연동 변경 시:
□ owner_id가 auth.uid() 기반인가? (하드코딩 UUID 금지)
□ RLS 정책이 plan_projects·plan_nodes에 모두 활성화되어 있는가?
□ 환경 변수 VITE_SUPABASE_URL·ANON_KEY가 .env.local에만 존재하는가?
```

---

## 🚫 Plannode 절대 금지 패턴

```javascript
// ❌ owner_id 하드코딩 (RLS 우회·인증 불일치)
supabase.from('plan_projects').insert({ owner_id: 'some-fixed-uuid', ... })

// ❌ SVG를 노드 DOM 컨테이너 밖 형제 레이어로 배치 (줌 불일치)
// → SVG(#EG)는 반드시 #CV(transform 대상) 내부에 위치해야 한다

// ❌ render() 없이 직접 DOM 노드 좌표만 갱신 (간선·미니맵 미동기)
nodeEl.style.left = x + 'px'  // drawEdges() + updMM() 없이

// ❌ parent_id에 프로젝트 ID 사용 (노드 id 체계와 혼용)
addNodeChild($currentProject.id)  // 루트 노드 id를 써야 함

// ❌ nodes 배열에 빈 배열 저장 후 루트 자동 생성 로직 미실행
localStorage.setItem(key, '[]')  // openProj 후 루트 노드 미생성
```

---

## 📋 도메인 규칙 파일 목록

```
.cursor/rules/
├── plannode-core.mdc           ← alwaysApply: 프로젝트 컨텍스트·에이전트 원칙
├── plannode-architecture.mdc   ← alwaysApply: 기능 구조·데이터 흐름·파일럿 브리지·Supabase 연동 표준
├── plannode-ui-identity.mdc    ← UI 색·타이포·반응형·레이어·카피 톤 표준
├── plannode-prd.mdc            ← 제품 요구(M/F/Phase, IA/와이어, LLM §10, v2 DB) — @promptor·TASK·QA가 추적
├── plannode-web.mdc            ← Vanilla HTML/JS + Svelte/CSS 편집 시 스택 규칙
├── plannode-badge-mapping.mdc  ← 배지 풀·가져오기·파일럿 추론 게이트
└── README.md                   ← `.mdc` 빠른 인덱스 · `.md` 편집 규약 (rules 폴더)
```

---

## 🤖 에이전트 호출 순서

```
STEP 1  아젠다 정의 (Ask 모드)                          👤 Stephen
         ↓
STEP 2  @promptor → plan-output.md 저장 (Agent 모드)    🤖 Haiku 4.5
         ↓
GATE A  계획 승인                                        👤 Stephen
         ↓
STEP 3  Plan Mode → TASK.md 작성 (Plan 모드 / Composer2) 🤖
         ↓
GATE B  TASK.md NOW 확정                                 👤 Stephen
         ↓
STEP 4  @harness-executor → NOW 루프 (Agent / Haiku 4.5)  🤖
         ↓
GATE C  NOW마다 구현 검증                                👤 Stephen
         ↓
GATE D  전체 구현 확인                                   👤 Stephen
         ↓
STEP 5  @qa 검수 (Agent / Haiku 4.5)                     🤖
         ↓
GATE E  최종 확인 → 커밋 허가 → git commit               👤 Stephen
```

> **경량모드**: 단일 파일·UI 수정·버그 수정 → Step2(@promptor)·GATE A 생략 가능.
> TASK.md 상단에 `경량 경로: step2·GATE A 생략` 한 줄 추가 후 GATE B 진입.

---

## 📁 디렉토리 구조

```
plannode/
├── AGENTS.md                       ← 이 파일 (프로젝트 전용 정체성·원칙)
├── index.html                      ← 파일럿 (기준 UX)
├── plannode.js                     ← 파일럿 로직 (기준 행동 명세)
├── docs/
│   ├── PILOT_FUNCTIONAL_SPEC.md    ← 파일럿 기능 분해 (포팅 기준 문서)
│   └── 하네스 워크플로우 update/   ← 하네스 원본 참조 문서
├── src/                            ← SvelteKit 구현 진행 중
├── .cursor/
│   ├── harness/
│   │   ├── README.md               ← 폴더 역할·DB 절·운영 원칙
│   │   ├── TASK.md                 ← 현재 스프린트 태스크 (AI 자동 갱신)
│   │   ├── plan-output.md          ← @promptor 출력 (GATE A 확정본)
│   │   ├── GSD_LOG.md              ← 실행 이력
│   │   └── context-hook.md         ← 컨텍스트 드리프트 방지 훅
│   ├── agents/
│   │   ├── harness-executor.md     ← Step4 실행 에이전트
│   │   ├── promptor.md             ← Step2 플래너 에이전트
│   │   └── qa.md                   ← Step5 QA 에이전트
│   ├── rules/
│   │   ├── plannode-core.mdc
│   │   ├── plannode-architecture.mdc ← 구현 아키텍처·데이터 흐름 표준
│   │   ├── plannode-ui-identity.mdc  ← UI 아이덴티티 표준
│   │   ├── plannode-prd.mdc          ← PRD(제품 진실)
│   │   ├── plannode-web.mdc
│   │   ├── plannode-badge-mapping.mdc
│   │   └── README.md               ← rules `.mdc` 인덱스
│   └── plans/
│       └── PLANNODE_INTEGRATED_GUIDE.md ← Git·Supabase·Vercel·DNS (경로: `.cursor/plans/`)
└── .env.local                      ← 환경 변수 (Git 제외)
```

---

*AGENTS.md | Plannode | Harness Flow v1.0 | 2026.04*

---
> Source: [pseriesadmin/plannode](https://github.com/pseriesadmin/plannode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
