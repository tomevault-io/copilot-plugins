## plannode-prd

> Plannode PRD — 상용 웹앱 개발계획 협업·M5·IA·와이어·LLM(§10)·DB·로드맵(§1.05~§1.06)


# Plannode PRD (Product Requirements Document)

**버전**: 1.4 (1.3 + **상용 웹앱 개발계획 협업 서비스** 포지셔닝 §1.05·§1.06, M5·로드맵 정합)  
**작성·갱신**: 2026-04-22 (§1.05·§1.06: 2026-06-04)  
**상태**: Development  
**제품 포지션(한 줄)**: **상용 웹앱 개발계획 협업 서비스** — 노드 트리 기반 구조 설계·팀 동기화·PRD/명세/IA/와이어 산출  
**상세 TypeScript/스키마 예시**: `.cursor/plans/plannode-ai-enhancement-v3.md` 참고

**구현 스택 정합성**: **배포 앱의 UI·라우팅·동기화 껍질**은 SvelteKit+TypeScript(`src/routes`, `src/lib/stores`, `src/lib/supabase`)이며, **트리 캔버스·줌·간선·문서 패널의 실행 단일 축**은 내장 파일럿 `src/lib/pilot/plannodePilot.js` + `pilotBridge.ts`가 담당한다(동작·포팅 기준은 여전히 `docs/PILOT_FUNCTIONAL_SPEC.md`, 루트 `index.html`+`plannode.js`와의 갭은 동 문서 §9~§10). **v2 LLM·DB 목표**(§10·§11)는 제품 방향의 진실로 두되, **클라이언트에 이미 존재하는 모듈**(`src/lib/ai/*` 일부: 직렬화·매트릭스·모델 선택·IA보내 보조 등)과 **Supabase에 실제로 올라간 스키마·RPC**를 우선해 코드·PRD 불일치 시 본문 또는 `plannode-architecture.mdc`에서 명시한다.

---

## 1. 서비스 개요

### 1.0 현재 구현된 시스템 (코드 기준 요약)

아래는 **요구사항이 아니라 저장소에 존재하는 구현**을 한 장으로 묶은 것이다. 세부 흐름·클라우드 동기·협업 계약은 `.cursor/rules/plannode-architecture.mdc`(특히 **§10**).

| 층 | 구현 내용 |
|----|-----------|
| **앱 셸** | `+layout.svelte`: Supabase 환경 가드·스플래시·`LoginGate`·세션(`authSession`) 후 슬롯. 클라우드 설정 시 로그인 뒤 `loadProjectsFromLocalStorage()`. |
| **메인 오케스트레이션** | `+page.svelte`: 툴바·뷰 전환·모달·클라우드 플러시·Presence 연동. 뷰는 `activeView`: `tree` \| `prd` \| `spec` \| `ia` \| `ai` — 파일럿 `pilotSetActiveView`와 쌍을 맞춘다. |
| **파일럿 런타임** | `plannodePilot.js` (`initPlannode`): 노드 DOM·SVG 간선·미니맵·PRD/기능명세/IA/AI 패널 갱신. 트리 편집 중 **단일 진실**은 파일럿 상태이며, 저장 시 브리지로 스토어에 반영된다. |
| **브리지** | `pilotBridge.ts`: `onPersist` → `pilotNodesToStore` → `persistNodesFromPilot`(localStorage + 더티 마킹), `currentProject` 변경 시 `hydrateFromStore`, 필요 시 액세스 토큰·`plan_project_id` 콜백. |
| **클라이언트 상태** | `projects.ts` 등: `Project`·`Node` 플랫 목록, `activeView`, `plannode_projects_v3` / `plannode_nodes_v3_<id>` / `plannode_current_project_v3`. 루트 노드 규칙은 아키텍처 문서 §4와 동일. |
| **Supabase(설정 시)** | `client.ts`·`env.ts`: 미설정 시 안전한 플레이스홀더. **워크스페이스**: 사용자별 `plannode_workspace` JSON 번들 업서트/풀, 업로드 전 `mergeRemoteWorkspaceBeforeUpload`(LWW 성격), `workspacePush`·`cloudBackgroundSync` 디바운스·주기·가시성 트리거. **협업**: ACL·공유 프로젝트 슬라이스·revision/lock RPC 경로(`sync.ts` 등). **Presence**: Realtime으로 **선택 노드 등 메타**만 — 노드 본문은 번들·RPC 경로(번들 Realtime 스트리밍 아님). |
| **데이터 교환** | `plannodeTreeV1.ts`: 트리 v1 스키마 파싱·가져오기/업서트(백업·이식). |
| **AI / v2 클라이언트(부분)** | `src/lib/ai/*`: `contextSerializer`, `promptMatrix`, `modelSelector`, IA보내·배지 파이프 등 **클라이언트 모듈**이 존재한다. **§11 전면**(예: `ai_generations` 영속, `plan_nodes.path` 트리거 일원화)은 로드맵·DB 마이그레이션과의 **갭**으로 본다 — 구현 시 PRD·SQL·아키텍처를 한 번에 맞출 것. |

