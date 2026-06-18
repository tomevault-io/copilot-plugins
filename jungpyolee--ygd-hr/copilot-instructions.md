## ygd-hr

> > 연경당 통합 근태 관리 시스템 (위치 기반 출퇴근 + 관리자 모니터링 PWA)

# CLAUDE.md — YGD HR 프로젝트 지침

> 연경당 통합 근태 관리 시스템 (위치 기반 출퇴근 + 관리자 모니터링 PWA)

---

## 1. 프로젝트 개요

| 항목 | 값 |
|------|-----|
| **GitHub** | `jungpyolee/ygd_hr` |
| **Git 계정** | `jungpyolee` / `jungpyo5789@gmail.com` |
| **Supabase Production** | `ymvdjxzkjodasctktunh` / `https://ymvdjxzkjodasctktunh.supabase.co` |
| **Supabase Dev** | `rddplpiwvmclreeblkmi` / `https://rddplpiwvmclreeblkmi.supabase.co` |
| **배포** | Vercel |
| **DB 연결** | Supabase Management API (psql 불가 — IPv6 전용) |

---

## 2. 기술 스택

- **Framework**: Next.js (App Router) + React 19 + TypeScript
- **Auth & DB**: Supabase (SSR 쿠키 기반 세션)
- **Styling**: Tailwind CSS v4 + shadcn/ui + Pretendard 폰트
- **PWA**: Serwist (개발 모드 비활성)
- **Toast**: sonner
- **날짜**: date-fns
- **테스트**: vitest
- **아바타**: react-nice-avatar
- **푸시**: web-push (VAPID)
- **빌드**: `npm run build` (webpack 강제, package.json에 설정됨)

---

## 3. 주요 명령어

```bash
npm run dev        # 개발 서버 (webpack 강제)
npm run build      # 빌드 검증 (코드 수정 후 항상 실행, webpack 강제)
npm run lint       # ESLint
npm run test       # vitest 단위 테스트
npm run test:db    # DB RLS 통합 테스트
```

---

## 3-1. dev → main 배포 절차 (반드시 준수)

> ⚠️ 이 절차를 건너뛰면 Production DB와 코드 간 불일치가 발생할 수 있다.

```
[STEP 1] 코드 준비
  - dev 브랜치 변경사항 모두 커밋 + push
  - npm run build 빌드 통과 확인

[STEP 2] 신규 마이그레이션 파악
  - docs/migrations/ 에서 마지막 prod 배포 이후 추가된 NNN_*.sql 파일 목록 확인
  - 각 마이그레이션의 Production 적용 여부 확인

[STEP 3] Production DB 마이그레이션 실행
  - source .env.local 로 토큰 로드
  - 신규 마이그레이션을 번호 순서대로 Production에 실행
  - 각 실행 후 검증 쿼리로 적용 확인

[STEP 4] dev → main 머지 & push
  - git checkout main && git pull origin main
  - git merge dev --no-edit
  - git push origin main

[STEP 5] Dev / Production 싱크 검증
  1. 테이블 목록 일치 확인
  2. 핵심 테이블 컬럼 일치 확인
  3. RLS 정책 전체 목록 일치 확인
  4. Realtime publication 테이블 확인
  5. 트리거 목록 일치 확인 (auth 스키마 포함)
  6. 데이터 무결성 확인 (필요 시)
```

### Production 마이그레이션 실행 명령어

```bash
source .env.local

curl -s -X POST "https://api.supabase.com/v1/projects/ymvdjxzkjodasctktunh/database/query" \
  -H "Authorization: Bearer $SUPABASE_PROD_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "SQL HERE"}'
```

> Production 마이그레이션은 Claude Code가 직접 실행한다.

---

## 4. DB 작업 방식

psql 직접 연결 불가. **Supabase Management API**로 SQL 실행.

> ⚠️ **일반 개발 작업은 반드시 Dev DB에서만 실행. Production은 섹션 3-1 배포 절차에서만 Claude Code가 직접 실행한다.**

```bash
source .env.local

# ✅ Dev (기본 — 항상 이걸 사용)
curl -s -X POST "https://api.supabase.com/v1/projects/rddplpiwvmclreeblkmi/database/query" \
  -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "SQL HERE"}'
```

### DB 스키마 변경 플로우

```
1. docs/db-issues/NNN-제목.md 작성 (배경, 원인, 계획)
2. docs/migrations/NNN_설명.sql 작성
3. Dev DB에 SQL 실행 (rddplpiwvmclreeblkmi)
4. Dev에서 검증 SQL 실행
5. docs/db-issues/NNN-제목.md 결과 기록
6. docs/schema.md 갱신
7. Production 반영은 배포 시 섹션 3-1 절차에 따라 Claude Code가 직접 실행
```

> ⚠️ Storage 버킷 신규 생성 시 RLS 정책 4개(INSERT/UPDATE/DELETE/SELECT) 필수 → `docs/db-management-plan.md` 참조

---

## 5. 코드 이슈 작업 플로우

```
1. docs/issues/NNN-제목.md 작성 (배경, 원인 분석, 수정 내용, 결과)
2. 코드 수정
3. npm run build 로 빌드 확인
4. docs/issues/NNN-제목.md 결과 기록
```

DB 이슈는 `docs/db-issues/NNN-제목.md`에 동일한 형식으로 작성.

---

## 5-1. 이용가이드 버전 업데이트 플로우 (반드시 준수)

이용가이드 버전을 올릴 때 **세 파일을 반드시 동시에** 수정한다.
하나라도 빠지면 레드닷이 잘못 동작한다.

