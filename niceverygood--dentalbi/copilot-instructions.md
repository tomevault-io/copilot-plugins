## dentalbi

> 치과 전자차트(MSSQL) 데이터를 AI로 분석하는 B2B SaaS 경영 대시보드.

# DentalBI - Antigravity Agent 설정

## 프로젝트 개요
치과 전자차트(MSSQL) 데이터를 AI로 분석하는 B2B SaaS 경영 대시보드.
서울대 치과경영정보학교실 최형길 교수 강의(2023.12) 기반 KPI 및 SQL 쿼리 설계.

## 기술 스택
- **Frontend**: Next.js 14 + TypeScript + TailwindCSS + Recharts
- **Backend**: Python FastAPI + pymssql + SQLAlchemy
- **Database**: PostgreSQL (SaaS 메타), Redis (캐시), MSSQL (치과 전자차트 읽기전용)
- **AI**: Claude API (Sonnet) — 경영 인사이트 자동 생성
- **배포**: Docker Compose, AWS ECS Fargate

## 코드 컨벤션
- 모든 코드에 한글 주석 필수
- Python: PEP 8, type hints 필수, async/await 사용
- TypeScript: strict mode, 컴포넌트는 함수형 + hooks
- SQL: 모든 치과 DB 쿼리에 WITH (NOLOCK) 필수 (진료 중 DB 부하 방지)
- 환자 개인정보(PII)는 해시 처리 후 전송, 원본 절대 클라우드 저장 금지
- 컴포넌트 파일명: PascalCase, API 라우트: kebab-case

## 핵심 도메인 용어
- 신환: 새로 내원한 환자 (pnt_newold = 'N')
- 구환: 재내원 환자 (pnt_newold = 'O')
- 총수익: 면세수납 + 부가세수납 + 공단부담금
- 공단부담금: 건강보험공단에서 지급하는 보험진료 비용 (corporation_fee)
- TxMix: 진료내역 구성비 (임플란트/크라운/발치/근관치료 등)
- Turn-away Patient: 마지막 진료 후 예약 없이 돌아선 환자
- Lost Patient: 진료 진행 중 미내원으로 잃어버린 환자
- 체어가동률: 진료 체어 시간 점유율

## 전자차트 DB 테이블 (MSSQL)
- tb_staff_info: 의사/직원 정보 (staff_id, staff_name, position_div, religion=진료층)
- tb_accept: 접수/진료 데이터 (doct_id, pnt_id, pnt_newold, acpt_dat)
- tb_account_book_settlement_info: 수납 정산 (patient_id, doctor_id, card_sum_fee, cash_sum_fee...)
- tb_account_book: 진료비 (patient_id, account_book_date, corporation_fee, sequence)
- tb_patient_info: 환자 기본정보 (pnt_id, registration_date, pnt_name)

## 현재 파일 구조
```
dentalbi/
├── frontend/
│   ├── src/app/              # Next.js App Router
│   ├── src/components/       # React 컴포넌트
│   └── DentalBI.jsx          # 현재 프로토타입 (여기서 분리 필요)
├── backend/
│   ├── main.py               # FastAPI 서버 (현재 단일 파일)
│   ├── init.sql              # PostgreSQL 스키마
│   └── .env.example          # 환경변수 템플릿
├── docker-compose.yml
├── AGENTS.md                 # 이 파일
└── README.md
```

## Agent 작업 시 주의사항
1. 치과 DB 스키마는 전자차트 종류(하나로/덴트웹/원클릭)마다 다름 — adapter 패턴 사용
2. 환자 이름/주민번호 등 PII는 절대 로그에 남기지 않을 것
3. MSSQL 쿼리는 반드시 읽기 전용 (SELECT만, INSERT/UPDATE/DELETE 금지)
4. 프론트엔드 차트 데이터는 만원 단위로 표시 (DB 원본은 원 단위)
5. 한국 시간대(KST, UTC+9) 기준으로 날짜 처리

---
> Source: [niceverygood/DentalBi](https://github.com/niceverygood/DentalBi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