**한 줄:** 로컬·파일럿에서 트리를 편집하고, Svelte 스토어·localStorage가 1차 저장이며, Supabase가 켜지면 **워크스페이스 번들 + ACL·슬라이스 + Presence**가 그 위에 얹힌다 — **팀이 같은 프로젝트 구조를 신뢰할 수 있는 협업 층**이 핵심이다.

### 1.05 제품 포지셔닝 (에이전트·기획 공통 — 정본)

| 항목 | 내용 |
|------|------|
| **포지션** | **상용 웹앱 개발계획 협업 서비스** — 웹·앱 **기능·화면·요구**를 노드 트리로 설계하고, PRD·기능명세·IA·와이어를 **팀이 동일 구조**로 편집·동기화·보내는 SaaS |
| **폐기 라벨** | “1인 내부 플래닝 도구”·“개인용 메모 수준” — **PRD·규칙·에이전트 기본 가정에서 사용 금지**. 저장소 **개발 운영**이 1인 에이전틱이어도 **제품·시스템 설계 완성도**는 **상용 협업** 기준을 따른다. |
| **핵심 가치 축** | (1) 트리 SSoT·구조적 합의 (2) **팀 공유·클라우드 동기화**(M5·§3) (3) 기획문서·IA/와이어 산출 (4) (선택) LLM·하네스 품질(§10) |
| **현행 코드 상한** | 프로젝트 접근 **최대 5계정**(소유자+멤버 4) — `plannodeCollabLimits.ts`. **베타·상용 확장**에서 인원·역할·충돌 UX는 단계 강화(§6 Phase 2+) |

### 1.06 에이전트 구현·설계 기준 (하네스 「경량화」와 구분)

`AGENTS.md` **GP-12**·하네스 **경량화**는 **PRD·TASK·plan-output 밖**의 불필요한 **신규 모듈·추상·미래용 뼈대** 억제이다. **아래는 “경량화”로 생략·축소하지 않는 상용 협업 정상 경로**다.

| 구분 | 에이전트가 따를 기준 |
|------|---------------------|
| **협업·동기화(M5)** | `plannode-architecture.mdc` **§10** — 번들·슬라이스·revision·structure_ops·Presence·모달 편집 중 pull 보류 등 **계약을 온전히** 유지·수정한다. “1인이면 pull/slice/ops 생략해도 됨” 설계 **금지**. |
| **동기화 결함** | meta drift false-skip·ACL 403·slice 누락 등은 **버그·불변식 위반** — BACKLOG·“나중에”로 미루지 않는다(§7·architecture §10.11~§10.12). |
| **알려진 한계** | OT/CRDT·필드 단위 병합 없음(LWW)은 **문서화된 제품 한계**이지, **번들/ops/revision 경로 자체를 단순화**할 명분이 아니다. |
| **오버엔지니어링** | PRD M#·F#·TASK에 **없는** 범용 프레임워크·중복 저장소·v2 LLM 전면 선구현은 여전히 **금지**(GP-12). **협업 경로의 견고함**과 **스코프 밖 확장**을 혼동하지 말 것. |

### 1.1 제품 정의

**Plannode**는 **상용 웹앱 개발계획 협업 서비스**(§1.05)로, **AI를 활용하는 기획 보조**(§10, 선택)와 **정보 구조(IA)·문서/와이어 산출**을 **동일한 제품 가치**로 둔다. 제품 기능을 노드 트리로 시각 설계하고, **PRD·기능명세**에 더해 **정보 구조(IA) 문서·와이어프레임(저충실도)**·(확장 시) API/ERD 등으로 **변환·내보내는** **웹 기반** 도구다.

