## kit-admin-system

> | 프로젝트 | KIT 외국인 학생 학사관리시스템 |

# KIT Admin System — 프로젝트 운영 규칙

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 프로젝트 | KIT 외국인 학생 학사관리시스템 |
| 현재 단계 | 2차 개발 |
| 배포 목표 | 2026-04-24 |
| 개발 체제 | 1인 개발 (프론트 + 백엔드) + AI |

## 담당 레포

| 역할 | 경로 |
|------|------|
| 관리자 프론트 (React/FSD) | `../kit-frontend-admin` |
| 관리자 API (Spring Boot/MyBatis) | `../kit-backend-admin` |

> kit-admin-system 자체에는 코드를 쓰지 않는다. 문서/상태 관리만 한다.

## 기획 스펙

**Figma 화면설계서 (Single Source of Truth)**
- URL: `https://www.figma.com/design/8frPKJNGKoRXvrD5RJ7xkH/`

---

## 프로젝트 초기화 절차 (최초 1회)

1. **요구사항 추출** — Figma에서 req 파일을 모듈별로 생성, manifest.json 초기화 (전체 status: "pending"), decisions.md 생성
2. **Architecture 생성** — **모든 req를 분석한 후** architecture_overview 작성. ERD 초안, 화면 IA, API 전체 목록 + 소유권, 공유 타입, 인프라 구성도 포함. req 하나만 보고 만들지 않는다
3. **Contract 순차 생성** — architecture 참조하며 공통(cmn) → 연동 → 개별 모듈 순서로 작성. 의존 대상 contract가 먼저 완료된 후 작성
4. 이후 일반 작업 흐름(구현 → TC → 검증)으로 전환

---

## 작업 시작 전 필수 절차 (매번 수행)

compact 발생 후에도 이 절차를 동일하게 수행하면 상태가 복원된다.

1. `base/manifest.json` 읽기 → 전체 상태 파악
2. `base/decisions.md` 최근 10건 확인 → 최근 결정사항 인지
3. `progress/current.md` 읽기 → 이전 작업 이어받기
4. 현재 작업 대상 req 파일 읽기
5. 해당 req의 contract 파일 읽기 (있는 경우)
6. 프론트엔드 작업 시 → `contract/design-system.md` 읽기 (있는 경우)


## 파일 체계

```
base/manifest.json                                    ← Single Source of Truth
base/decisions.md                                     ← 결정 로그 (append-only)
base/architecture_overview_{yyyyMMdd_HHmm}.md         ← 전체 설계서 (ERD/API/타입)
req/{모듈}_{reqID}_{yyyyMMdd_HHmm}.md                ← 요구사항
contract/{모듈}_{reqID}_{yyyyMMdd_HHmm}.md            ← 개발 규격
contract/design-system.md                             ← 디자인 토큰 + 컴포넌트 명세
tc/{모듈}_{reqID}_tc{번호}_{yyyyMMdd_HHmm}.md        ← 테스트 케이스
progress/current.md                                   ← 현재 진행 상황
```

## 모듈 코드 (2차 개발)

| 코드 | 메뉴 |
|------|------|
| `home` | 홈 (대시보드) |
| `usr` | 사용자관리 확장 |
| `stu` | 학생관리 |
| `cls` | 수업관리 |
| `grd` | 성적관리 |
| `cns` | 상담/평가관리 |
| `doc` | 출력물관리 |
| `brd` | 게시판관리 |
| `sys` | 시스템관리 확장 |
| `reg` | 등록관리 이월 |

---

## 상태 관리 규칙

### 작업 진행 시
- 시작 전: manifest.json status → "in_progress"
- 완료 시: status → "done"
- progress/current.md에 현재 작업 + 다음 할 일 갱신

### 파일 변경 공통 패턴 (req / contract / architecture 모두 동일)
1. 새 타임스탬프로 파일 생성
2. manifest.json 해당 경로 업데이트 + 이전 파일을 history에 추가
3. decisions.md에 변경 사유 + 영향 범위 append
4. **영향받는 하위 문서 정합성 확인** (architecture → contract → TC → 구현 코드)

### TC 작성 규칙
- contract 완료 후 tc/ 폴더에 작성. 파일명: `tc/{모듈}_{reqID}_tc{번호}_{yyyyMMdd_HHmm}.md`
- 하나의 contract에 TC 여러 개 가능. manifest.json `tc` 배열에 경로 추가
- TC 상태: `PENDING` → `PASS` / `FAIL`
- TC 실패 시 decisions.md에 기록 → 수정 → 재검증
- TC 실행 중 발견사항: req 누락 → req+contract+TC 수정 / TC 부족 → TC 추가 / 버그 → 구현 수정


