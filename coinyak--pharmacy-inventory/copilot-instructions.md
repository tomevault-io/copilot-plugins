## pharmacy-inventory

> 병원 약제부 항암제/일반약 재고관리 프로그램.

# 재고관리 시스템

## 프로젝트 개요
병원 약제부 항암제/일반약 재고관리 프로그램.
순수 HTML/CSS/JS + SheetJS(CDN). Cloudflare Pages + D1으로 호스팅.
관리자 2명(항암제/일반약 각 1명) + 읽기 전용 사용자(약제부 동료).

## 대상 파일
- `항암제_재고관리.html` (원본, 로컬 백업)
- `public/index.html` (배포용, Cloudflare Pages)

## 탭 구성
1. 대시보드 - 파일 업로드(4개 xlsx), 통계, 알림
2. 재고현황 - 항암제+일반약 필터링, 컬럼 리사이즈, 냉장약 토글, 숨기기, 엑셀 내보내기
3. 일일사용량/잉여 - 자동 잉여계산(vial sharing), 수동 잉여 입력
4. 수요예측 - PHS 월별사용량 기반 2주 예측, 발주 엑셀 출력
5. 제로관리약 - 환자별 스케줄, 달력 뷰, Day별 투여 체크, 일정 이동
6. 설정 - 백업/복원, 약품 마스터 목록(항암제/일반약 분류), 사용량 이력

## 핵심 state 구조
- `inventory` - 현재고(1.xlsx)
- `dailyUsage` - 당일 사용량(2.xlsx, 메모리만, 비저장)
- `inpatientMix` - 입원MIX(3.xlsx)
- `outpatientData` - 외래집계(4.xlsx)
- `drugMaster` - 항암제 마스터 목록
- `generalDrugMaster` - 일반약 마스터 목록
- `zeroDrugs` - 제로관리약 스케줄
- `manualSurplus` - 수동 잉여
- `coldStorage` - 냉장약 코드 목록
- `dailyData` - 날짜별 스냅샷(최근 7일)

## 코딩 규칙
- 이모티콘 사용 금지. 텍스트 또는 SVG 아이콘만 사용
- 테이블 셀은 타이트하게 (th 4px, td 3px padding, line-height 1.3)
- 마스터 목록(항암제+일반약) 필터링은 재고현황, 사용량/잉여 탭 모두 동일 적용
- 사용량(dailyUsage)은 localStorage에 저장하지 않음 (당일만 메모리 유지)

## Cloudflare 배포
- **Pages**: `public/` 디렉토리 호스팅
- **D1**: `pharmacy-db` (SQLite) - app_state, admin_config 테이블
- **Pages Functions**: `functions/api/` - auth.js, state.js, _middleware.js

### 배포 명령
```bash
export PATH="/c/Program Files/nodejs:/c/Users/duih/AppData/Roaming/npm:$PATH"
wrangler pages dev public          # 로컬 테스트
wrangler pages deploy public       # 실제 배포
wrangler d1 execute pharmacy-db --file=schema.sql  # DB 초기화
```

### API 엔드포인트
- `GET /api/state?type=chemo|general` - 데이터 조회 (인증 불필요)
- `PUT /api/state` - 데이터 저장 (관리자 토큰 필요)
- `POST /api/auth` - 관리자 로그인 (최초 접속 시 비밀번호 자동 설정)

### 인증 흐름
- 관리자: 비밀번호 → SHA-256 해시 → `type:hash` 토큰으로 API 인증
- 읽기 전용: 인증 없이 GET만 허용, UI에서 수정 요소 숨김 (`body.viewer-mode`)
- 세션: sessionStorage에 토큰 저장 (브라우저 탭 닫으면 만료)

### D1 스키마
- `app_state(id, data, daily_data, updated_at)` - state JSON 저장
- `admin_config(id, password_hash)` - 관리자 비밀번호 해시

## 리팩토링 플랜
`.Codex/plans/witty-wondering-clarke.md` 참조
`.Codex/plans/jolly-marinating-meteor.md` - Cloudflare 배포 플랜

## 남은 작업
- REQ 7: 공급뉴스 공공데이터 API 연동 → **완료** (4종 API 연동, DATA_GO_KR_API_KEY 설정 완료)
  - `.Codex/plans/news-openapi-upgrade.md` 참조
- REQ 8: 뉴스 원내/원외 매칭 → `.Codex/plans/news-drug-matching.md` 참조
  - 뉴스 항목에 "원내" 배지 표시 (drugMaster/generalDrugMaster와 약품명 매칭)
  - 3단계 매칭: EDI코드 > 약품명 포함 > 성분명 역방향
  - index.html만 수정 (news.js 변경 불필요)
  - 구현 전 drugMaster 실데이터 확인 필요 (콘솔: state.drugMaster.map(d=>d.name))

---
> Source: [Coinyak/pharmacy-inventory](https://github.com/Coinyak/pharmacy-inventory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