**용어 (혼동 방지, 에이전트·기획자 공통)**  
- **IA** = *Information Architecture* = **정보 구조** — 내비·화면(또는 모듈) 계층, 라벨, 사용자 이동 경로. **LLM이 아님.** 트리에서 **도출·렌더·내보내기**하는 **구조 기반 산출**이 본질.  
- **AI** = 본 문서에서 **인공지능/LLM**(Claude 등)으로 **의미**를 통일(§10). “AI 기획” 문구는 **LLM 지원**을 뜻하며 **IA(정보 구조)와 별도**다.

**핵심 문제 → 해결**

- 이미지/수동 PRD: 불일치·동기화 비용
- 팀 협업: 댓글 수준 → 구조적 합의 어려움
- **AI 개발 기획문서** 입력: **노드 텍스트만 전달 시 맥락 소실** → 일반론적 출력(§10)

**가치 제안**

**노드 맵(구조) → 품질 있는 PRD/명세·IA/와이어·(선택) LLM/하네스** — (1) **IA·와이어**는 **트리 구조에서 직접** 도출 가능해야 하고, (2) **기획문서( PRD/명세·하네스) LLM 품질**은 **ContextSerializer(10.2)로 컨텍스트 인코딩**이 핵심(§10).

### 1.2 USP (요지)

- 노드 기반 **자동 PRD/MD** 및 뷰
- **IA(정보 구조) + 와이어프레임** — 트리 → 화면(또는 모듈) **계층·이동 경로**·**저충실도 블록/구역** 문서(뷰+내보내기, §3 F2-4). *Figma 수준의 시각디자인 도구는 목표가 아님(§1.3).*
- Supabase **권한·워크스페이스 번들 동기·ACL·Presence**(설정 시); **노드 단위 실시간 공동편집(OT/CRDT)** 및 **§11 DB 전부**는 여전히 로드맵·갭으로 본다(`plannode-architecture.mdc` §10.9 한계).
- (계획·권장) **LLM: PRD/기획문서 심화·누락·하네스 등(§10)** — `content`만 넘기지 않고 v2 4-레이어로 전달. *선택으로 IA/와이어*에 대한 **설명/섹션 초안**을 LLM이 보강하는 것은 허용(§3 F2-4·§10.4).
- **영속성** — **localStorage**(현행 1차) + Supabase **Postgres**(`plannode_workspace` 번들 등, 설정 시); §11 수준의 노드 정규 스키마·`ai_generations` **전면**은 로드맵

### 1.3 타겟

- **주 타겟**: 스타트업·에이전시·소규모~중소 **제품·기획·UX 팀** — 웹/앱 **개발계획·PRD·IA**를 **노드 맵 + 협업**으로 맞추려는 조직
- **사용 맥락**: 비개발 기획자 중심이어도 **Cursor/AI 개발·하네스**로 이어지는 워크플로
- **Non-target**: 엔터프라이즈 SSO·세밀 RBAC·Figma/고충실 프로토타입 **대체**

### 1.4 비즈니스 단계(요지)

- Phase 1: **상용 협업 MVP** — 트리·공유(ACL)·클라우드 동기화·배포 안정(현행 5계정 상한 내)
- Phase 2: **베타·상용 강화** — **F2-4/4-3/4-4(IA/와이어)**·**F2-5 LLM**·**F5-2 동기화·충돌 UX**·스냅샷(F3-3)
- Phase 3: **상용 확장** — 역할·인원·충돌 알림·**LLM v2**·도메인 사전 — **IA 산출은 트리 기반 유지**

---

## 2. 기술 스택 (현행 + 목표)

