## eco

> 요청하신 대로, PoC 완료, 1차 롤아웃(20명), Windows 환경, 3명 관리자 체제, 향후 보안팀 리뷰 계획 등 현재까지 논의된 모든 전문가(아키텍트, SW, 운영, 보안, 사용자, 관리자) 관점의 조언과 현황을 종합하여 PRD 문서를 최신 상태로 업데이트했습니다.

요청하신 대로, PoC 완료, 1차 롤아웃(20명), Windows 환경, 3명 관리자 체제, 향후 보안팀 리뷰 계획 등 현재까지 논의된 모든 전문가(아키텍트, SW, 운영, 보안, 사용자, 관리자) 관점의 조언과 현황을 종합하여 PRD 문서를 최신 상태로 업데이트했습니다.
FMW 제품 요구사항 문서 (PRD)
 * 문서 기준일: 2025년 10월 31일
 * 문서 상태: PoC 완료. 1차 롤아웃(내부 팀 20명) 준비 완료.
1. 🎯 개요 (Overview)
1.1 제품 미션
FMW의 핵심 미션은 분산된 정책, 피처, 운영 상태 정보를 내부 사용자에게 빠르고 안정적으로 제공하는 것입니다. 이를 통해 데이터 기반의 신속한 의사결정을 지원하고 운영 효율(탐색 비용 제거)을 극대화합니다.
1.2 핵심 솔루션 전략
 * 스냅샷 우선 (Snapshot First): UI(Streamlit)는 사전에 생성된 스냅샷 파일을 우선 로드하여 빠른 조회 성능과 장애 복원력을 보장합니다.
 * 보안 2중 경계 (Dual-Layer Security): Nginx 게이트웨이에서 IP Allowlist로 1차 필터링하고, 내부 서비스는 루프백(127.0.0.1) 바인딩으로 외부 직접 접근을 원천 차단합니다.
 * 멱등성 및 정규화 (Idempotency & Normalization): identity_hash UNIQUE 키를 사용하여 데이터 동기화 시 중복을 방지하고 멱등성을 보장합니다. name_map(Taxonomy)을 통해 비표준 용어를 정규화합니다.