| 파일 | 수정 내용 |
|------|-----------|
| `src/app/guide/page.tsx` | `CURRENT_VERSION` 상수를 새 버전으로 변경 |
| `src/app/my/page.tsx` | `seen !== "vX.X.X"` 비교 문자열을 새 버전으로 변경 |
| `src/components/BottomNav.tsx` | `GUIDE_VERSION` 상수를 새 버전으로 변경 |

---

## 6. UI/UX 규칙 (토스 기준)

전체 가이드: `docs/toss-ui-ux-guidelines.md`

### 핵심 규칙 (절대 위반 금지)

1. **말투**: `~해요` 체 통일. `~합니다/~했습니다` 금지.
2. **다이얼로그**: `alert()`, `confirm()`, `window.prompt()` 절대 금지 → 커스텀 바텀시트/모달 사용
3. **로딩**: "로딩 중..." 텍스트 금지 → Skeleton UI 필수
4. **토스트**: `toast.error("저장 실패")` 처럼 단어만 금지 → 이유 + 해결방법 명시
5. **폰트**: Pretendard 단일 사용
6. **버튼 텍스트**: 동사형 (`"저장하기"`, `"출근하기"`)

### 컬러 토큰

| 토큰 | 값 |
|------|-----|
| primary | `#3182F6` |
| text-primary | `#191F28` |
| text-secondary | `#4E5968` |
| text-tertiary | `#8B95A1` |
| bg-default | `#F2F4F6` |
| border | `#E5E8EB` |
| primary-light | `#E8F3FF` |

---

## 7. 데이터베이스 핵심 사항

- **시간대**: DB timezone `Asia/Seoul` (KST). `timestamptz`는 UTC 저장, 날짜 함수는 KST 기준.
- **JS → DB**: `.toISOString()` (UTC ISO) 전달
- **DB → JS**: `new Date(timestamptz)` 로컬 변환 (KST 자동)
- **중복 출퇴근 방지**: `prevent_duplicate_attendance()` 트리거 — 당일(KST) 같은 타입 연속 + 60초 간격 차단
- **RLS**: 모든 테이블 활성화. `is_admin()` 함수로 관리자 우회.

### 주요 테이블

| 테이블 | 설명 |
|--------|------|
| `profiles` | 사용자 프로필 (auth.users 1:1) |
| `attendance_logs` | 출퇴근 기록 (IN/OUT, attendance_type) |
| `stores` | 매장 정보 (위치, GPS 설정, overtime 설정, QR 토큰) |
| `store_positions` | 근무지별 포지션 목록 |
| `employee_store_assignments` | 직원-근무지 배정 (N:N) |
| `weekly_schedules` | 주차별 스케줄 컨테이너 (draft/confirmed) |
| `schedule_slots` | 개별 근무 슬롯 (날짜·시간·장소·포지션) |
| `substitute_requests` | 대타 요청 (pending→approved/rejected→filled) |
| `overtime_requests` | 추가근무 인정/넘김 (UNIQUE(profile_id, date)) |
| `attendance_adjustments` | 근태 조정 신청 (직원 신청 → 관리자 승인/반려) |
| `payroll_periods` | 급여 정산 기간 |
| `payroll_entries` | 급여 정산 항목 (직원별 상세) |
| `payroll_settings` | 급여 정산 설정 (시급, 주휴 등) |
| `announcements` | 공지사항 (핀/타겟 근무지 필터) |
| `notifications` | 실시간 알림 + 웹 푸시 |
| `recipe_items` | 레시피 (썸네일, 영상, 재료, 단계) |
| `recipe_comments` | 레시피 댓글/대댓글 (소프트 삭제) |

`attendance_logs.attendance_type` 값:
- `regular` / `remote_out` / `business_trip_in` / `business_trip_out`
- `fallback_in` / `fallback_out` (GPS 실패 시 수동 처리)
- `qr_in` / `qr_out` (QR 스캔 출퇴근)

`overtime_requests.status` (ENUM `overtime_status`): `pending` / `approved` / `rejected` / `dismissed`

---

## 8. 보안 주의사항

- `.env.local`에 Supabase ANON Key, Access Token 있음 — 절대 커밋 금지
- Git pre-commit 훅: `jungpyolee` 계정 외 커밋 차단 (`.githooks/pre-commit`)
- Private 버킷 `hr-documents`: 서명된 URL(60초 만료)로만 접근

---

## 9. 참고 문서 (docs/)

| 파일 | 내용 |
|------|------|
| `WORKFLOW.md` | 초기 세팅, 전체 작업 플로우 |
| `project-analysis.md` | 프로젝트 전체 분석 (구조, 흐름, 컴포넌트) |
| `schema.md` | DB 스키마 현재 상태 (항상 최신 유지) |
| `toss-ui-ux-guidelines.md` | UI/UX 개발 기준 전체 |
| `db-management-plan.md` | DB 연결 방식, Storage 버킷 체크리스트 |
| `notification-system.md` | 알림 시스템 설계 (푸시, 인앱, 타입별 라우팅) |
| `analytics-and-error-handling.md` | 에러 로깅 + 모니터링 설정 |
| `ui-ux-analysis.md` | UI/UX 감사 보고서 |
| `performance-improvement-plan.md` | 성능 개선 계획 (N+1 쿼리 등) |
| `kurly-order-roadmap.md` | 컬리 발주 시스템 로드맵 |
| `issues/` | 코드 이슈 기록 (NNN-제목.md) |
| `db-issues/` | DB 이슈 기록 (NNN-제목.md) |
| `migrations/` | 실행된 SQL 파일 이력 |
| `planning/` | SaaS 전환 계획, 로드맵 |

---
> Source: [jungpyolee/ygd_hr](https://github.com/jungpyolee/ygd_hr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