| 레이어 | **현재 저장소 기준(구현)** | AI/클라우드 목표(v2·PRD 1.0 방향) |
|--------|---------------------------|-------------------------------------|
| Frontend | **SvelteKit + TS** 앱 셸 + 내장 **파일럿(JS)**; 클라이언트 AI 보조 코드는 `src/lib/ai/*`에 **부분 존재** | v2 모듈 경로·타입 완결, 필요 시 Vanilla 포팅 시 **동일 4-레이어 계약** |
| Backend/DB | **localStorage** 1차 + Supabase **`plannode_workspace`** 번들·Auth·RLS·ACL·RPC(설정 시) | §11 `plan_projects`/`plan_nodes` 확장·`path`·`ai_generations` 등 **제품 DB 진실** 완성 |
| 실시간 | **Presence**(프로젝트 채널, 선택 노드 메타 등) — 노드 데이터는 번들 경로 | PRD 1.0 방향의 협업·알림 확장 시에도 **데이터면과 메타면 분리** 원칙 유지 |
| AI | 파일럿 **AI 뷰** + `src/lib/ai/*`(직렬화·매트릭스·모델 선택·IA exporter 등 **클라이언트 층**) | §10 4-레이어 완주 + **`ai_generations` 서버 영속** |
| 배포 | Vercel 등(통합 가이드 준수) | 동일 |

---

## 3. 기능 요구사항 (모듈 요지)

### M1. 노드 트리 에디터

- **F1-1 노드 CRUD**: 무한 뎁스, `name`, `description`, `num`, `node_type` (기존: root|module|feature|detail) — **v2 보강**: LLM·기획문서를 위해 `spec|constraint|decision|risk` 등 확장이 DB·UX에서 허용되면 직렬화·매트릭스(§10.2)에 반영
- **F1-2 캔버스**: 드래그, 패닝, 준, 맞춤, 미니맵
- **F1-3 배지**: TDD|AI|CRUD|API|USP — PRD/명세·LLM 리스크 감지 키워드와 정합(§10.2·10.3)

### M2. 시각·문서 뷰

- **F2-1 트리 뷰 · F2-2 PRD · F2-3 기능명세**: PRD 1.0 수준 유지(마크다운, TDD/AI 배지 섹션 등)

#### F2-4. IA(Information Architecture) + 와이어프레임 **— LLM이 아닌 구조 산출(핵심 구분)**

| 구분 | IA(정보 구조) | 와이어프레임 |
|------|----------------|--------------|
| **목적** | 화면/모듈**계층**, 내비·**라벨**, 주요 **사용자 이동 경로**를 문서화 | 화면별 **저충실도** 레이아웃 — **블록/섹션/우선순위** (박스·영역) |
| **입력** | 동일 **노드 트리** + (선택) 노드 `description` | 동일; 화면마다 대표 노드(또는 L2~L3) 매핑 |
| **출력 형태** | 마크다운 목차/표, **Mermaid** `flowchart`/`graph`(선택), `*.md` 내보내기(§4) | 마크다운+ASCII/블록 다이어그램, **Mermaid** 블록도(선택) |
| **LLM** | **필수 아님** — 1단계는 **트리→템플릿 렌더**로 충분 | 동일. *선택*: §10로 **와이어 카피·빈 섹션 보강** |
| **오인 방지** | “IA”를 **채팅 AI 기능**이나 F2-5와 **같은 메뉴**로만 묶지 말 것 — UI 라벨에 *정보 구조(IA)* 병기 권장 | Figma/고충실 프로토타입 **대체 아님** |

- **F2-5. LLM/AI 분석**(API 기반, §10) — *기획문서 최적화·“AI 분석 뷰”*는 **이 항목만** 해당 (PRD 심화, 하네스, 갭, 리스크 등)

| 버튼/의도(개념) | PRD 1.0 | v2 정렬 |
|------------------|---------|--------|
| PRD/기획문서 심화 | “완전한 PRD” | `OutputIntent.PRD` + LAYER1~3 |
| 누락/갭 | 누락 탐지 | `validate` 단계 + [GAP] |
| TDD/리스크 | TDD 분류 | `RISK_ANALYSIS` 매트릭스 + Sonnet 강제(§10.3) |
| 하네스 | Cursor 프롬프트 | `serializeToPrompt` 결과가 입력 코어 |
| (추가) | — | `USER_STORY`·`API_SPEC`·`ERD`·`STATE_MACHINE` 등 `OutputIntent` (v2) |
| (선택·IA/wire) | — | `IA_SITEMAP`·`WIREFRAME_COPY` 등 — **구조는 트리 고정, LLM은 문장/빈칸만** (§10.4) |

**F2-5 수용(목표)**: API 키/환경 변수, 응답·토큰 한도(v2: 단계별 maxTokens), **구조 맥락 누락 시** 재생성·호출 **거부(설계 원칙)**.

