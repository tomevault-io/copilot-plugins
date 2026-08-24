## pongdang-manager

> 이 파일은 Claude Code가 매 세션 시작 시 가장 먼저 읽고 프로젝트 맥락을 파악하는 자급자족형 지침이다.

# 풍당수학 학원 관리 시스템 — CLAUDE.md

이 파일은 Claude Code가 매 세션 시작 시 가장 먼저 읽고 프로젝트 맥락을 파악하는 자급자족형 지침이다.
세션 마무리 시 Claude Code가 이 파일을 직접 갱신하여 다음 세션으로 컨텍스트를 넘긴다.

---

## 1. 앱 기본 정보

- **앱 이름**: 풍당수학 학원 관리 시스템
- **현재 파일**: `index.html` (단일 HTML 파일)
- **GitHub 저장소**: https://github.com/beybusiness-bit/pongdang-manager
- **배포 URL**: https://beybusiness-bit.github.io/pongdang-manager/

```javascript
const AUTH = {
  ALLOWED_EMAILS: ['pongdang.math@gmail.com'],
  CLIENT_ID: 'TBD',
  SHEET_ID: '1_CFlWnHOi13EFlp8QAUYATdo6lruzXMDfAelxR1Xa0o',
};
```

---

## 2. 앱 아키텍처 요약

- **앱 성격**: 풍당수학 학원장과 강사들이 학생·결제·교재·근무·매출을 한 곳에서 관리하는 내부 운영 도구. 기존에 노션과 구글 시트로 분산해 쓰던 자료를 단일 앱으로 통합 마이그레이션.
- **UI 구조**: 좌측 사이드바 + 메인 콘텐츠. 사이드바 메뉴: 대시보드 / 학생 관리 / 교재 관리 / 결제 관리(서브탭: 결제기록·월별등록대장) / 캘린더 / 리포트(매출 리포트 등). 데스크탑 우선, 일부 페이지는 모바일 비대응 안내.
- **로그인**: Google OAuth 필수. `ALLOWED_EMAILS`에 등록된 계정만 진입 가능.
- **사용자 역할**: 학원장(OWNER) / 선생님(TEACHER) / 보조선생님(ASSISTANT) — 역할별로 세부 권한(보기·쓰기) 분기. 현재 더미 사용자는 학원장 역할.
- **외부 연동**: Google OAuth (로그인) + Google Sheets API (데이터 저장). 추후 문자 CRM, 이메일 등 연동 예정.
- **기술 스택**: 단일 HTML 파일 + 인라인 CSS/JS, GitHub Pages 배포

---

## 3. 세션 운영 원칙

### 세션 시작 시
Claude Code는 **반드시 이 파일을 먼저 읽고** 다음을 파악한 뒤 작업을 시작한다:
- "개발 단계 현황"에서 현재 어느 phase에 있고 무엇이 완료되었는지
- "다음 세션 시작점"에서 당장 이어서 할 작업이 무엇인지
- "가이드 문서 참고 내용 누적"에서 누적된 UX·입력 규칙·주의사항

### 용어 구분 (중요)
- **Phase**: "개발 단계 현황"에 나열된 큰 개발 단위 (예: "결제 관리", "근무일지" 등)
- **작업 단계**: 한 세션 안에서 일을 쪼개서 진행하는 하위 단위
- ⚠️ 둘을 절대 혼동하지 말 것. "작업 단계 하나 끝났다"는 이유로 마무리 작업(이 파일 갱신)을 수행하면 안 된다.

### 새 세션 권유 (Claude Code가 먼저 제안 가능)
다음 경우 사용자에게 새 세션을 권유할 수 있다:
- 한 phase가 완전히 마무리되었을 때
- 컨텍스트가 과도하게 쌓여 작업 효율이 떨어질 때
- 새로운 기능 영역에 진입할 때
- 오류 반복으로 맥락이 꼬였을 때

권유는 권유일 뿐. 사용자의 응답을 기다린다.

### CLAUDE.md 갱신 트리거 (사용자의 명시적 의사 표현 필수)
이 파일의 갱신은 **오직 사용자가 명시적으로 세션 종료 의사를 표현할 때만** 수행:
- "마무리할게" / "세션 끝내자" / "새 세션 열게" 등
- Claude Code의 권유에 사용자가 "응" / "그래" / "그러자" 등으로 동의

❌ **절대 금지**:
- 작업 단계 하나가 끝났다는 이유로 자동 갱신
- Claude Code가 권유한 직후 사용자 응답 없이 자동 갱신
- 사용자 피드백/테스트 결과 받기 전 자동 갱신

### 갱신 시 반영 대상
- 완료된 phase는 ✅로 표시
- "다음 세션 시작점" 업데이트
- DB 구조 변경 사항 반영
- 이번 세션에서 가이드 문서에 참고할 만한 내용을 "가이드 문서 참고 내용 누적"에 추가
- ⚠️ 원본 내용을 임의로 축약·삭제하지 말 것. 사용자와의 논의로 합의된 변경만 수행.

---

## 4. Phase 2: 개발 진행 프로토콜

### 코드 작업 원칙

```javascript
// today() — 반드시 로컬 날짜 기준 (toISOString() UTC 방식 절대 금지)
const today = () => {
  const d = new Date();
  return d.getFullYear() + '-'
    + String(d.getMonth() + 1).padStart(2, '0') + '-'
    + String(d.getDate()).padStart(2, '0');
};
```

- 파일 상단의 `const AUTH = {...}` 상수는 매 작업 후에도 위 실제값을 유지한다.
- 작동하는 코드는 함부로 전체 교체하지 않는다. 부분 수정 우선.
- 사용자가 명시적으로 지시하지 않은 변경·추가 구현을 임의로 진행하지 않는다. 의문이 생기면 반드시 확인.

