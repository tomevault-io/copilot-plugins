## biz-health-web

> 이 파일은 모든 에이전트와 명령이 자동으로 읽는 프로젝트 공통 맥락이다.

# Biz Health Dashboard (Web) — 프로젝트 공통 컨텍스트

이 파일은 모든 에이전트와 명령이 자동으로 읽는 프로젝트 공통 맥락이다.  
글로벌 공통 규칙은 `/home/ubuntu/CLAUDE.md` 참조.  
역할별 상세 규칙은 각 에이전트 파일을 참조하라.

---

## ⚠️ Leo(pm) 필수 준수 규칙 — 전문 에이전트 협업 의무

**Bennhee가 지시한 작업은 반드시 전문 에이전트와 협업하여 수행한다. Leo 단독으로 처리 금지.**

| 작업 유형 | 전담 에이전트 | Leo 단독 처리 여부 |
|---------|------------|:---:|
| API 설계·구현 | developer | ❌ 금지 |
| 코드 리뷰 | reviewer | ❌ 금지 |
| 데이터 검증 | validator | ❌ 금지 |
| 아키텍처·구조 변경 검토 | architect | ❌ 금지 |
| KPI/지표 정의 | felix (kpi-strategist) | ❌ 금지 |
| 화면 명세 | solution-designer | ❌ 금지 |
| UX/화면 흐름·인터랙션·디자인 시스템 설계 | nora (ux-designer) | ❌ 금지 |
| React/ECharts 화면 실제 구현 | ui-developer | ❌ 금지 |
| 배치 실행 | batch | ❌ 금지 |
| 문서 구조·Why-How 검증 | **Ethan** | ❌ 금지 |
| 스냅샷·push·메모리 | **Sophie** | ❌ 금지 |

**Leo가 해도 되는 것**: 현황 파악, 우선순위 결정, 에이전트 조율, 문서 내용 업데이트 지시(검증은 Ethan), 스냅샷 저장 지시(실행은 Sophie)

---

## 역할 분리 (Leo / Ethan / Sophie)

| 에이전트 | 담당 | 프로젝트 범위 |
|---------|------|-------------|
| **Leo** | biz-health-web PM — 조율·우선순위·계획·현황 관리 | biz-health-web 전용 |
| **Ethan** | 문서 구조·Why-How·설계 기준 3단 구조 검증 | 글로벌 (cwd 기반 프로젝트 판단) |
| **Sophie** | 스냅샷·GitHub push·메모리 관리 (운영 전담) | 글로벌 |

> biz-health-dashboard PM은 David. 두 프로젝트 간 Mart View 연동은 Leo ↔ David 라인으로 조율한다.

---

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 목적 | Impala DB 매출/수주 데이터 → 웹 기반 사업건전성 대시보드 |
| 사용자 | 사업부장, 사업팀장 |
| 기술 스택 | FastAPI (Python) + React + Apache ECharts |
| DB 연결 | impyla (Impala 3.4.0) |
| 작업 범위 | 백엔드 API + 프론트엔드 대시보드 구현 |

---

## 기술 스택 상세

### 백엔드

| 항목 | 내용 |
|------|------|
| 프레임워크 | FastAPI |
| DB 드라이버 | impyla + thrift_sasl |
| 데이터 직렬화 | Pydantic v2 |
| 비동기 | async/await (asyncio) |
| 패키지 관리 | pip + requirements.txt |

### 프론트엔드

| 항목 | 내용 |
|------|------|
| 프레임워크 | React 18 |
| 차트 라이브러리 | Apache ECharts (echarts-for-react) |
| 상태 관리 | React Query (서버 상태) + Zustand (클라이언트 상태) |
| 스타일 | Tailwind CSS |
| 패키지 관리 | npm |

---

## 데이터 규모 (모든 설계의 전제)

- **dm_customer_status**: 10억+ row (파티셔닝 필수, 풀스캔 금지)
- **고객 수**: 15,000+
- **기간**: 과거 3~4년 월별 이력
- **법인**: 본사 1 + 해외 법인 5 = 6개
- **상품**: 30~40개 represent_name
- **통화**: 멀티 통화, 환율 변환 필수

---

## 6개 차원 원칙 (모든 API/컴포넌트의 필수 충족 조건)

| # | 차원 | 구현 컬럼 |
|---|------|----------|
| 1 | 상품별 | represent_name |
| 2 | 법인별 (전체 포함) | company ('ALL' = r003=Real 전체 합산) |
| 3 | 매출유형별 | metric_type + metric_value |
| 4 | 산출주기별 | period_type (monthly/quarterly/half_yearly/yearly/ytm) |
| 5 | 비교축별 | status_vs_xxx 7개 컬럼 |
| 6 | 고객상태별 | New/Expansion/NoChange/Contraction/Churn |

산출 지표: cur_amount (매출 금액) + customer_count (고객 수)

---

## 3-Layer 아키텍처