### M3. 데이터

- **F3-1 localStorage** — MVP, 용량·성능 PRD 1.0 수준
- **F3-2 Supabase** — `plan_projects`·`plan_nodes`·협업·스냅샷 + **v2 보강(§11)**: `node_type`·`depth`·`path`·`metadata`·`plan_node_relations`·`ai_generations`
- **F3-3 스냅샷/버전** (Phase 2) — PRD 1.0

### M4. 내보내기 (기획문서 출력)

- **F4-1** Feature Map **MD** · **F4-2 PRD** — PRD 1.0, 파일명·섹션 유지
- **F4-3 IA(정보 구조)**: `{프로젝트명}-ia.md` (또는 합본 섹션) — 목차/경로/라벨·선택 Mermaid
- **F4-4 와이어프레임 키트**: `{프로젝트명}-wireframes.md` — 화면별 블록/섹션(저충실), PRD/IA와 **교차 링크** 가능

### M5. 협업 (상용 핵심 — Phase 1~2)

**M5는 부가 기능이 아니라 §1.05 상용 포지션의 핵심 모듈**이다. 구현·에이전트는 `plannode-architecture.mdc` §5·§10·§10.10~§10.11을 정본으로 한다.

- **F5-1 공유**: 프로젝트 ACL·초대·`cloud_workspace_source_user_id` — 이메일 기반(현행), `user_id` RLS 전환(로드맵)
- **F5-2 팀 동기화**: 워크스페이스 번들·공유 슬라이스·revision 신호·structure_ops·Presence — **eventual 일관성**(수 초), LWW·OT/CRDT 없음은 **명시 한계**
- **F5-3 (상용 품질)**: 동기화 결함·충돌·권한 오류는 **상용 서비스 결함**으로 취급 — “내부 1인용이면 충분” 기준 **금지**(§1.06)
- RLS·owner_id·협업 RPC는 v2·배포 가이드·`docs/supabase/`와 병행

### M6. 프로젝트 CRUD/목록 — PRD 1.0

---

## 4. 비기능 (요지; PRD 1.0 유지 + AI)

| 항목 | 기준 |
|------|------|
| 응답/로드/스토리지/브라우저 | PRD 1.0 |
| **IA/와이어** | **구조 산출**는 LLM 없이 **동일 트리 → 동일 뼈대** 재현; 내보내기 ≤ PRD/MD 수준 응답 |
| **LLM(§10)** | v2: 저위험·Haiku/고위험·`validate`·`STATE_MACHINE`/`RISK`는 **Sonnet 강제(§10.3)**, `detectHighRiskContext` |
| **path/트리거** | v2: **path 오염 = 전체 컨텍스트 오염** — 트리거 TDD |
| **보안** | RLS, `ai_generations`에 context 스냅샷(디버그·감사) |

---

## 5. UI/UX (PRD 1.0 + IA/LLM 구분)

색·뎁스·배지·탑바: 기초 PRD 5절. **탭/메뉴**: **“정보 구조(IA) / 와이어”**와 **“AI 분석(LLM)”**을 **라벨**로 구분(영문 `IA` 단독은 *Information Architecture*임을 툴팁 등으로 병기 가능).

---

## 6. 구현 로드맵 (통합)

| 구간 | 내용 |
|------|------|
| **MVP** | PRD 1.0 체크리스트(에디터·뷰·localStorage·내보내기·프로젝트) — **F2-4/4-3/4-4**는 **Phase 2** 우선(일정은 구현 시 조정) |
| **Phase 1** | Supabase, F5-1, 배포(가이드) + **v2 1~2주차: DB(path·relations·ai_generations) + LAYER1 types·contextSerializer** |
| **Phase 2** | **F2-4 IA/와이어** 뷰·**F4-3/4-4** 내보내기(우선), F2-5(LLM), F5-2, F3-3, **v2 LAYER2~3** |
| **Phase 3+** | **LAYER4 domainDictionary** (rental, b2g_saas, …) — 크레이지샷·도메인 킥오프 전 필수( v2) |

기술 명세 **주차별** 세부(파일 단위)는 `plannode-ai-enhancement-v3.md` PART C.

---

## 7. 성공 기준 (보강)