### decisions.md 기록 형식
```
## {yyyy-MM-dd HH:mm} — {제목}
- 유형: 🧑 사람 결정 / ✅ 자체 결정 / 🔄 역제안 / 🔍 TC 실행 중 발견
- 내용: {무엇을 결정했는지}
- 사유: {왜 이렇게 결정했는지}
- 영향: {어떤 req/contract/architecture에 영향이 있는지}
```

---

## Architecture 갱신 트리거

아래 중 하나 발생 시 architecture_overview를 갱신한다:
- 테이블 신설/삭제/이름 변경
- API 경로 변경
- Phase 상태 변경 또는 구현 방식 근본 변경
- Phase 완료 체크포인트

갱신 시 파일 상단에 "v{n} 변경 요약" 작성. 변경 공통 패턴을 따른다.

---

## req 프로세스 발동 기준

### 즉시 처리 (req 불필요)
아래를 **모두** 충족: CSS 스타일 단독 변경, API 없음, DB 변경 없음, 비즈니스 로직 변경 없음.
단, progress/current.md에 한 줄 기록 필수.

### 무조건 발동
API 엔드포인트 신설·변경 또는 DB 스키마 변경이 수반되는 경우.
"급하게", "간단히" 등 표현과 무관하게 req → contract → 구현 순서를 밟는다.

---

## Contract 작성 의무

### 기본 규칙
- 구현 코드 작성 **전에** contract 먼저 생성. contract 없으면 "in_progress" 불가
- subagent 위임 시 "contract 먼저 작성하고 manifest 갱신 후 구현하라" 포함 필수

### Contract 체크리스트
- [ ] API 엔드포인트 전체 목록 (URL, method, 요청/응답 타입)
- [ ] DB 스키마 변경 (신규 테이블, 컬럼, 인덱스)
- [ ] 역할/권한 조건
- [ ] 비즈니스 규칙 (상태 전이, 유효성 검사 등)
- [ ] 프론트엔드 화면 구성 요소

### 작업 완료 검증 (Definition of Done)
아래 **모두** 충족해야 "done":
1. contract 파일이 존재하고 manifest.json이 가리키고 있다
2. decisions.md에 완료 기록이 있다
3. progress/current.md가 갱신되어 있다
4. TC가 전체 PASS이고 manifest.json tc 배열에 등록되어 있다

→ **하나라도 빠지면 "done" 불가**

### Contract 누락 발견 시
1. 즉시 사용자에게 보고 → 소급 작성 → manifest 수정 → decisions.md 기록
2. 새 작업보다 누락 건 먼저 처리


## 디자인 적용 규칙

`contract/design-system.md`가 존재하는 경우에만 적용한다.

### 기본 규칙
- 프론트엔드 구현 시 design-system.md를 먼저 읽는다
- 색상/폰트/간격은 **토큰 값만 사용**한다. 임의 값 금지
- 컴포넌트(버튼/테이블/입력폼/배지/모달)는 명세에 정의된 스타일을 따른다
- 명세에 없는 패턴이 필요하면 **사람에게 확인 후** design-system.md에 추가한 뒤 구현한다

### 페이지별 예외
- 특정 화면에서 글로벌 토큰과 다른 스타일이 필요하면 `contract/design-override-{모듈}_{reqID}.md` 생성
- override 파일에는 **변경 항목만** 기록 (전체 복사 금지)
- decisions.md에 예외 사유 기록

---

## 프론트엔드 작업 원칙

- 신규 FE 구현이 필요한 경우에만 아래 절차를 따른다:
  1. `feature/ui-2nd` 브랜치의 퍼블리싱된 화면을 현재 브랜치에 머지
  2. 머지된 마크업을 기반으로 API 연동 + FSD 구조로 재조립
  3. 기존 코드 패턴 확인 후 동일한 방식 준수
  4. 구현 완료 후 불필요한 퍼블리싱 껍데기 화면 제거

---

## 백엔드 작업 원칙

- 기존 레포에서 **같은 패턴의 코드**를 먼저 찾고 그 패턴을 그대로 따른다
- Spring Boot + MyBatis 기반 (kit-backend-admin)
- DB 스키마는 별도 개발자가 담당 → contract에 필요한 테이블/컬럼만 명시
- 이해 불가한 코드는 구현하지 않고 **사람에게 보고 후 대기**

---

## 역제안 규칙

검토 관점: 보안/법적 리스크, 기술적 효율성, 유지보수 비용
- ✅ 기술 판단 (비즈니스 의도 불변): decisions.md 기록 후 자체 반영
- ⚠️ 비즈니스 판단 (스코프/방향 변경): 사람에게 선택지 제시, 작업 중단

---

## 사용 가능한 스킬

스킬 목록은 `.claude/skills/registry.md`를 참조한다.

---

## manifest.json 필수 필드

```
project, version, updated_at, phase,
architecture_overview, architecture_overview_history,
design_system (optional),
modules.{mod}.name,
modules.{mod}.items.{id}.title, .status, .current, .contract, .tc[], .history[]
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iduk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