```
[Impala Mart View] mart_revenue / mart_contract / mart_cs_summary / mart_cs_customer_detail / mart_arr_waterfall
  ↓
[FastAPI Backend] routers/ → services/ → db/
  ↓
[React Frontend] pages/ → components/ → charts/
```

- FastAPI는 반드시 Mart View만 조회. Staging/Domain 직접 쿼리 금지.
- 프론트엔드는 FastAPI API만 호출. Impala 직접 연결 금지.

---

## 프로젝트 폴더 구조

```
biz-health-web/
├── CLAUDE.md
├── backend/
│   ├── main.py              # FastAPI 앱 진입점
│   ├── requirements.txt
│   ├── routers/             # API 라우터 (엔드포인트 정의)
│   ├── services/            # 비즈니스 로직
│   └── db/                  # Impala 연결·쿼리 실행
├── frontend/
│   ├── package.json
│   └── src/
│       ├── components/      # 공통 UI 컴포넌트
│       ├── pages/           # 페이지 단위 컴포넌트
│       └── charts/          # ECharts 차트 컴포넌트
└── docs/
    ├── requirements.md
    ├── design.md
    ├── erd.md
    └── snapshots/
```

---

## API 설계 기준

- RESTful 원칙 준수
- 응답 형식: `{ data: [...], meta: { total, page, ... } }`
- 에러 형식: `{ error: { code, message } }`
- 공통 쿼리 파라미터: `company`, `represent_name`, `period_type`, `ym_from`, `ym_to`
- 대용량 데이터: 페이지네이션 또는 집계 후 반환 (raw row 전체 반환 금지)

---

## 기술 확정 사항 문서 반영 규칙

API/코드 작업 중 확정된 기술 사항은 대화로만 끝내지 말고 즉시 아래 문서에 반영한다.

| 문서 | 반영 대상 |
|------|---------|
| requirements.md | 요구사항 수준의 확정 사항 |
| design.md | 구현 방식/로직 확정 사항 |
| snapshot | 세션 재개 시 컨텍스트 복원에 필요한 기술 결정 사항 |

특히 API 응답 구조, 쿼리 파라미터 규칙, 에러 처리 방식 등 비자명한 기술 결정은 반드시 문서화한다.

---

## 볼륨 대응전략 사전 고려

`dm_customer_status` 관련 API 개발 시 아래 3가지를 착수 전 체크리스트로 반드시 제시한다.

1. **ym 파티션 필터 강제** — API 파라미터에 ym 범위 필수값으로 지정, 미입력 시 에러 반환
2. **집계 레벨 응답** — row 단위가 아닌 집계된 결과만 반환 (mart 뷰 활용)
3. **쿼리 타임아웃** — Impala 쿼리 타임아웃 설정 (기본 30초), 초과 시 에러 처리

---

## 와이어프레임 인터랙티브 기준

신규 화면 제작 시 아래 5가지 핵심 원칙을 반드시 적용한다. (상세 25개 체크리스트는 `ux-designer.md` 참조)

1. **필터 즉시 반영** — 버튼 없이 `change` 이벤트로 즉시 업데이트 (예외: 텍스트 입력·다중선택·고레이턴시)
2. **툴팁** — 모든 차트 hover 시 표시, 진입 150ms 딜레이, 화면 경계 자동 반전
3. **애니메이션** — count-up 400ms / 차트 transition 300ms / `prefers-reduced-motion` 대응 필수
4. **로딩/에러** — 스켈레톤 UI(200ms 초과 시) + NULL "—" 처리 + 에러 유형별 대응
5. **구현 기술** — ECharts + React

---

## 피드백 루프 규칙

### Test/Fix/Repeat 루프 종료 조건

| 조건 | 에스컬레이션 대상 |
|------|----------------|
| 동일 오류 3회 반복 (validator 기준) | Leo(pm) — 의무 에스컬레이션 |
| reviewer 동일 지적 2회 반복 | Leo — developer 가이드라인 업데이트 요청 |
| API 수정만으로 해결 불가한 구조 문제 | architect |
| KPI/산출 로직 근본 불명확 | kpi-strategist |

---

## 세션 종료 루틴 (Sophie 전담)

> 전체 루틴(Ethan 검증 → git push → npm publish)은 **글로벌 CLAUDE.md §7** 참조.  
> 아래는 biz-health-web 전용 스냅샷 작성 단계만 기술한다.

```
# biz-health-web 전용 — 스냅샷 작성
docs/snapshots/snapshot_vXXX.md 신규 작성
docs/snapshots/snapshot_latest.md 갱신
# 이후 글로벌 §8 루틴 실행 (git push + npm publish)
```

---

## 주요 문서 위치

| 문서 | 경로 |
|------|------|
| 요구사항 정의서 | docs/requirements.md |
| 구현 설계서 | docs/design.md |
| ERD | docs/erd.md |
| 최신 스냅샷 | docs/snapshots/snapshot_latest.md |

---
> Source: [bennhee-ai/biz-health-web](https://github.com/bennhee-ai/biz-health-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