- PRD 1.0 메트릭 유지
- **협업(M5)**: 2프로필 이상 공유 프로젝트에서 **노드 구조·개수**가 수 초~수십 초 내 **양방향 수렴**(동일 id·parent_id·주요 필드; LWW 한계는 문서화)
- **협업 신뢰**: revision·ops·slice **skip 오판**(meta drift 등) 없음 — architecture §10.11 불변식 준수
- **LLM(§10)**: 동일 노드+인텐트에서 `serializeToPrompt` 없이 “텍스트만” 호출 경로 **금지(설계 원칙)**
- **IA/와이어(F2-4)**: **트리 기반** 산출이 **재현 가능**(동일 트리 → 동일 뼈대; LLM 보강은 선택·버전 기록)
- `path`·`node_type` 일관성, `getLatestGeneration` 캐시·재사용 UX

---

## 8. 위험 (PRD 1.0 + v2)

- RLS/실시간/충돌 — PRD 1.0
- **path 트리거·마이그레이션 실패** — DB 컨텍스트 전부 틀어짐
- **modelSelector 잘못** — 결제·동시성 구간 Haiku 취약
- **도메인 사전 누락** — 잘못된 도메인 일반론(재고/결제 등), v2 8절

---

## 9. 향후 (PRD 1.0)

i18n, 슬랙, Figma, TS 마이그레이션, 오픈소스 등 — 기초 PRD 9절; AI·기획문서 모듈은 v2 트리와 병행해 확장

---

## 10. LLM: 기획문서 출력 품질·구조적 컨텍스트 (v2 핵심)

### 10.1 문제

노드 클릭 → `node.content`만 API에 넣으면 **depth·타입·계층·비계층 관계**가 사라져 **기획문서( PRD/하네스)가** 얕게 나온다. **필수**: `ContextSerializer` → **구조화된 컨텍스트 패킷** → LLM.

### 10.2 4개 레이어(우선순위)

1. **LAYER1 — 컨텍스트 직렬화(최우선)**: `NodeContext`(current, ancestors, siblings, children, relations, `projectMeta`의 `domain`·`techStack`·`outputIntents`), `serializeToPrompt` , `buildContextFromDB` (또는 Vanilla 동등). **LAYER1 없이 이후 의미 없음.**
2. **LAYER2 — 출력 인텐트 × 노드 타입 매트릭스**: `getSystemPrompt(nodeType, outputIntent)` — 추상 서술 금지, 엣지케이스·수치·형식(ADR, Mermaid, AC 등) **프롬프트에서 강제**.
3. **LAYER3 — 다단계 파이프**: Skeleton(골격) → Deepen(섹션 병렬) → Validate(PR·상태기계 등, [GAP] 보강) — 2/3 stage, `GenerationResult`에 `pipeline`·`modelUsed`·토큰 기록.
4. **LAYER4 — 도메인 사전**: `injectDomainContext` (rental 재고/상태전이/버퍼, b2g_saas 권한·감사 등).

### 10.3 모델 선택(정책)

- `selectModel` / `SONNET_REQUIRED_CONDITIONS`: `validate` 단계, 결제·동시성 맥락, `STATE_MACHINE`, `RISK_ANALYSIS`, (도메인별) `API_SPEC` 등 v2 **고위험 규칙** 준수 — **TDD 권장(v2, AGENTS.md 방향과 정합).**
- `detectHighRiskContext`: 결제/환불/동시/잠금 등 키워드.

### 10.4 `OutputIntent` (v2, 예) — **IA(정보 구조)와의 관계**

- **LLM·구조·명세 중심**: `PRD` | `USER_STORY` | `API_SPEC` | `ERD` | `STATE_MACHINE` | `RISK_ANALYSIS` (및 제품이 추가하는 항목)
- **선택·IA 보조**(이름은 구현 시 조정): `IA_SITEMAP` (내비/라벨**카피**·요약) | `WIREFRAME_COPY` (블록**설명**·플레이스홀더) — **골격은 항상 트리(§3 F2-4)에서 결정**하고, LLM은 **문장 보강**에 한정(구조 뒤집기 금지)
- `NodeType` (v2, 예·확장): `root` | `feature` | `spec` | `constraint` | `decision` | `risk` — 기초 `module` 등과 **매핑**은 구현 시 명시