### 절대 금지 사항
- `<script>` 내부 백틱(`) 중첩
- `<script>` 내부 `&` `<` `>` `"` 직접 삽입 (HTML 파싱 깨짐 위험)
- `innerHTML` 사용 시 null 체크 생략
- `toISOString()` 기반 UTC 날짜 사용
- `</script>` 바깥에 코드가 빠져나오는 구조 (과거 버그 사례 있음 — 반드시 `</script>` 안쪽에서 함수 정의)
- 같은 함수 이름의 중복 정의 (뒤의 정의가 앞을 덮어쓰는 사고 발생함)

### 매 작업 후 필수
- `node --check` 또는 동등한 방식으로 script 블록 문법 검사 (HTML 파일이라 `node`로 직접은 안 되니, script 태그 내부만 추출해서 `new Function(scriptContent)` 검증)
- 사용자가 결과물을 받아볼 수 있는 형태로 전달

### 수정 방식
- 부분 수정: `Edit` 또는 `str_replace` 활용
- 대량 교체(50줄 이상): Python 스크립트로 라인 범위 교체
- 새 함수 추가: 관련 함수 근처에 추가하되, 반드시 `</script>` 안쪽인지 확인

### 모달 중첩 / z-index
- 현재 `openModal()` 함수가 자동으로 z-index를 누적 계산 (1000부터 +10씩)
- 모달 위에 모달 띄우기 가능
- `closeModal()`은 300ms setTimeout으로 display:none 처리하므로, 닫고 바로 다른 모달 열 때는 100ms 정도 setTimeout 권장

---

## 5. Phase 3: 배포 프로토콜

1. `index.html` GitHub 저장소 (https://github.com/beybusiness-bit/pongdang-manager) 에 업로드
2. GitHub Repository → Settings → Pages → Branch: `main` → Save
3. Google Cloud Console → OAuth 클라이언트 ID에 배포 URL(`https://beybusiness-bit.github.io/pongdang-manager/`) 추가
4. ALLOWED_EMAILS 계정들이 OAuth 동의 화면 테스트 사용자로 등록되어 있는지 확인
5. 배포 후 ALLOWED_EMAILS 계정으로 로그인 테스트

빠른 테스트 옵션:
- UI만 빠르게 보고 싶다 → Chrome으로 index.html 직접 열기
- Sheets 포함 전체 동작 → netlify.com/drop 에 드래그 앤 드롭으로 임시 배포
- 최종 → GitHub Pages

---

## 6. Phase 4: 가이드 문서 작성 프로토콜

배포 완료 후 사용자 요청 시, 아래 프롬프트를 완성해서 출력한다.
이 시점에서 "가이드 문서 참고 내용 누적" 항목의 내용을 프롬프트 안에 포함시킨다.

```
지금까지 이 프로젝트에서 개발한 앱의 사용 가이드를 노션에 작성해줘.
노션 MCP로 바로 작성. 작성할 노션 페이지 URL: [URL]

앱 이름: 풍당수학 학원 관리 시스템
접속 URL: https://beybusiness-bit.github.io/pongdang-manager/
로그인: Google OAuth (허가된 이메일만)
허가 이메일: pongdang.math@gmail.com (외 추가된 계정)
데이터 저장: Google Sheets (ID: 1_CFlWnHOi13EFlp8QAUYATdo6lruzXMDfAelxR1Xa0o)

주요 기능 (사이드바 순서): [구현된 메뉴 전체]
주요 워크플로우: [반복 업무 흐름]

개발 과정에서 수집된 가이드 참고 내용:
[아래 "가이드 문서 참고 내용 누적" 항목 전체 삽입]

문서 요건:
- 대상: 기술 배경 없는 처음 사용자
- 구성: 시작하기 → 메뉴별 기능 → 주요 워크플로우 → 주의사항·FAQ
- 각 섹션에 "📸 이미지 추가 위치: [캡처할 화면]" 안내 포함
- 관리자(학원장)용 섹션 포함
```

---

## 7. 개발 단계 현황

✅ **1단계: 기본 구조 및 로그인**
- Google OAuth 로그인
- 권한 시스템 (학원장 / 선생님 / 보조쌤)
- 세부 권한 (보기·쓰기)
- 기본 레이아웃 (사이드바, 헤더)

✅ **2단계: Google Sheets 연동 및 데이터 구조**
- 20개 시트 생성 및 스키마 정의
- Sheets API 연동
- 기본 CRUD 함수

✅ **3단계: 대시보드**
- 학생 수, 매출, 공지 등 간소화된 대시보드
- 공휴일 표시

✅ **4단계: 학생 관리 기본**
- 학생 목록 (필터링, 검색)
- 학생 추가/수정/삭제
- [개선] 학생 추가/수정 모달이 콜백 지원 (월별등록대장에서 재사용 가능)
- [개선] 행 전체 클릭 시 상세 모달 오픈
- [개선] "작업" 컬럼 제거
- [개선] 필터 헤더 '학년' → '학교급' 교정
- [개선] 테이블 헤더/셀 중앙 정렬

✅ **5단계: 학생 관리 고급**
- 학생 상세 페이지 (모달)
- 권한별 데이터 접근 제어
- [개선] 상세 모달의 "수정" 버튼 정상 동작 (중복 함수 제거 + setTimeout 딜레이)

✅ **6단계: 교재 관리**
- 교재 목록
- 교재 CRUD
- 학생-교재 연결
- 프로필 사진 시스템 (Base64)
- 등록/퇴원 기록

🔄 **7단계: 결제 관리** (진행 중)

✅ 완료된 부분:
- 결제 내역 입력 (개별)
- 결제기록 모달 UI 개선
- 요일 양방향 연동 (결제기록 ↔ 학생 관리)
- 더미 데이터 (32개 결제 + 31명 학생, studentId 자동 매핑)
- 결제기록 탭 헤더 sticky (각 `<th>`에 적용)
- 결제 모달: "신입 학생 이름 직접 입력" 필드 제거 → 학생 DB에서 선택하도록 통일
- 결제 모달: 신입 토글 ON일 때 "+ 학생 추가" 버튼 표시
- 결제 모달 ↔ 학생 추가 모달 중첩 흐름 구현 (`openStudentFormFromPayment`)

✅ 월별등록대장 (완료):
- 월별 탭 시스템 (탭 추가 / 검색 / 정렬)
- 월 추가는 앱내 모달 + `<input type="month">` 날짜선택기 사용
- 결제 관리 페이지 진입 시마다 현재 월 자동 선택
- 학교급 판단: `grade` 첫 글자(초/중/고)로 그룹 분류 (schoolType 필드 저장하지 않고 파생)
- 초/중/고 그룹별 테이블 + 각각 합산 행
- 신입 그룹은 학교급 무관하게 별도, 하단에 "+ 신입 추가" 버튼 행 + 합산 행
- 칼럼: No, 학년, 담당T, 학생명, 요일, 교습비, 수강반, 횟수, 입금금액, 수수료, 결제날짜, 입금날짜, 결제수단
- 정렬: 학년 오름차순 → 이름 가나다순 (신입은 학교급 → 학년 → 이름)
- No는 각 그룹 내 독립 카운트
- 행 전체 클릭 시 결제 수정 모달 오픈
- **그룹별 키컬러**:
  - 초등: 노랑 (header `#FFF4B8` / subtotal `#FFEC99` / title `#B58900`)
  - 중등: 연두 (header `#D4F0C5` / subtotal `#BBE8A6` / title `#3E8B3E`)
  - 고등: 주황 (header `#FFD9B3` / subtotal `#FFC28C` / title `#C25A0A`)
  - 신입: 연청록 (header `#C8ECE7` / subtotal `#A5DED6` / title `#2E7D72`)

🔜 7단계 대기 중:
- 사용자 테스트 및 피드백 (현재 시점)
- 피드백 후 결제 관리 나머지 기능 (있다면) 마무리

---

🔄 **교차 작업: UX 개선 및 반응형 대응** (2026-04-21 세션에서 진행, 사용자 피드백 대기)

✅ 이번 세션에 완료하여 배포한 항목 (검증 필요):
- **검색 UX 전면 개선**: 기존 "타이핑 중 리렌더로 한글 조합 깨짐" + "커서 튕김" 문제를 composition-aware 실시간 검색으로 해결. 검색 버튼 없이 타이핑만으로 즉시 필터링. 적용 범위: 학생 검색 / 교재 검색 / 월별등록대장 월 검색 / 결제기록 학생명 검색 (4곳 전체). `handleXxxSearchInput` + `oncompositionstart/end` 패턴 사용. `renderPreservingFocus` 헬퍼로 리렌더 후 포커스 복원.
- **사이드바 구조 재편**:
  - 기존 외부 독립 토글 버튼 제거
  - 사이드바 내부 우측상단에 `.sidebar-collapse-btn` (접기, `✕`) — position: absolute
  - 사이드바 외부에 `.sidebar-expand-btn` (펼치기, `☰`) — `.sidebar.collapsed ~` 조합으로 접혔을 때만 표시
  - 모바일은 기존 `mobile-header`의 햄버거를 `.open` 클래스 토글하도록 `toggleSidebar()` 로직 수정 (데스크톱은 `.collapsed` 클래스, 모바일은 `.open` 클래스 사용)
  - `.page-header { margin-left: 0 }` 기본, 접혔을 때만 60px (펼치기 버튼 공간)
- **학생 관리 뷰 전환 탭**: 기존 `btn btn-primary/btn-outline` → 결제 관리와 동일한 `.tab-btn active` 스타일로 통일
- **`+ 학생 추가` 모달 크래시 수정**: 학생 폼 HTML에 `student-number` 숨김 필드가 없어서 `document.getElementById('student-number').value = ...`가 null 참조 에러로 터짐 → 숨김 input 추가로 해결 (결제 모달 안에서 학생 추가, 그리고 학생 관리 페이지에서 학생 추가 버튼 양쪽 모두 영향받던 버그)
- **행 호버 효과**: `tr.clickable-row:hover` CSS 추가 후 결제기록/월별등록대장 행에 적용
- **신입 그룹 🆕 이모지 제거**: 월별등록대장 신입 그룹 제목
- **모달 열릴 때 스크롤 최상단 리셋**: `openModal()`에 `requestAnimationFrame` → `scrollTop = 0` 추가. 모든 모달에 영향. 상세→수정 전환 시 중간부터 보이던 문제 해결.

✅ 모바일 전용 개선 (사용자가 명시적으로 "데스크탑에 영향 없어야 함" 조건 걸었음):
- **페이지 타이틀/설명 전체 숨김** (모든 페이지): `@media(max-width: 768px) { .page-header { display: none } }`. 이유: `mobile-header`에 페이지명이 이미 노출되어 중복.
- **학생 관리 필터/정렬/버튼 3단 배치**:
  - Row 1: 검색창 (가로 꽉 채움)
  - Row 2: 학교급 필터 + 정렬 셀렉트 (한 줄, 각 50%)
  - Row 3: CSV 업로드 + 학생 추가 (한 줄, 각 50%)
  - 구현: `.filter-sort-row`, `.page-action-buttons` wrapper를 desktop에선 `display: contents`로 투명하게, 모바일에선 `display: flex`로 재배치
- **학생 목록 카드뷰** (테이블 대체): 사용자가 지정한 레이아웃:
  - 좌측상단: 체크박스 / 우측상단: 학생번호
  - 왼쪽: 프로필 사진 56px
  - 오른쪽 3줄: (1) 이름 + 학년, (2) 학교, (3) 담당선생님, 요일
  - 속성명 없이 값만 표시, 호버·클릭은 데스크톱 테이블과 동일 동작 (상세 페이지 이동)
- **등록/퇴원 기록 수정 모달 반응형**: 기존 테이블을 data-label 기반으로 모바일에서 카드 스택 형태로 변환. 유형/날짜/메모/삭제 각각 레이블과 함께 세로 배치.

---

🔄 **교차 작업: 2026-04-22 세션 피드백 반영** (배포 완료, 사용자 테스트 대기)

> 사용자 원문 그대로 (축약 금지):
>
> "1. 사이드바가 펼쳐져있을 때에는 접기 버튼의 테두리와 배경색을 없애서 그냥 사이드바의 우측상단에 X 표시가 있는 것처럼 보이게 해
> 2. 검색창은 모든 페이지에서 한글로 쳤을 때 여전히 깨져서 쳐지고 그래서 검색이 안됩니다. 다시 개선하세요
> 3. 모든 수정/추가 모달에서 이렇게 적용시켜: 모달 상단에 스티키로 바를 두고, 바의 좌측 정렬로는 어떤 모달인지 제목을, 우측 정렬로는 수정 및 닫기 버튼 등 동작 버튼을 띄워서 어느 스크롤에서든 모달 제목과 동작버튼을 볼 수 있게 해줘
> 4. 학생 관리 모바일로 봤을 때 전체 선택 체크박스가 없음. 만들어라
> 5. 학생 관리 모바일로 볼 때 카드뷰 이렇게 개선해: 선택 박스가 카드 우측 상단으로 오게 바꿔. 학생 번호는 학생 이름 오른쪽 옆으로 () 괄호 안에 넣어서 나오게 만들어. 카드 안에서 한 줄에 같이 있는 요소들은 사이에 여백을 지금보다 좀 더 넓혀줘. 요소들 사이 구분좌를 두지 말고 여백으로 구분되게 만들어. 지금 보면 담당선생님과 요일 사이에 콤마로 구분되어있어. 그거 없애."
>
> 사용자 추가 지침 (동일 메시지): "너가 정리해준 항목들 모두 테스트해봤고, 따로 언급을 안하는 항목은 지금 상태가 마음에 드는 거라고 생각하면 돼"
>
> → 이 말은 2026-04-21 세션의 검증 필요 항목 중 **이번에 언급되지 않은 항목은 모두 통과**로 간주해도 된다는 사용자의 명시적 승인. 재질문 불필요.

✅ 이번 세션에 완료하여 배포한 항목 (검증 필요):
- **사이드바 접기 버튼 미니멀 스타일**: `.sidebar-collapse-btn`에서 `background`, `border`, `border-radius` 제거 → `background: transparent; border: none`으로 변경. 크기는 36×36 → 24×24로 축소. 호버 시 색만 `--text3` → `--text`로 변경. 펼친 상태에서 그냥 X 글리프만 우상단에 떠있는 것처럼 보임.
- **검색 한글 IME 깨짐 근본 해결 (구조 재편)**: 기존 composition-aware 방식(re-render 시 DOM 교체 → IME 컨텍스트 손상)을 폐기. **shell + body** 구조로 전환:
  - `renderXxxPage()` = 쉘 렌더 (검색 입력창 포함). 쉘은 뷰 전환/CRUD 시에만 재렌더.
  - `renderXxxListBody()` = 리스트 본체 렌더. 검색·필터·정렬·선택 변경 시 이 함수만 호출 → 검색 input DOM이 교체되지 않음 → IME 컨텍스트 유지.
  - 적용 대상 4곳:
    - **학생 관리**: `renderStudentsPage` + `renderStudentsListBody` (mount `#students-list-body`). 핸들러 변경: `handleStudentSearchInput`, `updateStudentFilter`, `updateStudentSort`, `toggleAllStudents`, `toggleStudentSelection`, `clearSelection`, `changePage`, `changePageSize` 모두 body만 재렌더.
    - **교재 관리**: `renderTextbooksPage` + `renderTextbooksListBody` (mount `#textbooks-list-body`). 핸들러: `handleTextbookSearchInput`, `updateTextbookFilter`, `updateTextbookSort`, `changeTextbookPage`, `changeTextbookPageSize`, `toggleTextbookSelection`, `clearTextbookSelection`, `changeTextbookView`, `toggleAllTextbooksSelection`.
    - **월별등록대장**: `renderEnrollmentLedger`를 `void` 함수로 전환(직접 `#payments-tab-body`에 쓰기) + `renderLedgerMonthTabs` (mount `#ledger-month-tabs`) + `renderLedgerMonthData` (mount `#ledger-month-data`). 월 검색은 탭만 재렌더, 월 선택은 탭+데이터 재렌더. `toggleLedgerSortOrder`는 정렬 버튼 텍스트만 DOM 직접 업데이트 + 탭 재렌더.
    - **결제기록**: `renderPaymentRecords`를 `void` 함수로 전환 + `renderPaymentRecordsListBody` (mount `#payment-records-body`). 핸들러: `handlePaymentRecordSearchInput`, 필터/정렬 onchange, `changePaymentPage`, `changePaymentPageSize`, `togglePaymentSelection`, `toggleSelectAllPayments` 모두 body만 재렌더. `bulkDeletePayments`는 CRUD라 전체 재렌더.
  - **구조 변경의 부작용**: 학생/교재의 `.bulk-actions-bar`가 기존에 tab-bar와 page-actions **사이**에 있었는데, 이제 page-actions **아래** (list body 안)에 위치. 선택했을 때만 보이는 UI라 큰 시각적 차이 없음.
  - 기존 `oncompositionstart/end` 핸들러 및 `renderPreservingFocus` 호출 모두 제거. `renderPreservingFocus` 함수 자체는 남아있지만 호출처 0개 (데드코드).
- **모달 스티키 헤더 일괄 적용**:
  - CSS: `.modal-title { position: sticky; top: 0; z-index: 10; background: white; display: flex; justify-content: space-between; align-items: center; }`. 제목 좌측, 액션 버튼 우측. 모달 스크롤 시 상단 고정.
  - 런타임 마이그레이션 함수 `migrateModalHeaders()` 추가 (DOMContentLoaded 시 1회 실행):
    - 각 `.modal-backdrop`의 `.modal-title`을 `.modal-title-text` (좌측 텍스트) + `.modal-title-actions` (우측 액션 영역) 구조로 전환.
    - 기존 `.modal-title`에 있던 `id`는 `.modal-title-text` 스팬으로 이관 → `getElementById('student-form-title').textContent = '학생 수정'` 같은 기존 코드가 그대로 동작.
    - `.modal-footer` 안의 버튼들을 `.modal-title-actions`로 이동 → 원래 우하단에 있던 취소/저장 버튼이 우상단 스티키로 이동.
    - 이동 후 `.modal-footer`에 `hidden-by-sticky` 클래스 부여 → CSS `display: none`으로 숨김.
    - 이미 제목 영역에 flex가 인라인 스타일로 들어간 상세 모달 (`modal-textbook-detail`, `modal-student-detail`)은 인라인 `style` 제거 후 기존 구조에 클래스만 추가.
  - 영향 범위: 정적 HTML로 선언된 11개 모달 전체.
- **학생 모바일 카드뷰 전체선택 체크박스 추가**: 카드 리스트 바로 위에 `.student-cards-mobile-header` (모바일 전용 `display: flex`, 데스크톱 `display: none`)를 추가하고 "전체 선택" 라벨과 체크박스 배치. 체크박스 클릭 시 `toggleAllStudents(this)` 호출 → 현재 페이지 학생 전부 선택/해제.
- **학생 모바일 카드뷰 레이아웃 개선**:
  - 체크박스: 카드 **우측 상단**으로 이동 (`.student-card-checkbox { position: absolute; top: 12px; right: 12px; }`). 카드에 `padding: 14px 36px 14px 14px` (우측 여백 확대 — 체크박스 공간 확보).
  - 학생 번호: 이름 옆에 `(번호)` 형태로 인라인 표기 (`.student-card-number-inline`). 별도 우상단 번호 삭제.
  - 각 카드 라인 내 요소 간 여백: `display: flex; gap: 14px` (기존 콤마 구분 → flex gap으로).
  - 담당선생님과 요일 사이 **콤마 제거**: `'${teacher}, ${days}'` → 두 개의 `<span>` 요소로 분리. CSS의 gap으로 시각적 구분.
- **기타 구조적 영향**:
  - `textbook-search-input`에 `oncompositionstart/end` 제거, `oninput="handleTextbookSearchInput(this)"`만 유지.
  - `ledger-month-search`, `payment-record-search` 동일.
  - 리스트 mount 패턴으로 변경되면서 `container.innerHTML = ...` 전에 `<div id="xxx-list-body"></div>` 같은 빈 마운트가 들어감 → 초기 렌더에서 `renderXxxListBody()` 호출이 연속으로 일어남.

✅ **학교 DB + 관리 대분류 메뉴 재구성** (이전 세션 완료, main 커밋 `1ac024d`)

- 사이드바 관리 대분류 메뉴명에서 "관리" 제거: 학생/교재/결제/급여/수강반/학교/선생님
- 학교 선택 → 학교급 자동 연결 (`autoFillSchoolLevel()` 함수, schoolsData 연동)
- 학교급 배지 표시 (학생 폼에서 학교 선택 시 자동)
- `schoolType`은 여전히 **DB에 저장하지 않음**. `grade` 첫 글자로 런타임 파생.

---

✅ **교차 작업: 수강반/학교/선생님 페이지 전면 개편** (2026-04-24 세션 완료, 검증 필요)

- **공통**: shell+body 패턴 (검색 IME 보존), 검색·필터·정렬·페이지네이션, 카드/목록 뷰 전환 (데스크탑), 모바일 카드뷰 강제, 체크박스 일괄선택, 행/카드 클릭 → 상세 모달
- **수강반**: 교습비·요일·시간 필드 추가. 상세 모달에 소속 학생 목록 (이름 클릭 → 학생 상세)
- **학교**: 학교급 필터 + 배지. 학생수 표시. 상세 모달에 재원 학생 목록 (이름 클릭 → 학생 상세)
- **선생님**: 정산비율 필드 (학원장만 보임·수정), 메모 필드. `ownOnly` 권한 적용 (선생님 역할은 본인만 보임). 상세 모달에 담당 학생 목록. 수강반/학교는 선생님 역할 읽기만 가능.
- **보조쌤 → 보조선생님** 전면 변경
- `.manage-card-grid`, `.detail-info-grid` CSS 신규 추가
- 관련 상태 변수: `currentClassesFilter/Sort/DisplayView`, `classesPagination`, `selectedClassIds`, `currentDetailClass` (학교·선생님도 동일 패턴)

✅ **8단계: 근무일지** (2026-04-24 세션 구현 완료, 사용자 테스트 대기)

구현 내용:
- 일별 뷰 / 선생님별 뷰 / 학생별 뷰 3탭 구조
- **공휴일**: `KR_HOLIDAYS_MAP` (2025~2027) 하드코딩 + `customHolidaysData` (학원장 직접 관리). 공휴일엔 일별 뷰 상단 배너 표시 + 학생 목록 미표시. `workLogForceOpenDates` 배열로 공휴일인데 수업하는 날 예외 처리.
- **재원 학생 기준**: `getStudentStatus(student) !== '퇴원'` — enrollmentRecords 최신 항목 type 기반
- **일별 뷰**: 날짜 이동(이전 날/다음 날), 학교급(초/중/고/기타) 그룹별 표시, 그룹 내 드래그 순서 조정 (`workLogStudentOrder` 키: `'YYYY-MM-DD_학교급'`). "학생 추가" 버튼 → 검색/필터/정렬+체크박스 다중선택 모달
- **카드 입력 항목**: 수업 내용(textarea), 숙제(textarea), TEST 체크박스(→ TEST이름/난이도/만점/점수 필드 토글), 특강 체크박스(→ 특강명/회차 필드 토글), 코멘트(textarea). 저장 버튼 클릭 시 `saveWorkLogEntry()` 호출 후 toast만 표시 (리렌더 없음)
- **체크박스 토글**: DOM 직접 조작 (`element.style.display`) — 리렌더 없음. 다른 필드 값 보존.
- **읽기 전용 표시**: 값이 있는 필드만 표시, 빈 필드 숨김
- **선생님별 뷰**: 하루/기간 모드 전환. 하루=날짜 컬럼 없음, 기간=날짜 컬럼 좌측. 컬럼: (날짜), 학생명, 수업내용, 숙제, TEST, 난이도, 결과(점수/만점), 코멘트, 특강, 회차. 빈 셀은 공백 (대시 없음)
- **학생별 뷰**: 학생 검색 shell+body 패턴 (IME 보존). 검색 범위: 이름/학교/학년/선생님/요일
- **휴일 관리**: "휴일 관리" 버튼 → 커스텀 휴일 추가/삭제 모달. 학원장만 접근 가능.
- **학생 추가 모달**: 검색(전체속성: 이름/학교/학년/선생님/요일), 학교급 필터, 선생님 필터, 정렬(이름/학년), 체크박스 다중선택, "선택 학생 추가" 일괄 추가

🔜 **피드백톡 주차 표기 기능 (9단계로 이관, 스펙 보존)**:
- 근무일지 카드에서 날짜 표기 / 주차 표기 선택 가능 (기본값: 날짜 표기)
- 주차 표기 선택 시: `getWeekNumberInMonth()` 함수로 해당 날의 월-주차 자동 계산 (예: "26년 4월 2주차")
- 수동 입력 가능: "3~4주차" 등 직접 오버라이드
- 피드백톡(9단계) 구현 시 이 기능도 함께 추가. `{{날짜}}` 변수에 날짜/주차 선택 적용.

✅ **9단계: 피드백톡 시스템** (2026-04-26 세션 완료, 사용자 테스트 대기)

구현 내용:
- **근무일지 카드 체크박스 선택**: 일별 뷰 / 선생님별 뷰 모두 카드/행에 체크박스 추가. 상단 "피드백톡 생성 (N명)" 버튼 — 1명 이상 선택 시에만 표시.
- **템플릿 선택 모달** (`modal-feedback-template`): 템플릿 목록 카드. `feedbackBulkQueue`에 선택 학생 ID 배열 저장.
- **미리보기 모달** (`modal-feedback-preview`): 수정 가능한 textarea. 날짜 표기 / 주차 표기 + 주차 직접입력 옵션. [저장] [복사] 버튼.
- **일괄 처리 모달** (`modal-feedback-bulk`): N명 → 순차(이전/다음). 각 학생마다 내용 수정 가능. 마지막 학생 저장 후 자동 완료.
- **`generateFeedbackTalk(template, varMap)`**: 2단계 처리:
  1. `[IF 변수명]...[/IF]` 블록: 조건 변수가 빈 값이면 블록 전체(정적 텍스트 포함) 제거. regex `/\[IF ([^\]]+)\]([\s\S]*?)\[\/IF\]/g`
  2. 줄 단위 변수 치환: 해당 줄의 모든 변수가 빈 값이면 줄 전체 제거. 연속 빈 줄 → 1개로 정리.
- **`buildFeedbackVarMap(studentId, dateStr, useWeek, weekOverride)`**: 근무일지 데이터 → 변수 맵 생성. 변수명 전부 국문.
- **변수명**: `날짜, 학생명, 수업, 숙제, 테스트, 난이도, 결과, 코멘트, 특강` (모두 한국어)
- **`[IF 변수]` 조건 구역**: 사용자가 템플릿 편집 시 조건부 블록 삽입. `FTF_BLOCKS` 상수로 TEST 구역 / 특강 구역 버튼 제공.
- **클릭 삽입 버튼**: `FTF_VARS` 배열로 변수 버튼, `FTF_BLOCKS` 배열로 구역 버튼. 클릭 시 `insertFtfVar()` / `insertFtfBlock()`로 커서 위치에 삽입.
- **도움말 모달** (`modal-feedback-help`): "? 작성법" 버튼 클릭 → 변수 테이블 + [IF] 문법 설명 + 예시(TEST 있을 때 / 없을 때 비교) 표시.
- **피드백톡 관리 페이지** (사이드바 "피드백톡" 메뉴): `renderFeedbackPage()` (shell+body 패턴)
  - 템플릿 관리: 목록 + 추가/수정/삭제 (type 필드 제거, 이름만)
  - 기록 히스토리: 검색 / 페이지네이션(10/20/50) / 체크박스 일괄선택 / 선택삭제 / 행 클릭 → 상세 모달 + 복사
- **FeedbackTemplates 유형(type) 필드 제거**: 기획에 있었으나 UI에 영향 없음 → 제거. 이름(name)만 사용.
- 더미 기록 데이터 22건 추가 (정상/삭제된 학생/삭제된 템플릿/긴 내용 등 엣지 케이스 포함)

🔜 **미구현 (10단계 이후로 이관)**:
- **주차 표기 선택 기능**: 피드백톡 미리보기 모달의 "주차 표기" 라디오는 UI만 존재, `getWeekNumberInMonth()` 연동 구현 필요
- **문자 발송 버튼**: UI는 있으나 실제 발송 미연결 (10단계 문자 CRM 연동 시 구현)
✅ **10단계: 회의록** (2026-04-27 세션 완료)

구현 내용:
- 기본 CRUD (추가/수정/삭제)
- 날짜/제목/참석자/내용 검색 (날짜 검색 추가됨)
- **상세 모달 인라인 편집**: 별도 수정 모달 없이 상세 모달 자체에서 직접 편집·저장. 학원장만 편집 가능, 선생님은 읽기 전용.
- **할일 인라인 추가/수정/순서변경**: `modal-task-form` 별도 모달 제거. 상세 모달 내 "+ 할일 추가" 버튼 → 인라인 입력 행 (content input + assignee select + 확인·취소 버튼). 수정도 해당 행을 인라인 input으로 교체. ↑↓ 버튼으로 순서 변경.
- **완료/완료취소 양방향 토글**: `toggleTaskComplete()`가 `t.isComplete = !t.isComplete`로 양방향 처리. 별도 "완료취소" 버튼 없음.
- **할일 수정/삭제 아이콘 버튼**: `"수정"/"삭제"` 텍스트 → `"✏️"/"🗑️"` 아이콘으로 교체.
- **삭제 버튼 키컬러**: `background:var(--danger)` → `btn btn-outline btn-sm style="color:var(--danger);border-color:var(--danger);"`.
- **`openMeetingFormFromDetail()` 제거**: 상세 모달이 편집 폼을 통합했으므로 별도 수정 라우트 불필요.
- **`saveMeeting()` 분기**: `currentEditingMeetingId` 있으면 `closeModal('modal-meeting-detail')`, 없으면 `closeModal('modal-meeting-form')`.

✅ **교차 작업: 학원 캘린더 연동** (2026-04-27 세션 완료)
- **근무일지 배지**: teacher 파생 수정 (`w.teacher` 직접 참조 → `studentsData.find(s => s.id === w.studentId)?.teacher`). 해당 날짜 근무일지가 1건 이상인 선생님마다 배지 표시. 배지 클릭 → `calendarWorklogClick(teacherName)` → 근무일지 선생님별 탭 + 필터 이동.
- **회의록 배지**: 해당 날짜 회의록마다 배지 표시. 클릭 → `calendarMeetingClick(meetingId)` → 회의록 탭 + 해당 상세 모달 오픈.
- **시험대비 배지**: 시험기간에 포함된 날짜에 배지 표시. 클릭 → `calendarExamPrepClick(epId)` → 시험대비 탭 + 상세 오픈.
- 배지 클릭 이벤트: `event.stopPropagation()` 포함하여 셀 클릭과 충돌 방지.

✅ **교차 작업: PWA 설치 배너** (2026-04-27 세션 완료)
- `beforeinstallprompt` 이벤트 감지 → 하단 고정 배너 표시.
- `sessionStorage.getItem('pwa-banner-dismissed')` 기반 닫기 (세션 내 재표시 방지).
- `pwaTriggerInstall()`: prompt → userChoice 확인. `pwaDismissBanner()`: 배너 숨김 + sessionStorage 저장.

🔄 **11단계: 시험대비 체크리스트** (2026-04-27 세션 기본 구현 완료, UI 개선 미완성 — 아래 "다음 세션 시작점" 참고)

구현 내용:
- **상태 변수**: `examPrepsData`, `examPrepStudentsData`, `currentExamPrepId`, `currentExamPrepFilter`, `expandedEpStudentIds`, `currentEditingEpId`, `epAddStudentSearch`, `epAddStudentSelected`, `epAddTextbookEpsId`, `epAddTextbookSearch`
- **EP_CHECKS 상수**: 7개 체크박스 정의 배열. `{field, label}`. `checkBookwork/checkBookCorrection/checkErrorNote/checkTest10/checkTextbookMaterial/checkSchoolMaterial/checkConceptSummary`
- **목록 페이지**: 진행중/완료됨 그룹, 완료율 진행바. `getPageContent('exam-prep')` → `<div id="exam-prep-page-content"></div>` 마운트. `renderExamPrepPage()` → `#exam-prep-page-content` 타겟.
- **상세 페이지**: `renderExamPrepDetail(container)`. 현재 모바일/데스크탑 모두 학교별 그룹 + 카드뷰(토글). 체크박스 클릭 즉시 반영 + `renderExamPrepDetailBody()` 재렌더 + `expandedEpStudentIds.add(epsId)` 유지.
- **학생 추가 모달**: 검색 + 중복 방지 ("이미 추가됨" 배지). 현재 `renderEpAddStudentBody()` 전체 재렌더 방식 → 커서 튕김 버그 있음 (다음 세션에서 shell+body 패턴으로 수정 필요).
- **교재 추가/제거**: 학생별 교재 목록.
- **결과/메모 저장**.
- **권한 분기**: 학원장/선생님/보조선생님 접근 제어.

✅ DB 추가 (11단계):
- `ExamPreps`: id, title, subject, examStartDate, examEndDate, prepStartDate, lastClassDate, memo, status (진행중/완료), createdAt
- `ExamPrepStudents`: id, examPrepId, studentId, teacher, boostDays, boostTime, textbooks (배열: id, title), checkBookwork, checkBookCorrection, checkErrorNote, checkTest10, checkTextbookMaterial, checkSchoolMaterial, checkConceptSummary (모두 boolean), result, memo, createdAt
✅ **교차 작업: Firebase 인증 버그 수정 + 선생님 지급방식 3종** (2026-08-01 세션 완료)

- **설정 탭 계정 역할 버그 수정**: Firestore에 저장된 역할 값이 대문자(`'OWNER'`,`'TEACHER'`,`'ASSISTANT'`)인 반면 `ROLES` 상수는 소문자라 `roleLabel()` 비교 실패 → 항상 "보조선생님"으로 표시되던 문제. `fetchAllowedAccountsFromFirestore`, `saveAccountFromModal`, `handleFirebaseUser` 세 곳에 `.toLowerCase()` 정규화 추가.
- **로그인 화면 플래시 제거**: CSS에 `.login-screen { display: flex }`라서 페이지 로드 즉시 로그인 화면이 표시됐다가 Firebase Auth 확인 후 앱으로 전환되는 깜빡임 발생. `display: none`으로 변경 + `#loading-overlay` (흰 배경 + 스피너) 추가 → 항상 로딩 오버레이가 먼저 보이고 인증 상태 확인 후 오버레이 숨김 + 적절한 화면(앱/로그인) 표시.
- **선생님 지급방식 3종 추가** (보조선생님 포함):
  - 비율 정산: 선생님 비율 입력 시 학원 비율 자동 계산 (readonly 칸에 즉시 표시)
  - 기본급+인센티브: 기본급 (원화, 1000단위 콤마 자동 포맷) + "담당학생 N명 초과 시 1명당 M원" 문장형 입력
  - 시급제: 시급 입력. 근무시간은 매출 리포트 급여 계산 섹션에서 월별 입력
  - `fmtCommaInput(el)` / `parseCommaNumber(str)` 헬퍼 함수 추가
  - `updateTeacherPaymentFields()` / `updateAcademyRate()` 함수 추가
  - 매출 리포트 급여 계산 섹션 전면 재설계: 시급제 근무 입력 테이블 → 보조T 근무 배분 → 선생님별 급여(지급방식 배지+계산식) → 보조선생님별 급여 4섹션 구조
  - `updateHourlyEntry(m, teacherId, hours)` 함수 추가
  - `getParams(t)` 헬퍼: 현재 월은 live 데이터, 과거 월은 스냅샷 반환
  - 예시 더미 데이터 3개 추가 (최선생-기본급+인센티브, 정선생-시급제선생님, 한보조-시급제보조선생님)

🔲 **12단계: 매출 리포트**
🔲 **13단계: 운영비용 관리**
🔲 **14단계: 급여 관리**
🔲 **15단계: 설정 탭**
🔲 **16단계: AI 학생 분석 기능**
🔲 **17단계: 학생 리포트 탭**
🔲 **18단계: 이용 가이드**
🔲 **19단계: 문자 CRM 연동** (최후 단계로 이관)
🔲 **20단계: 최종 테스트 및 배포**

---

## 8. DB 구조 (Google Sheets 시트 구성)

> 현재 시점에서 본격 사용 중인 데이터 구조. Sheets 실제 연결은 단계적으로 진행 중이며, 클라이언트 메모리에 더미 데이터로 동작하는 부분도 있다.

### Students (학생)
```
id, number, name, school, grade, birthdate, teacher, class, days,
phone, parentPhone, parentType, address, memo, profileImage,
enrollmentRecords (서브 배열: id, type, date, memo),
createdAt
```
- ⚠️ `schoolType` 필드는 **저장하지 않는다**. `grade`의 첫 글자(초/중/고)로 런타임에 판단.
- `grade` 형식: `"초5"`, `"중2"`, `"고1"` 같은 형태
- `days` 형식: `"월, 수"` 같은 콤마 구분 문자열
- `profileImage`: Base64 인코딩된 이미지 문자열

### Textbooks (교재)
```
id, title, author, publisher, price, level, subject, createdAt
```

### StudentTextbooks (학생-교재 연결)
```
id, studentId, textbookId, assignedDate
```

### Payments (결제)
```
id, studentId, studentName, enrollmentMonth, isNewStudent,
teacherName, days, classId, discountTypeId, count,
baseTuition, discountAmount, finalTuition,
paymentDate, depositDate, paymentMethodId,
depositAmount, fee, isTeacherPayment,
createdAt
```
- `studentName`은 학생 DB에서 파생 (신입/기존 무관하게 `studentId` 기반으로 자동 세팅)
- `enrollmentMonth` 형식: `"2026-04"` (YYYY-MM)
- `isNewStudent`: boolean — 신입 여부
- `isTeacherPayment`: boolean — 강사 직접 수금 여부

### Classes (수강반)
```
id, name, teacher, tuition, days, time
```
- `tuition`: 교습비 (정수, 원 단위). 결제 모달에서 수강반 선택 시 `data-tuition` 속성으로 자동 연동됨 (`loadClassTuition()`)

### DiscountTypes (할인 종류)
```
id, name, discountRate, fixedAmount, memo
```

### PaymentMethods (결제수단)
```
id, name, feeRate
```

### Teachers (선생님)
```
id, name, type (선생님/보조선생님), phone,
paymentType ('ratio'|'base_incentive'|'hourly'),
settlementRate (비율정산 시 선생님 비율 %),
baseSalary (기본급+인센티브 시 기본급, 원),
incentiveThreshold (담당학생 초과 기준 인원),
incentivePerStudent (초과 1명당 추가금액, 원),
hourlyRate (시급제 시 시급, 원/시간),
memo, icon
```
- `paymentType`: `'ratio'`(비율정산), `'base_incentive'`(기본급+인센티브), `'hourly'`(시급제). 기본값 `'ratio'`.
- `settlementRate`: 비율정산 시 선생님 비율(%). 학원장만 조회·수정 가능. 선생님 역할에는 노출 안 됨.
- `baseSalary`, `incentiveThreshold`, `incentivePerStudent`: base_incentive 타입 전용.
- `hourlyRate`: hourly 타입 전용. 실제 근무시간은 `monthlySalaryData.hourlyEntries`에 별도 저장.
- `type` 값: `'선생님'` 또는 `'보조선생님'` (구 `'보조쌤'`에서 변경됨)
- 보조선생님도 3가지 paymentType 동일하게 적용됨.

### MonthlySalaryData (월별 급여 계산 데이터 — 급여 관련 파생 구조)
```
monthlySalaryData[YYYY-MM] = {
  assistants: [{teacherId, totalHours, rate, amount}],  // 비율정산 보조선생님 근무 배분
  hourlyEntries: [{teacherId, hours}],                   // 시급제 선생님+보조선생님 근무시간
  teacherSnapshots: {teacherId: {paymentType, settlementRate, baseSalary,
                                 incentiveThreshold, incentivePerStudent, hourlyRate}},
  ...
}
```
- `teacherSnapshots`: 과거 월 급여 계산 시 당시 파라미터 스냅샷. `getParams(t)` 헬퍼가 현재 월이면 live 데이터, 과거 월이면 스냅샷 반환.
- `hourlyEntries`: 시급제 선생님/보조선생님 월별 근무시간. 매출 리포트 → 급여 계산 섹션에서 입력.

### Schools (학교)
```
id, name, type (초/중/고)
```

### CostCategories (비용 항목 카테고리)
```
id, name, defaultValue, sortOrder
```

### OperatingCosts (운영비용)
```
id, monthYear, categoryId, amount, memo
```

### WorkLogs (근무일지)
```
id, date (YYYY-MM-DD), studentId,
content (수업 내용), homework (숙제),
hasTest (boolean), testName, testDifficulty, testFullScore (number|null), testScore (number|null),
isSpecialClass (boolean), specialClassName, specialClassCount (number|null, 회차 직접입력),
comment (코멘트), createdAt
```
- 한 `(date, studentId)` 쌍에 대해 1개 항목이 원칙. `getWorkLogEntry(date, studentId)` 헬퍼로 조회.
- `specialClassCount`: 자동 계산 아님. 사용자가 직접 입력.
- `saveWorkLogEntry()` 호출 시 신규면 push, 기존이면 `Object.assign(entry, data)` 업데이트. 리렌더 없음.

### CustomHolidays (커스텀 휴일)
```
id, date (YYYY-MM-DD), name
```
- `KR_HOLIDAYS_MAP` (2025~2027 하드코딩) + `customHolidaysData` 배열 조합으로 공휴일 판단.
- `workLogForceOpenDates` 배열: 공휴일인데 수업하는 날 예외 처리용 날짜 배열 (YYYY-MM-DD).

### FeedbackTemplates (피드백톡 템플릿)
```
id, name (템플릿명), content ({{변수}} 포함 템플릿 본문), variables (쉼표 구분 변수목록)
```
- `content` 안의 변수는 `{{변수명}}` 형식 (모두 국문: 날짜, 학생명, 수업, 숙제, 테스트, 난이도, 결과, 코멘트, 특강)
- `[IF 변수명]...[/IF]` 조건 구역: 해당 변수가 빈 값이면 블록 전체 제거
- `variables`는 `"날짜,학생명,수업,숙제,테스트,난이도,결과,코멘트,특강"` 형태
- 기본 제공 템플릿: 평시 / 복귀 / 시험. 사용자 정의 템플릿 추가 가능.
- ⚠️ `type` 필드는 **저장하지 않는다**. 이름(name)만 사용.

### FeedbackTalks (피드백톡 기록) — 9단계에서 사용
```
id, date (YYYY-MM-DD), studentId, templateId, content (생성된 최종 메시지 본문), createdAt
```
- `content`: 변수 치환 + 빈 줄 제거 후 최종 메시지 텍스트
- 발송 여부는 별도 필드 없이 생성 기록만 관리 (발송은 복사/문자로만)

### Meetings (회의록)
```
id, date (YYYY-MM-DD), title, attendees (콤마 구분 문자열), content, createdAt
```
- 할일 목록은 `meetingTasksData` 배열로 별도 관리 (Meeting과 1:N)
- `MeetingTasks`: id, meetingId, content, assignee, isComplete (boolean), completedDate, sortOrder

### ExamPreps (시험대비)
```
id, title, subject, examStartDate (YYYY-MM-DD), examEndDate (YYYY-MM-DD),
prepStartDate (YYYY-MM-DD), lastClassDate (YYYY-MM-DD), memo, status (진행중/완료), createdAt
```
- `status`: 기본값 '진행중'. 학원장이 수동으로 '완료'로 변경 가능.

### ExamPrepStudents (시험대비-학생 연결)
```
id, examPrepId, studentId, teacher, boostDays (보강요일), boostTime (보강시간),
textbooks (배열: [{id, title}]),
checkBookwork (boolean), checkBookCorrection (boolean), checkErrorNote (boolean),
checkTest10 (boolean), checkTextbookMaterial (boolean), checkSchoolMaterial (boolean),
checkConceptSummary (boolean),
result, memo, createdAt
```
- 7개 체크박스는 `EP_CHECKS` 상수 배열로 관리: `{field, label}`.
- `textbooks`: Base64 직렬화 또는 JSON stringify 저장 예정.

### 그 외 시트 (12단계 이후 본격 사용 예정)
공지사항, 급여 등

---

## 9. 가이드 문서 참고 내용 누적

### 학생 관리
- 학생 행 전체를 클릭하면 상세 모달이 뜬다 (이름만 클릭할 필요 없음)
- 상세 모달의 "수정" 버튼은 상세 모달을 닫고 100ms 후 수정 모달을 띄움
- 학교급 필터는 학교급(초/중/고) 단위로만 필터링됨 — 학년 단위는 별도 정렬·검색으로
- "작업" 컬럼이 없는 이유: 행 클릭 = 상세 진입이라 중복 제거함
- 프로필 사진은 Base64로 저장됨 → 너무 큰 이미지는 성능 저하 우려, 사용자에게 적당한 크기 안내 필요

### 결제 관리
- 결제 모달의 "신입 여부" 토글은 결제기록을 신입 그룹으로 분류하는 용도일 뿐, 학생 DB는 신입/기존 모두 동일하게 사용함
- 신입 토글 ON 시에만 "+ 학생 추가" 버튼이 보임 (결제 모달 안에서 학생 추가 가능)
- 결제 모달에서 학생을 선택하면 담당T, 요일이 자동 채워짐
- 결제 모달에서 요일을 학생 원래 요일과 다르게 변경하면 "변경된 요일은 학생 정보에도 반영됩니다" 안내가 뜸 — 저장 시 학생 DB의 요일도 같이 업데이트됨

### 월별등록대장
- 결제 관리 페이지에 진입할 때마다 현재 월 탭으로 자동 리셋됨 (이전에 다른 월 탭을 선택했어도)
- 월 추가는 `+ 월 추가` → 앱내 모달에서 날짜선택기로 월 선택 → 추가
- 신입 그룹은 학교급(초/중/고) 무관하게 별도로 표시되며, 정렬은 학교급 → 학년 → 이름 순
- 그룹별 키컬러: 초등(노랑) / 중등(연두) / 고등(주황) / 신입(연청록)
- 합산 행: 학생 수, 교습비, 입금금액, 수수료 합계
- 신입 그룹 하단의 "+ 신입 추가" 버튼 → 신입 토글 ON + 등록월이 현재 탭 월로 고정된 결제 추가 모달이 뜸

### 데이터 입력 규칙
- 학년은 항상 `"초5"`, `"중2"`, `"고1"` 형식. 학교급 한 글자 + 학년 숫자
- 등록월은 `YYYY-MM` 형식
- 요일은 `"월, 수"` 같은 콤마+공백 구분 문자열
- 모든 날짜는 로컬 기준 `today()` 함수 사용 (UTC 변환 금지)

### UX 주의사항
- 모달 닫고 바로 다른 모달을 띄울 때는 100ms setTimeout 필요 (애니메이션 충돌 방지)
- 모달 중첩 시 z-index는 `openModal()`이 자동 계산해줌
- 같은 함수를 두 번 정의하면 뒤의 것이 앞을 덮어쓰는 사고 발생 → 함수 추가 시 grep으로 중복 확인 필수
- `</script>` 바깥에 코드가 빠져나오면 화면 하단에 글자로 보이고 함수가 실행되지 않음 → 신규 함수는 반드시 `</script>` 안에 있는지 확인
- 모든 모달은 열릴 때 `scrollTop = 0`으로 최상단 리셋 (`openModal`에 내장). 모달이 중간부터 보이는 문제 방지.

### 검색 구현 표준 (프로젝트 공통)
- **실시간 검색 기본**: 타이핑과 동시에 필터링. 검색 버튼·Enter 요구 금지.
- **한글 IME 대응 필수**: `oncompositionstart="this.dataset.composing='1'"`, `oncompositionend="this.dataset.composing='0'; handleXxxSearchInput(this)"` 패턴 사용. 핸들러 진입 시 `if (input.dataset.composing === '1') return;`으로 조합 중 필터 건너뜀.
- **핸들러 구조** (예: 학생): `handleStudentSearchInput(input)` → composing 체크 → `currentStudentFilter.search = input.value` → pagination 리셋 → `renderPreservingFocus(renderStudentsPage)`.
- **참고 구현**: `bey-manager`의 시간표 검색(`handleScheduleSearch`)에서 영감. 거기선 입력 DOM 자체가 정적이라 포커스 유실이 원천 차단됨. pongdang은 전체 페이지 리렌더 구조라 `renderPreservingFocus` 헬퍼로 포커스 + 커서 복원.
- **앞으로 새 검색창을 만들 때**: 반드시 위 패턴 따르기. `onchange`, Enter-only, 검색 버튼 모두 사용 금지.

### 사이드바 구조
- **접기 버튼**: 사이드바 내부 우측상단 (`.sidebar-collapse-btn`, `position: absolute`, 아이콘 `✕`). 사이드바가 접히면 함께 화면 밖으로 이동하므로 자동 숨김.
- **펼치기 버튼**: 사이드바 외부 좌측상단 (`.sidebar-expand-btn`, `position: fixed`, 아이콘 `☰`). 기본 `display: none`, `.sidebar.collapsed ~ .sidebar-expand-btn { display: flex }`로 접혔을 때만 노출.
- **모바일**: `mobile-header`의 햄버거 SVG가 사이드바 열기. 모바일에선 `.sidebar-expand-btn`을 `display: none !important`로 숨김.
- **토글 로직**: `toggleSidebar()`가 `window.innerWidth <= 768` 분기 — 모바일은 `.open` 클래스 토글, 데스크톱은 `.collapsed` 클래스 토글 + localStorage 저장.
- **페이지 헤더 좌측 여백**: 기본 `margin-left: 0`, `.sidebar.collapsed ~ .main .page-header`일 때만 `60px` (펼치기 버튼 회피).

### 모바일 대응 패턴
- **대원칙**: 사용자가 모바일 전용 변경을 요청할 때는 `@media (max-width: 768px) { ... }` 안에서만 처리. 데스크톱에 영향 주지 말 것.
- **페이지 헤더**: 모바일에선 숨김 (`.page-header { display: none }`). `mobile-header`가 페이지명을 이미 제공.
- **리스트 → 카드뷰**: 모바일에서 테이블은 가독성 나쁨. 학생·결제 페이지는 테이블을 숨기고 카드뷰 표시.
  - 학생 카드뷰 구조: 좌측상단 체크박스, 우측상단 번호, 왼쪽 프로필 56px, 오른쪽 3줄(이름+학년 / 학교 / 담당T+요일). 속성명 없이 값만.
- **필터/버튼 다단 배치**: `.filter-sort-row`, `.page-action-buttons` 같은 wrapper에 데스크톱 `display: contents` + 모바일 `display: flex`. wrapper 자체는 데스크톱 레이아웃에 영향 없음.
- **반응형 테이블 (모달 내부)**: `data-label` 속성 + CSS `td:before { content: attr(data-label) }`로 레이블 표시 + `display: block`으로 세로 스택. 등록/퇴원 기록 수정 모달에서 사용.

### 탭 컴포넌트 통일
- 페이지 내 탭은 `.tab-btn.active` 스타일 사용 (결제 관리 / 학생 관리 뷰 전환 공통).
- 버튼 그룹 (`btn-primary`/`btn-outline` 조합)은 탭이 아니라 "모드 선택" 용도로만 쓸 것.

### 수강반/학교/선생님 관리 페이지
- 세 페이지 모두 shell+body 패턴 적용. 검색 input은 shell에 고정 → IME 안 깨짐.
- 행/카드 클릭 → 상세 모달 → 상세 모달의 "수정" 버튼 클릭 → 수정 폼 모달. (closeModal + 100ms setTimeout 패턴)
- **수강반 상세**: 소속 학생 목록 표시. 학생 이름 클릭 시 수강반 상세 닫힘 + 150ms 후 학생 상세 오픈.
- **학교 상세**: 재원 학생 목록 표시. 학교급 배지(초/중/고 색상 구분). 학생 이름 클릭 시 학교 상세 닫힘 + 학생 상세 오픈.
- **선생님 상세**: 담당 학생 목록. 정산비율은 학원장만 보임 (isOwner 분기). 학생 이름 클릭 시 선생님 상세 닫힘 + 학생 상세 오픈.
- **ownOnly 권한**: 선생님 역할은 `teachersData` 중 본인 이름(currentUser.name)과 일치하는 항목만 목록에 표시. 수강반·학교는 선생님도 읽기 가능(쓰기 불가).
- **교습비 자동 연동**: 수강반에 `tuition` 필드 저장 → 결제 모달에서 수강반 선택 시 `loadClassTuition()`이 `data-tuition`으로 교습비 자동 세팅.
- **카드 뷰 CSS**: `.manage-card-grid` (3열 그리드), `.manage-card`, `.manage-card-checkbox` (우상단 절대위치), `.manage-card-icon/name/meta/sub`. 모바일에서는 2열로 전환. 모바일에서는 이모지 아이콘 숨김 (`display: none`).
- **상세 모달 정보 CSS**: `.detail-info-grid` (2열 그리드), `.detail-info-item`, `.detail-info-label`. 모바일에서 1열로 전환.

### 근무일지 (WorkLog) 구현 패턴
- **카드 체크박스 토글 (TEST/특강)**: 리렌더 없이 DOM 직접 조작. `toggleWorkLogTest(studentId)`, `toggleWorkLogSpecial(studentId)` → `element.style.display` 변경. 이유: 리렌더 시 사용자가 입력 중인 다른 필드 값 소실.
- **저장 (`saveWorkLogEntry`)**: 모든 입력 필드 값 읽어서 entry 업데이트 후 `showToast()` 표시만. 리렌더 없음.
- **드래그 순서 (`workLogStudentOrder`)**: 키 = `'YYYY-MM-DD_학교급'`. 그룹 내 studentId 배열 저장. 드래그 완료 시 `renderWorkLogBody()` 호출 (리스트 body만 재렌더).
- **학생별 탭 검색**: shell+body 패턴. `renderWorkLogPage()` 쉘에 검색 input 고정. `handleWorkLogStudentSearch(input)` → `renderWorkLogStudentBody(body)` 만 호출 → IME 보존.
- **공휴일 판단**: `isHoliday(dateStr)` = `KR_HOLIDAYS_MAP[year][MM-DD]` 존재 OR `customHolidaysData` 포함. `workLogForceOpenDates`에 있으면 공휴일이어도 수업일로 처리.
- **`getWeekNumberInMonth(dateStr)`**: 해당 날짜의 월 내 주차 계산 함수. 피드백톡 9단계에서 사용 예정. 현재는 존재하지만 호출처 없음.

### 회의록 (Meeting) 구현 패턴
- **상세 모달 = 편집 모달**: `openMeetingDetail(id)` → 모달이 바로 편집 가능 폼으로 열림. 별도 수정 모달 없음.
- **새 회의록 작성**: `openMeetingForm(null)` → `modal-meeting-form`. 기존 회의록 편집과 분리.
- **할일 인라인 편집**: `showInlineTaskAdd(meetingId)` → 할일 목록 하단에 input row 삽입. `openInlineTaskEdit(meetingId, taskId)` → 해당 행 outerHTML을 input row로 교체. 취소 시 `refreshTasksList()` → 원상 복원.
- **완료 토글**: `toggleTaskComplete(taskId)` → `t.isComplete = !t.isComplete` 양방향. 별도 완료취소 버튼 없음.
- **`saveMeeting()` 분기**: `currentEditingMeetingId`가 있으면 상세 모달에서 호출된 것, 없으면 새 회의록 폼에서 호출된 것. 각각 다른 모달 닫음.

### 페이지 추가 시 필수 체크리스트
새 페이지/탭을 추가할 때 반드시 아래 두 단계 모두 처리:
1. `getPageContent(pageId)` 함수에 케이스 추가 → `<div id="new-page-content"></div>` 반환.
2. `renderNewPage()` 함수가 `document.getElementById('new-page-content')`를 타겟으로 innerHTML 설정.
누락 시: 네비게이션 파괴 또는 레이아웃 "야생" 상태(마진/패딩 없음) 발생.

### 피드백톡 (FeedbackTalk) 구현 패턴
- **핵심 개념**: 근무일지 데이터를 템플릿 `{{변수}}`에 치환 → 학부모 메시지 초안 생성
- **변수명 전부 국문**: `날짜, 학생명, 수업, 숙제, 테스트, 난이도, 결과, 코멘트, 특강`
- **`generateFeedbackTalk(template, varMap)`**: Step1 — `[IF 변수]...[/IF]` 조건 블록 처리 (조건 변수 빈 값이면 블록 통째 제거), Step2 — 줄 단위 치환 (줄의 모든 변수가 빈 값이면 줄 제거), Step3 — 연속 빈 줄 → 1개로 정리.
- **`[IF 변수]...[/IF]`**: 정적 텍스트("TEST :", "난이도 :" 등)도 블록 안에 두면 조건부 제거 가능. 사용자가 직접 작성하거나 "구역 삽입" 버튼으로 삽입.
- **`{{특강}}` 특수 처리**: `isSpecialClass === true`이면 `specialClassName + ' ' + specialClassCount + '회차'`, 아니면 빈 문자열 → 해당 줄 자동 제거
- **`{{날짜}}` 특수 처리**: 날짜 표기(기본) or 주차 표기 선택. 주차 표기 시 `getWeekNumberInMonth()` + 직접 오버라이드 가능. ⚠️ 현재 구현은 UI만 있고 `getWeekNumberInMonth()` 연동은 미완성.
- **피드백톡 생성 버튼**: 근무일지 카드에 체크박스 → 1개 이상 선택 시 "피드백톡 생성 (N명)" 버튼 표시 → 클릭 시 `openWlFeedbackSelected(dateStr)` 호출.
- **저장 독립성**: 근무일지 [저장]과 [피드백톡 생성]은 완전히 별개 동작. 저장 안 해도 피드백톡 생성 가능 (현재 입력 필드값 기준으로 생성).
- **wlCheckedStudentIds / wlCurrentViewStudentIds**: 체크박스 선택 상태. 뷰 전환 시 초기화됨.
- **피드백톡 관리 페이지 패턴**: `renderFeedbackPage()` (shell+body). 검색 input은 shell에 고정. `renderFeedbackHistoryBody()`가 기록 목록만 재렌더. `feedbackHistoryPagination`, `selectedFeedbackTalkIds`, `feedbackHistorySearch` 상태 변수로 관리.
- **템플릿 type 필드 없음**: `feedbackTemplatesData` 객체에 `type` 없음. 이름만 표시. 코드에서 `tpl.type` 참조 금지.
- **동적 모달 생성 (`openFbHistoryDetail`)**: 피드백톡 상세 모달은 정적 HTML 없음. 클릭 시 `document.createElement`로 생성 후 `migrateModalHeaders()` 재호출.

### Firebase 인증 & 계정 관리 (설정 탭)
- **역할 정규화**: Firestore 문서의 `role` 필드는 과거에 대문자(`'OWNER'`, `'TEACHER'`, `'ASSISTANT'`)로 저장된 것이 있음. `ROLES` 상수는 소문자(`owner`, `teacher`, `assistant`). 모든 읽기 지점에서 `.toLowerCase()` 정규화 필수. 새 저장도 항상 소문자로.
- **로그인 플래시 방지 패턴**: `.login-screen { display: none }` (기본 숨김) + `#loading-overlay` (기본 표시). `onAuthStateChanged` 콜백에서 오버레이 숨김 후 앱 or 로그인 화면 표시.
- **`handleFirebaseUser(user)`**: Firestore `allowedAccounts`에서 계정 조회 → `currentUser` 세팅 → 오버레이 숨김 → `#app.active`.
- **계정 없으면**: 오버레이 숨김 → 로그인 화면 표시.

### 선생님 지급방식 3종 구현 패턴
- **`paymentType`**: `'ratio'`(비율정산), `'base_incentive'`(기본급+인센티브), `'hourly'`(시급제). 보조선생님도 동일.
- **선생님 폼 UI**: `#teacher-payment-type` select → `updateTeacherPaymentFields()` → 해당 섹션(`#teacher-section-ratio/base-incentive/hourly`) 표시/숨김.
- **비율정산 UI**: 선생님 비율 입력(`#teacher-settlement-rate`) + 학원 비율 자동계산(`#teacher-academy-rate`, readonly). `updateAcademyRate()` — 입력할 때마다 `100 - 선생님비율` 실시간 표시.
- **기본급+인센티브 UI**: `#teacher-base-salary` (1000단위 콤마) + `#teacher-incentive-threshold` + `#teacher-incentive-per-student`. "담당학생 [threshold]명 초과 시 1명당 [perStudent]원" 문장형 표시.
- **시급제 UI**: `#teacher-hourly-rate` (원/시간). 근무시간은 선생님 폼이 아니라 매출 리포트 → 급여 계산에서 입력.
- **`fmtCommaInput(el)`**: 입력 중 숫자 외 제거 → `parseInt().toLocaleString()` → el.value 업데이트. 원화 금액 입력 칸 전용.
- **`parseCommaNumber(str)`**: 콤마 포함 문자열 → 정수 파싱.
- **스냅샷 패턴**: 과거 월 열람 시 `salary.teacherSnapshots[teacherId]`에 당시 파라미터 저장. `getParams(t)` 헬퍼가 현재 월이면 `t`(live), 과거 월이면 스냅샷 반환.
- **시급제 근무시간 저장**: `monthlySalaryData[YYYY-MM].hourlyEntries = [{teacherId, hours}]`. `updateHourlyEntry(m, teacherId, hours)` 함수로 업데이트.
- **급여 계산 섹션 구조**: (1) 시급제 근무 입력 테이블 (시급제 선생님+보조선생님), (2) 보조T 근무 배분 (비율정산 보조선생님만), (3) 선생님별 급여 계산 테이블 (지급방식 배지 표시), (4) 보조선생님별 급여 계산 테이블.
- **실제 급여 계산 공식**: ratio → `총매출 × 비율%`, base_incentive → `기본급 + max(0, 담당학생수 - threshold) × perStudent`, hourly → `hourlyRate × hours`.

---

## 10. 기기 전환 자동화 (중요)

이 프로젝트는 여러 기기에서 교차 작업됨. Claude Code는 아래 규칙을 **자동으로** 수행한다.
(사용자용 명령어 가이드는 `WORKFLOW.md` 참고)

### 세션 시작 시 동작
사용자가 "시작할게" / "켰어" / "작업 시작" 등을 말하거나, 첫 요청이 들어올 때 **선제적으로**:
1. `git status` — 커밋 안 된 로컬 변경이 있으면 사용자에게 먼저 보고
2. `git pull origin main` — 원격 최신 반영
3. 결과를 1-2줄로 요약 보고 (변경 없으면 "원격과 동일, 최신입니다" 수준)

다만 이미 세션이 진행 중이고 사용자가 단순 요청만 하는 경우엔 굳이 매번 pull 하지 않음. 기기 전환/첫 요청 맥락일 때만.

### 세션 종료 시 동작
사용자가 "끝났어" / "마무리할게" / "다른 기기 옮길게" / "세션 끝내자" 등을 말하면:
1. `git status`로 변경된 파일 확인
2. 변경 목록 + 제안 커밋 메시지를 사용자에게 보여줌
3. 승인 후 `git add [지정 파일들] && git commit && git push origin main`
4. CLAUDE.md 자체의 갱신은 섹션 3의 규칙을 별도로 따름 (명시적 종료 의사 필수)
5. **다음 세션 스타터 메시지 출력** (필수): CLAUDE.md 갱신 후, 아래 형식으로 출력한다.
   - `.claude/settings.json`을 읽어 토큰을 확인하고, 토큰을 포함한 복사용 블록을 채팅 메시지로 출력.
   - 출력 형식:
     ```
     ── 다음 세션 시작 메시지 (아래 전체 복사 후 첫 메시지로 붙여넣기) ──
     시작할게. 깃헙 토큰: [.claude/settings.json에서 읽은 토큰]
     ────────────────────────────────────────────────────────────────
     ```
   - 토큰을 읽는 명령: `python3 -c "import json,re; d=open('.claude/settings.json').read(); print(re.search(r'ghp_\w+', d).group())"`

⚠️ `git add .` 또는 `git add -A` 지양. 바뀐 파일을 명시적으로 지정.

### WIP 상태로 기기 이동
사용자가 "덜 끝났는데 다른 기기 가야 돼" 같은 말을 하면:
- WIP 커밋(`git commit -am "WIP: [간단 설명]"`) 생성 후 push
- 다음 기기에서는 pull 후 이어서 작업 가능
- stash는 로컬 전용이라 기기 이동엔 쓸 수 없음을 명시

### 충돌 발생 시
`git pull` 중 충돌이 나면 사용자 혼자 마커(`<<<<<<<`) 손대지 않게 안내하고, Claude가 직접 분석·해결.
대부분 원인: 다른 기기에서 push 안 한 작업이 남아있음 → 이 가능성 먼저 의심.

### 패치 후 자동 푸시 (기본값)
사용자가 요청한 코드 수정이 끝나면 **매번** 자동으로 커밋 + 푸시한다. 사용자가 배포 URL로 바로 확인할 수 있도록 하기 위함.

흐름:
1. 수정 완료 → `node`로 script 문법 검증
2. 검증 통과 시 **자동으로** `git add [변경 파일들] && git commit && git push origin main`
3. 한 사용자 지시 안에 여러 수정이 있으면 **1개 커밋으로 묶어서** 푸시 (여러 커밋 지양)
4. 푸시 후 안내: "푸시 완료. GitHub Pages 반영까지 ~1분, 화면 그대로면 Cmd+Shift+R (하드 리프레시)"

### 작업 브랜치 → main 자동 머지 (기본값)
세션 지침에 따라 feature 브랜치에서 작업하더라도, 작업 완료 후 **별도 확인 없이 자동으로 main에 머지 + 푸시**한다.
- 이유: GitHub Pages는 main 기준 배포이므로, 브랜치에만 있으면 사용자가 테스트 불가
- 머지 완료 후 작업 브랜치는 그대로 둬도 됨 (삭제 여부는 사용자 판단)
- 예외: 충돌이 있거나 대규모 구조 변경으로 리스크가 있다고 판단될 때만 확인

커밋 메시지: 해당 턴의 수정 내용을 한국어로 1-3줄 요약. Co-Authored-By 라인 포함.

**푸시 보류하고 먼저 확인 권장해야 하는 경우** (Claude가 먼저 제안):
- 대규모 리팩토링·구조 변경
- 테스트가 어려운 복잡한 로직
- 사용자가 명시적으로 "로컬에서 먼저 확인할게"라고 말한 경우
- 문법 검증은 통과했지만 런타임 에러 가능성이 높아 보이는 경우

**CLAUDE.md 자체 변경은 섹션 3 규칙이 우선.** 사용자의 명시적 세션 종료 의사 없이 CLAUDE.md를 변경해서 푸시에 포함하지 않는다. (단, 이 섹션 10의 자동화 흐름에 따라 사용자가 명시적으로 요청한 CLAUDE.md 수정은 함께 푸시해도 됨)

---

## 11. 다음 세션 시작점

**최종 세션 날짜**: 2026-08-01. 최신 커밋: Firebase 인증 버그 수정 + 선생님 지급방식 3종 구현 (`main` 브랜치, 커밋 `e43f1a1`).

**배포 상태**: main 브랜치 = GitHub Pages 배포 브랜치. PWA 설치 가능 상태. 이번 세션에서 배포된 항목: 계정 역할 버그 수정, 로그인 플래시 제거, 선생님 지급방식 3종(비율정산/기본급+인센티브/시급제) + 급여 계산 섹션 재설계.

**사용자 테스트 대기 항목**: 이번 세션에서 배포된 모든 항목 (역할 수정, 로딩 오버레이, 선생님 지급방식 UI/급여 계산).

### 다음 세션 시작 시 할 일

1. `git fetch origin main && git pull origin main`
2. 아래 "미완성 작업 목록"을 순서대로 구현한다. **사용자 원문을 기준으로 한 글자도 누락하지 말 것.**

---

### ⚠️ 미완성 작업 목록 (2026-04-27 세션에서 지시받았으나 컨텍스트 초과로 미완성)

> 사용자 원문(번호 순서, 축약 금지):

#### 결제기록 🔲 미완성
사용자 원문 그대로:
> "결제기록에서 전체선택 아직도 동작 안해. 해당 체크를 누르면 페이지에 있는 모든 결제기록들을 한 번에 선택할 수 있게 작동해야할 거아냐"
> → `toggleSelectAllPayments` 함수가 있으나 현재 페이지의 모든 결제기록을 실제로 `selectedPaymentIds`에 추가하는 기능이 작동 안 함. `filteredPayments` 또는 현재 페이지 slice 기준으로 선택되어야 함.

> "결제기록들이 한개 이상 선택되었을 때에 상단에 삭제, 선택해제, 일괄수정 이런 버튼들이 떠야지"
> → 1개 이상 선택 시 `.bulk-actions-bar`가 표시되어야 하는데 현재 표시 안 됨. 학생 관리의 `.bulk-actions-bar` 패턴 그대로 적용. 버튼: 삭제, 선택해제, 일괄수정. 각 버튼 기능도 실제 동작해야 함.

#### 시험대비 체크리스트 🔲 미완성
사용자 원문 그대로:
> "시험대비 학생 추가 눌러서 검색하려고 하면 한 문자만 쳐도 커서 벗어나는 현상 발생해. 이거는 다른 검색창에서 계속 발생해서 계속 교정을 요청했던 건데 왜 계속 생겨? 다른 검색창 참고해서 고쳐"
> → `renderEpAddStudentBody()` 안에서 전체 body를 재렌더할 때 input DOM이 교체되어 IME 커서 튕김. **Shell+body 패턴으로 수정 필요**: 모달 열릴 때 검색 input을 shell에 고정(`modal-ep-add-student-search` 또는 동등한 ID), 학생 목록만 `#ep-add-student-list-body` div로 분리하여 재렌더. `handleEpAddStudentSearch(input)` 핸들러 → body만 재렌더. 참고: `renderStudentsPage()` + `renderStudentsListBody()` 패턴.

> "시험대비에 추가된 학생들 뷰가 지금 카드로 되고 토글로 여닫게 되어있는데, 이건 모바일에서만 카드뷰로 그대로 가고 데스크탑은 학생 관리나 결제기록 페이지의 데스크탑처럼 테이블로 만들어줘. 들어가는 정보는 모두 동일하고 내가 정보를 전달해준 순서대로 칼럼 순서를 배열해."
> → 데스크탑: 테이블뷰. 칼럼 순서 (스펙 문서 순서 그대로): **이름, 학교, 학년, 담당T, 보강요일, 보강시간, 시험기간(시작~종료), 시험대비시작일, 직전보강일, 시험대비교재, 교재풀기, 교재오답, 오답노트, TEST10개, 교과서자료, 학교자료, 개념정리, 결과, 메모**.
> → 모바일: 카드뷰 유지 (`@media (max-width: 768px)` 내에서만 카드뷰 표시).

> "추가된 학생들은 자동으로 학교별로 그룹화되게 하고, 학교별로 토글로 여닫을 수 있게 해. 토글에 어떤 학교인지 표시해주고, 그룹 안에서 학생들의 정렬을 어떻게할지를 위에서 선택할 수 있게 해줘. 담당T, 학년, 학생이름별로 오름/내림차순 가능하게"
> → 학생들을 `epsItem.school`(또는 `studentsData.find(s => s.id === epsItem.studentId)?.school`) 기준으로 그룹화.
> → 각 학교 그룹마다 토글(여닫기) 헤더에 학교명 표시. `expandedEpSchoolGroups` 상태 Set으로 관리.
> → 상단에 정렬 선택 UI: `epDetailSort` 상태 변수 (`{field: 'teacher'|'grade'|'name', dir: 'asc'|'desc'}`). 각 그룹 내부에서만 정렬 적용.

> "학생별 선택 체크박스, 전체 체크박스, 그룹별 체크박스 만들고 작동할 수 있게 해주고"
> → 학생별 개별 체크박스: `selectedEpStudentIds` Set으로 관리.
> → 그룹별 체크박스: 해당 학교 그룹의 모든 학생 선택/해제.
> → 전체 체크박스: 현재 시험대비의 모든 학생 선택/해제.
> → 모두 실제 동작해야 함.

> "학생 하나라도 선택이 되면 일괄수정(각 속성들을 일괄적으로 수정하는 거임), 삭제, 선택해제 버튼이 생기게 만들어(당연히 기능이 작동도 해야겠지^^?)"
> → 1개 이상 선택 시 `.bulk-actions-bar` 또는 동등한 fixed/sticky 바 표시.
> → **일괄수정**: 선택된 모든 학생의 특정 속성(보강요일, 보강시간, 담당T, 체크박스 등)을 한 번에 수정하는 모달 또는 인라인 폼.
> → **삭제**: 선택된 모든 학생 `examPrepStudentsData`에서 제거 + 재렌더.
> → **선택해제**: `selectedEpStudentIds.clear()` + 재렌더.

---

### 구현 시 주의사항 (다음 세션 Claude에게)

- **shell+body 패턴 적용 위치**: 시험대비 학생 추가 모달의 검색 input은 반드시 modal HTML에 정적으로 선언되어 있어야 함 (재렌더 시 교체 안 됨). 검색 핸들러에서 list body만 재렌더.
- **시험대비 상세 페이지 구조**: `renderExamPrepDetail(container)` 안에서 `renderExamPrepDetailBody()` 분리 패턴. 체크박스 클릭 등은 body만 재렌더.
- **`expandedEpStudentIds`**: 현재 카드 펼침 상태 Set. 데스크탑 테이블뷰로 전환 후에는 이 Set이 불필요할 수 있음. 학교별 그룹 토글 상태는 `expandedEpSchoolGroups`(신규)로 별도 관리.
- **결제기록 bulk action 위치**: `renderPaymentRecordsListBody()` 내에서 `selectedPaymentIds.size > 0`이면 `.bulk-actions-bar` 표시. 기존 학생 관리 패턴과 동일하게.
- **`modal-task-form` 모달**: 10단계에서 인라인 방식으로 전환했으므로 HTML에 남아있더라도 호출처 없음. 삭제해도 됨.
- **캘린더 연동 완료 여부**: `calendarMeetingClick`, `calendarWorklogClick`, `calendarExamPrepClick` 함수 이미 구현됨. `createDateCell()`에서 호출 중. 다음 세션에서 재구현 불필요.

---

### 검증 체크리스트 (8단계 근무일지 + 9단계 피드백톡, 사용자 테스트 미완료)

> 배포 URL: `https://beybusiness-bit.github.io/pongdang-manager/` (main 기준)

**📅 근무일지 — 일별 뷰**
- [ ] 오늘 날짜로 진입, 이전 날 / 다음 날 이동 정상
- [ ] 공휴일 날짜 진입 시 상단에 공휴일 배너 표시 + 학생 목록 없음
- [ ] 재원 학생만 목록 표시 (퇴원생 제외)
- [ ] 학교급(초/중/고/기타) 그룹별 분리 표시
- [ ] 그룹 내 카드 드래그로 순서 변경 → 재렌더 후 순서 유지
- [ ] "학생 추가" 버튼 → 모달: 검색(한글 IME 정상), 학교급 필터, 선생님 필터, 정렬 동작
- [ ] 학생 추가 모달: 체크박스 다중선택 → "선택 학생 추가" 클릭 시 일괄 추가
- [ ] 이미 추가된 학생: "이미 추가됨" 뱃지 표시
- [ ] 카드 - 수업 내용 / 숙제 입력 후 저장 → toast 표시, 카드 리렌더 없음
- [ ] TEST 체크박스 토글 → 필드(이름/난이도/만점/점수) 토글. 다른 필드 값 유지됨.
- [ ] 특강 체크박스 토글 → 필드(특강명/회차) 토글. 다른 필드 값 유지됨.
- [ ] 저장 후 읽기 전용 전환: 값 있는 필드만 표시

**👩‍🏫 근무일지 — 선생님별 뷰**
- [ ] 선생님 필터 드롭다운 동작
- [ ] "하루" 모드: 날짜 컬럼 없음, 나머지 컬럼(학생명/수업내용/숙제/TEST/난이도/결과/코멘트/특강/회차) 표시
- [ ] "기간" 모드: 날짜 컬럼 좌측, from-to 날짜 선택기 동작
- [ ] 빈 셀: 대시 없이 공백으로 표시

**👤 근무일지 — 학생별 뷰**
- [ ] 학생 검색창에 한글 입력 시 IME 깨짐 없음 (shell+body 패턴)
- [ ] 검색 범위: 이름/학교/학년/선생님/요일 모두 검색됨
- [ ] 학생 선택 시 해당 학생의 근무일지 목록 표시

**🗓 근무일지 — 휴일 관리 (학원장만)**
- [ ] "휴일 관리" 버튼 클릭 → 모달 오픈
- [ ] 커스텀 휴일 추가 (날짜 + 이름) → 목록에 표시
- [ ] 커스텀 휴일 삭제 → 목록에서 제거
- [ ] 추가한 휴일 날짜로 일별 뷰 이동 시 공휴일 배너 표시

**💬 피드백톡 — 근무일지에서 생성**
- [ ] 카드에 체크박스 표시됨 (일별 뷰 / 선생님별 뷰)
- [ ] 1명 이상 체크 시 "피드백톡 생성 (N명)" 버튼 표시
- [ ] 템플릿 선택 모달: 템플릿 목록 카드 표시 (유형 배지 없음, 이름만)
- [ ] 1명 선택 → 미리보기 모달: 생성된 내용 textarea로 표시, 직접 수정 가능
- [ ] 미리보기 모달: 날짜/주차 표기 전환 라디오 버튼 동작
- [ ] 미리보기 모달: [저장] → 피드백톡 기록에 저장됨, [복사] → 클립보드 복사
- [ ] N명 선택 → 일괄 처리 모달: 이전/다음으로 순차 처리

**📋 피드백톡 — 관리 페이지**
- [ ] 사이드바에서 피드백톡 메뉴 진입
- [ ] 템플릿 관리: 목록 표시 / + 템플릿 추가 버튼
- [ ] 템플릿 추가 모달: 이름, 내용(변수 버튼 클릭 삽입, 구역 삽입), "? 작성법" 버튼
- [ ] "? 작성법" 버튼 → 도움말 모달: 변수 표 + [IF] 설명 + 예시
- [ ] 템플릿 수정 / 삭제 정상 동작
- [ ] 피드백톡 기록: 22건 표시 (기본 10개/페이지)
- [ ] 기록 검색: 한글 IME 정상, 실시간 필터링
- [ ] 페이지네이션: 이전/다음, 페이지 번호, 페이지당 건수 변경
- [ ] 체크박스: 개별 선택, 헤더 전체 선택, 선택 삭제
- [ ] 기록 행 클릭 → 상세 모달(전체 내용 표시) + 복사 버튼
- [ ] 삭제된 학생 행: "삭제된 학생" 회색 표시 확인 (ftk18)
- [ ] 삭제된 템플릿 행: "삭제된 템플릿" 회색 표시 확인 (ftk19)

---

### 코드만 봐선 파악 안 되는 맥락

- **사용자 피드백 스타일**: 번호 매긴 리스트로 한 번에 여러 개 지시. 각 번호 축약·생략·자의적 해석 금지. "따로 언급 안 한 건 마음에 든 것".
- **shell+body 검색 패턴**: 검색 input은 shell에 고정(재렌더 안 함), 리스트만 body 재렌더. 수강반/학교/선생님/근무일지학생탭 모두 동일 패턴 적용.
- **`renderPages()` 아키텍처**: 로그인 후 `renderPages()`가 각 메뉴별 `<div class="page" id="page-XXX">` + `page-header` + `page-body` 구조를 일괄 생성. `getPageContent(pageId)`가 각 페이지의 inner HTML(마운트 div)을 반환. `navigateTo()`는 active 클래스 토글만 함. **각 render 함수는 반드시 `#xxx-page-content` div를 타겟으로 해야 함 — `#main` 또는 전체 페이지 HTML 직접 수정 금지.** 위반 시 네비게이션이 완전히 파괴됨.
- **새 페이지 추가 시 필수**: `getPageContent(pageId)` 함수에 케이스 추가 → `<div id="new-page-content"></div>` 반환. `renderNewPage()` 함수가 `#new-page-content` 타겟. 빠뜨리면 "야생의 레이아웃"이 됨 (11단계 초기 버그 사례).
- **근무일지 카드 체크박스**: DOM 직접 조작. `toggleWorkLogTest()` / `toggleWorkLogSpecial()` → `style.display` 변경. 리렌더 절대 금지.
- **근무일지 저장**: `saveWorkLogEntry()` 호출 후 toast만. 리렌더 없음. 이게 의도된 설계임.
- **모달 스티키 헤더**: `migrateModalHeaders()` (DOMContentLoaded 1회 실행)가 정적 HTML 모달의 `.modal-footer` 버튼을 `.modal-title` 우측으로 이동시킴. 새 모달 추가 시 확인 필요. 동적 생성 모달(`document.createElement`)은 생성 후 `migrateModalHeaders()` 재호출 필요.
- **보조선생님**: 구 `'보조쌤'`에서 변경됨. DB의 `type` 필드값도 `'보조선생님'`으로 저장.
- **정산비율**: `teacher.settlementRate` (정수%). 학원장만 조회·수정. 선생님 역할에는 노출 안 됨.
- **ownOnly 선생님**: `getVisibleTeachers()`가 teacher 역할 시 `currentUser.name` 매칭으로 본인만 반환.
- **`.claude/` 폴더 untracked**: `.gitignore`에 추가 미완료.
- **`renderPreservingFocus`**: 데드코드. 호출처 없음. 삭제해도 무방.
- **`getWeekNumberInMonth()`**: 존재하지만 현재 피드백톡 미리보기 모달의 주차 표기 라디오와 완전 연동되지 않음. `formatFeedbackDate(dateStr, useWeek, weekOverride)` 함수가 있으나 주차 계산 부분 미완성.
- **피드백톡 `tpl.type` 완전 제거됨**: 과거에 `'평시'/'복귀'/'시험'` 값이 있었으나 코드와 더미 데이터에서 모두 삭제. 새 코드에서 `tpl.type` 접근 금지.
- **`wlCheckedStudentIds` 전역 배열**: 근무일지 카드 체크박스 선택 상태. 뷰 전환(일별↔선생님별↔학생별) 시 초기화 안 됨 — 의도된 동작. `wlCurrentViewStudentIds`는 현재 뷰의 표시 학생 ID 배열 (교집합 계산용).
- **피드백톡 동적 모달 (`openFbHistoryDetail`)**: 정적 HTML 없이 `document.createElement('div')`로 생성. 생성 후 반드시 `migrateModalHeaders()` 재호출해야 스티키 헤더 적용됨.
- **`feedbackBulkQueue`**: 일괄 피드백톡 생성 시 선택 학생 ID 배열. `feedbackBulkIndex`로 현재 처리 중인 학생 인덱스. `feedbackBulkResults`에 생성 결과 누적.
- **시험대비 `EP_CHECKS`**: 7개 체크박스 정의 상수 배열. `{field, label}`. 체크박스 렌더링·집계·저장 모두 이 배열을 순회.
- **시험대비 `currentExamPrepId`**: null이면 목록 페이지, 값이 있으면 상세 페이지 (`renderExamPrepPage()` 내 분기).
- **캘린더 이벤트 연동**: `calendarMeetingClick(id)`, `calendarWorklogClick(teacherName)`, `calendarExamPrepClick(epId)` 함수 구현됨. 배지 클릭 이벤트에 `event.stopPropagation()` 포함.
- **GitHub 푸시 인증**: 세션이 바뀌면 원격 URL이 초기화될 수 있음. `~/.claude/settings.json`의 SessionStart hook이 토큰 기반 URL 복원을 처리함. 토큰은 사용자에게 직접 받을 것 (CLAUDE.md에 기록 금지).
- **Firebase Auth 역할 정규화**: `ROLES` 상수 = 소문자(`owner/teacher/assistant`). Firestore에 대문자로 저장된 구 문서가 존재할 수 있음. 모든 읽기/쓰기 지점에서 `.toLowerCase()` 필수. `handleFirebaseUser`, `fetchAllowedAccountsFromFirestore`, `saveAccountFromModal` 3곳에 반드시 적용.
- **`#loading-overlay`**: 기본 `display: flex` (흰 배경 + 스피너). `onAuthStateChanged` 콜백에서 반드시 숨겨야 함. 로그인 화면(`.login-screen`)은 기본 `display: none`. Firebase 콜백 전 UI 깜빡임 방지를 위한 필수 구조.
- **선생님 지급방식 폼**: `teacher-payment-type` select 변경 시 `updateTeacherPaymentFields()` 호출 → `teacher-section-ratio / teacher-section-base-incentive / teacher-section-hourly` 3개 섹션 표시/숨김 전환. 저장 시 `paymentType`과 해당 타입의 파라미터 필드만 저장.
- **선생님 상세 모달 지급방식 표시**: `isOwner` 분기로 학원장만 볼 수 있음. `dpt = t.paymentType || 'ratio'` 기준으로 분기 렌더링.

---
> Source: [beybusiness-bit/pongdang-manager](https://github.com/beybusiness-bit/pongdang-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
