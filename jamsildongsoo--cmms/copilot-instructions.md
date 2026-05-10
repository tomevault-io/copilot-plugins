## cmms

> 요구사항·구조·스키마는 `docs/` 참조 — 이 파일은 **CMMS 고유 규칙과 gotcha**만 다룹니다.

# CMMS Coding Guide

## 문서 참조
요구사항·구조·스키마는 `docs/` 참조 — 이 파일은 **CMMS 고유 규칙과 gotcha**만 다룹니다.
- `docs/PRD.md` — 제품 요구사항, 멀티테넌시, ID 채번, RBAC 역할, 상태 코드
- `docs/ARCHITECTURE.md` — 시스템 구조, 설계 패턴, 보안, DB 규칙, 인프라
- `docs/DB_SCHEMA.md` — 테이블 스키마, ERD, 참조 관계
- `docs/API_SPEC.md` — 전 엔드포인트 사양, Request/Response
- `docs/TODO.md` — 개선 TODO, 보류 항목
- `docs/MOBILE.md` — 모바일 전용 화면 구성안, 라우팅, UI 설계

## CMMS 도메인 필드명
| 용도 | DB | 코드 |
|------|-----|------|
| 멀티테넌시 | `company_id` | `companyId` |
| 플랜트 | `plant_id` | `plantId` |
| 부서/담당 | `dept_id`, `person_id` | `deptId`, `personId` |
| 설비 연결 | `equipment_id` | `equipmentId` |
| 다형성 참조 | `ref_entity`, `ref_id` | `refEntity`, `refId` |
| 결재 연결 | `approval_id` | `approvalId` |
| 비용/공수 | `cost`, `time` | `cost`, `time` |

## CMMS Backend 규칙
- `@Transient` 필드(equipmentName, personName, deptName) → Service에서 `loadNameMaps()` batch enrichment (N+1 금지)
- 목록 API는 `Pageable` 파라미터 지원
- 상세 조회(getById)에 `@EntityGraph` 적용하여 하위 items 즉시 로딩
- 파일 삭제: 물리 파일 trash 이동 (`uploads/trash/{companyId}/`) + file_item DB 삭제
- 파일 업로드: SHA-256 체크섬 자동 저장
- 결재 원문서 연동: 직접 호출 (Event 패턴 미사용, 동일 트랜잭션 원자성 보장)
- ID 채번 (마스터): `SystemService.generateId(companyId, refEntity, "GLOBAL")` → `EQ000001`
- ID 채번 (트랜잭션): `SystemService.generateDeptId(companyId, refEntity)` → `AP-MNT-26-000001` (PREFIX-부서-YY-SEQ, 현재 사용자 부서 자동 조회)
- 점검(Inspection) 코드 규칙: ID prefix=`PM`, refEntity(결재)=`PM`, 코드유형=`PM_TYPE` — 테이블명/클래스명/API 경로는 `inspection` 유지

## CMMS Frontend 규칙
- Submit: `type="button"` + `onClick={() => onSave('A')}` + `getValues()` — `type="submit"` + state 방식 금지 (React 비동기 배치 이슈)
- Select: `SearchableSelect`로 통일 (Radix Select는 disabled 전달 문제)
- 실적 입력: refId/refEntity 있으면 결과/비고만 편집 (`isRefLocked` 패턴)
- 인쇄: 웹 콘텐츠 `print:hidden`, Print 컴포넌트 `hidden print:block`
- 출력 양식: 헤더 레이아웃 통일, `equipmentId / equipmentName` 형식
- 직급/직책 → "직급/직책" 필드(position)로 병합 (DB title 컬럼 유지)
- `STATUS`(`constants/status.ts`) — 상태 비교에 상수 필수 사용 (리터럴 `'T'`,`'C'` 직접 비교 금지)
- `DECISION_TYPE`, `STEP_RESULT`, `MANAGER_ROLES` — 결재/역할 상수 필수 사용
- 트랜잭션 목록은 `PageResponse<T>` + `Pagination` 컴포넌트, 마스터 목록은 전체 로딩

## 구현 시 주의사항 (프로젝트 gotcha)

### Equipment
- Equipment ↔ EquipmentItem: `@OneToMany CascadeAll, orphanRemoval=true`
- 점검 생성 시 설비 선택하면 `equipment.items → inspection.items` 자동 복사

### Work Permit
- template id `com` → DB 컬럼 `checksheet_json_com` (하위호환), `toJsonKey()` 헬퍼로 변환
- 복수 연장 시 `extendedEndDt` 덮어쓰기 (최종 연장만 유효, 원래 endDt 보존)
- 라벨: "신청부서"→"관리부서", "신청자"→"담당자", "신청일자"→"허가일자"
- 완료/연장: MANAGER 이상만 (작성자 권한 삭제)
- 설비 선택: 직접 등록 시 `status='C'`인 모든 설비 허용 (`workpermitYn` 필터 없음) — WO 목록에서만 `workpermitYn='Y'` 설비에 허가 버튼 노출

### Work Order
- 고장 코드: `failPart`, `failMode`, `failCause` — code_item 마스터 연동 (FAIL_PART, FAIL_MODE, FAIL_CAUSE)
- `codeItem === 'BM'`(고장보전)일 때만 고장 분석 카드 표시 (등록/상세 모두)
- codeItem은 code_item 마스터의 `itemId`를 저장 — 하드코딩 금지, API 조회(`getCodeItems`)로 Select 구성
- 비용/공수: header `cost`/`time`만 사용, `vendor`는 DB 유지·UI 제거 (향후 item 레벨 INT/EXT 확장 시 `docs/TODO.md` F-3 참조)

### Approval
- 결재본문: Thymeleaf 템플릿 (`backend/src/main/resources/templates/approval/`) + 직접 기안(RichTextEditor)
- 결재 제목: `[점검 계획/실적]`, `[작업지시 계획/실적]`, `[안전작업 허가]` + name
- 결재본문은 HTML 유지 (직접 기안 RichTextEditor와 호환 필요, JSONB 전환 부적절)
- layout.html에 모바일 반응형 CSS 적용 (`@media max-width:600px`)
- 결재선: 결재/합의는 메인 영역, 참조는 하단 별도 영역 (`ApprovalLineModal` 공통 컴포넌트)
- 참조함: 결재 완결(status=CONFIRMED) 후에만 노출
- 원문서 보기: 모달(Dialog)로 표시 (페이지 이동 X)

### Inventory
- 이동평균법: 창고 단가 = `SUM(amount)/SUM(qty)` per (companyId, storageId, inventoryId)
- MOVE 양방향: MOVE_OUT.refId=moveInHistoryId, MOVE_IN.refId=moveOutHistoryId (처리 전 미리 채번)
- 소급 입력 불가 (txDate 제거, createdAt만 사용)

## Dev Quick Reference
- `.env` 파일로 자격증명 관리 → `${DB_URL}`, `${JWT_SECRET}` 등 (상세: `docs/ARCHITECTURE.md` §6)
- **부트스트랩 계정**: 회사코드 `0000` / ID `system` / 비밀번호 `12345678` (최초 로그인 후 변경 권장)
- data.sql 3섹션 구조: ① 부트스트랩(0000/system, 삭제금지) ② 공통코드(회사별 복사) ③ 테스트데이터(운영 시 제거)
- Frontend dev: port 5173, `/api` → proxy → localhost:8080
- Password: 최소 8자, backend `validatePassword()` + frontend `minLength:8`

---
> Source: [jamsildongsoo/cmms](https://github.com/jamsildongsoo/cmms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