### 10.5 UX

진행: “골격” → “섹션 N/M” → “검증”. 동일 (node, intent)의 최근 생성 `getLatestGeneration` / 재사용.

---

## 11. DB/스키마 요구 (v2 — 제품+기술)

- **`plan_projects`**: `domain`, `tech_stack[]`, `output_intents[]`, `owner_id` ( v2, PRD 1.0 필드와 통합)
- **`plan_nodes`**: `content`, `position` 외 **필수**: `node_type`, `depth`, **`path`(uuid[] 조상)**, `metadata` jsonb — **path + 트리거** `update_node_path` ( v2)로 조상 O(1)
- **`plan_node_relations`**: `depends_on` | `conflicts_with` | `references` | `derived_from` …
- **`ai_generations`**: `output_intent`, `pipeline_stage`, `model_used`, skeleton/deepened/validated/final, `context_snapshot`, `token_usage`

**세션·마이그레이션 순서(필수)**: DB( path·RLS) 선행 → `types` → `contextSerializer` — **DB 없이 AI 전부 짜기 금지**( v2 “주의사항”)

---

## 12. AI 모듈 의존 (v2, 요지)

`PlanNode(DB)` → `contextSerializer` → `promptMatrix` → `generationPipeline` → `modelSelector` + `domainDictionary` → `generationStore`

상세 **의존 다이어그램·`saveGeneration` / `getLatestGeneration`**: v2 PART B (계획서가 있는 경우).

---

## 13. Cursor/에이전트용 작업 쪼개기( v2, 요지)

- **세션1**: path·트리거·relations·`ai_generations`·RLS, path TDD
- **세션2**: `types`, 직렬화, 매트릭스, `modelSelector`, 2-stage 파이프, store — **modelSelector TDD**
- **세션3**: `domainDictionary` + 3-stage validate( PRD, STATE_MACHINE)

원문 프롬프트 템플릿: `plannode-ai-enhancement-v3.md` PART D.

---

## 14. 결론

Plannode는 **상용 웹앱 개발계획 협업 서비스**(§1.05)로, **(1) 팀이 신뢰하는 노드 트리·클라우드 동기화(M5)**, **(2) PRD·명세·IA/와이어 산출**, **(3) (선택) LLM·하네스 품질(§10)** 이 **동등한 제품 축**이다. IA≠LLM(§3). **구현**은 `plannode-core`·`plannode-architecture` §10·배포/DB·AGENTS와 정합 — **협업 경로를 미니멀하게 축소하는 설계는 PRD 위반**(§1.06).

**문서 히스토리**

- v1.1: `plannode-ai-enhancement-v3`(구 v2) 및 초기 PRD 초안 통합
- v1.2: **IA(정보 구조) + 와이어** (F2-4, F4-3/4-4), **F2-5=LLM**, IA≠AI
- v1.3: **§1.0 현재 구현 시스템 요약** 추가, §2 스택 표를 **코드 기준**과 목표로 이원화, §1.2 USP의 Supabase 문구를 구현·갭에 맞게 수정
- v1.4: **§1.05 상용 웹앱 개발계획 협업 서비스** 포지셔닝, **§1.06 에이전트 구현 기준**, M5·§7·Phase 재라벨 — “1인 내부 도구” 가정 폐기
- **복구**: 2026 — `plannode-prd.mdc` 재생성(저장소 누락)

**승인(기초 PRD)**: Stephen Cconzy (PSeries) — 2026-04-21

---

**에이전트 지시**: **제품 포지션·M5 협업·구현 완성도**는 **본 PRD §1.05~§1.06·§3 M5·§7**이 정본이다. “1인·내부·미니멀”로 **동기화·ACL·충돌 경로를 축소·스킵**하지 말 것. 기획문서·**IA/와이어 vs LLM(§10)**·DB는 동일 문서를 따른다. **IA = Information Architecture**. 코드·TypeScript는 **v2**·실스키마 우선; 협업은 **`plannode-architecture.mdc` §10** 우선. 애매하면 `.cursor/plans/PLANNODE_INTEGRATED_GUIDE.md`·`plannode-core.mdc`와 **불일치를 명시**하고 수정안을 제시한다.

---
> Source: [pseriesadmin/plannode](https://github.com/pseriesadmin/plannode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