1.3 핵심 파이프라인
데이터 흐름은 다음 4단계로 요약됩니다.
Sync → Snapshot(features) → History(changes) → Overview(daily_counts)
2. 📈 1차 롤아웃 성공 지표 (KPIs)
1개월간의 1차 롤아웃(내부 팀 20명)의 성공은 다음 지표로 측정합니다.
 * 사용자 채택 (1차 사용자 관점): 주간 활성(WAU) \ge 70%
 * 응답 지연 (1차 사용자 관점):
   * 홈(Home) 화면 평균 로딩 < 1초
   * 주요 조회 P95 < 5초
   * /api/v1/* API P95 < 3초
 * 신선도/신뢰성: 일일 데이터 동기화 성공률 \ge 98%
 * 운영 효율 (관리자 관점): 관리자(3명)의 일상 점검 시간 \le 30분/일 (평균)
 * 안정성 (운영 관점): 1개월간 Sev-2 이상 장애(서비스 중단) \le 1건
3. 👥 단계별 사용자 및 요구 (JTBD)
 * Phase 1 (1차 롤아웃): 1차 사용자 (팀 동료 20명)
   * JTBD: "내가 개발 중인 피처/정책의 현재 상태를 즉시 확인하고 싶다."
   * 요구: 압도적인 속도와 편의성.
 * Phase 2 (2차 확대): 2차 사용자 (그룹 사용자)
   * JTBD: "보안이 검증된 공식 툴에서, 내 역할에 맞는 데이터만 보고 싶다."
   * 요구: SSO 인증, RBAC(권한 분리), 안정성.
 * Admin/Ops (관리자 3명 / 고과권자)
   * JTBD: "안정적으로 서비스를 운영하며, 최소 리소스 투입으로 1차 사용자의 KPI(WAU 70%+)를 달성하여 프로젝트의 ROI를 증명하고 싶다."
   * 요구: 안정적 운영(자동화), 운영 효율(운영 부담 \le 30분/일).
4. 📦 범위 (Scope)
4.1 1차 롤아웃 포함 범위 (In-Scope)
 * 스냅샷 우선의 읽기 전용 조회 기능 (Streamlit).
 * API를 통한 데이터 제공 (기본 CSV, 옵션 JSON).
 * 운영 요약 대시보드 및 CSV Export 기능.
 * IP Allowlist 및 루프백 바인딩을 통한 사내망 보안 적용.
 * run_all.py 및 Windows 배치 스크립트를 통한 운영 자동화.
4.2 제외 범위 (Out-of-Scope)
 * UI를 통한 데이터 직접 쓰기/수정(CUD) 기능.
 * SSO / RBAC (Phase 2 보안팀 리뷰 후 로드맵 대상).
 * 외부망 공개 및 멀티테넌시.
5. 🛠️ 아키텍처 및 1차 롤아웃 제약
5.1 1차 롤아웃(Phase 1) 핵심 제약 (Constraints)
PoC 완료 후, 20명 사용자 대상의 안정적인 1차 롤아웃을 위해 다음 제약 조건을 반드시 준수해야 합니다.
 * 플랫폼: 모든 서버 및 1차 사용자 클라이언트는 Windows OS입니다.
 * 데이터베이스 (아키텍트 조언): 20명 동시 접속을 위해 SQLite 사용을 금지하며, 반드시 Postgres (또는 동급의 서버형 DB)를 사용해야 합니다.
 * 프로세스 (SW/운영 조언): cmd 수동 실행을 금지합니다. NSSM 등을 사용하여 Django/Streamlit 프로세스를 Windows 정식 서비스로 등록해야 합니다. (자동 시작 및 자동 재시작 보장)
5.2 C4 아키텍처 (1차 롤아웃 기준)
 * Nginx (Gateway):
   * 보안 정책: IP Allowlist 기반 접근 제어.
   * location /streamlit/* (1차 사용자 20명 IP 허용) \rightarrow 127.0.0.1:8501 (Streamlit)
   * location / (관리자 3명 IP 허용) \rightarrow 127.0.0.1:8000 (Django)
 * Django (API/OPS): 127.0.0.1:8000 루프백 바인딩. (관리자 3명만 Nginx를 통해 접근)
 * Streamlit (Viewer): 127.0.0.1:8501 루프백 바인딩. (사용자 20명만 Nginx를 통해 접근)
 * Database (DB): Postgres (20명 동시 접속 대응).
6. 💾 데이터 모델 (Data Model)
(v3 템플릿 기준)
 * features: (구 FeatureRecord) 기능 상태의 정규화된 현재 원장 (identity_hash UNIQUE)
 * changes: (구 AuditLog) C/U/D 변경 이력 원장 (보안팀 리뷰 대비)
 * runs: 동기화(Sync) 실행 단위 원장 (운영 모니터링용)
 * daily_counts: 오버뷰 화면을 위한 일별 집계 테이블 (UI 성능 최적화)
 * name_map: (구 TaxonomyMap) 별칭 ↔ 표준명 매핑 테이블
7. 🔌 API 설계 (1차 롤아웃 기준)
 * /api/v1 (공개 읽기 전용): (Nginx: 사용자 20명 + 관리자 3명 IP 허용)
   * GET /api/v1/all: 필터링 가능한 전체 features 스냅샷 (CSV 기본).
   * GET /api/v1/{feature_group}: 그룹별 features 스냅샷.
 * /api/dev (운영 제어): (Nginx: 관리자 3명 IP만 허용)
   * POST /api/dev/sync/run: ETL 파이프라인 비동기 트리거.
   * GET /api/dev/runs/summary: daily_counts 기반 오버뷰 집계 (UI 바인딩용).
   * GET /api/dev/changes: changes 이력 조회 (디버깅 및 감사용).
8. 💎 비기능 요구사항 (NFRs) - 1차 롤아웃
8.1 성능 (Performance)
 * NFR 8.1.1 (UI 로딩): 홈(Home) 화면 평균 응답 시간 < 1초.
 * NFR 8.1.2 (UI 조회): 주요 조회 화면(Overview, Detail) P95 응답 시간 < 5초.
 * 전술: 스냅샷 우선 로딩, daily_counts 집계 테이블 활용.
8.2 보안 (Security) - Phase 1
 * NFR 8.2.1 (접근 통제): Nginx에서 IP Allowlist를 통한 접근 제어 (비허용 IP 100% 차단).
 * NFR 8.2.2 (권한 분리): 사용자(20명) IP가 관리자용 /api/dev 경로에 접근 불가해야 함.
 * NFR 8.2.3 (네트워크 경계): Django/Streamlit 루프백(127.0.0.1) 바인딩.
8.3 안정성 및 운영성 (Reliability & Operability)
 * NFR 8.3.1 (프로세스): **Windows 서비스 등록(NSSM 등)**으로 자동 시작 및 자동 재시작 보장 (SW/운영 전문가 조언).
 * NFR 8.3.2 (파일 잠금 방지): 스냅샷 파일 교체는 원자적(Atomic) 교체로 구현 (아키텍트 조언).
 * NFR 8.3.3 (데이터 복구): Windows 작업 스케줄러를 통한 일일 DB(pg_dump) 및 스냅샷 자동화 백업 (운영 전문가 조언).
 * NFR 8.3.4 (배포): deploy.bat 등 배포 스크립트를 통한 표준화된 배포 (SW 전문가 조언).
9. 🗺️ 롤아웃 계획 (Roadmap)
 * Phase 1 (1차 롤아웃 / 1개월): "가치 증명 (Prove Value)"
   * Action: 20명 팀원 대상 서비스 런칭. (NFR 8.1, 8.2, 8.3 준수)
   * Goals: KPI 달성 (WAU \ge 70%, 성능 NFR 만족, 운영 안정성 \le 30분/일).
   * Output: 1개월간의 성공적인 KPI 리포트 및 사용자 피드백 확보.
 * Phase 2 (보안팀 리뷰 / 2차 확대): "보안 및 규모 증명 (Prove Scale)"
   * Action: Phase 1 리포트를 기반으로 보안팀 공식 리뷰 및 협의 시작.
   * Goals: 보안팀 요구사항 적용 (SSO, RBAC, HTTPS 등). 2차 사용자(그룹)로 확대.
   * Architecture (SW 전문가 조언): 2차 확대(예: 200명+) 시점의 Streamlit 메모리 한계를 인지하고, API + Web(React/Vue) 아키텍처로의 전환을 검토.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haapy-mind-life)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/haapy-mind-life)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
