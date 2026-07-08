## readaloud

> 한국 영어 학원용 PWA. 학생 앱 + 관리자 앱 두 개로 구성.

# 큰소리 영어 (ReadAloudApp) — 작업 컨텍스트

## 프로젝트 개요
한국 영어 학원용 PWA. 학생 앱 + 관리자 앱 두 개로 구성.

- **학생 앱**: `public/index.html` + `public/js/app.js` + `public/style.css`
- **관리자 앱**: `public/admin/index.html` + `public/admin/js/app.js` + `public/admin/style.css`
- **서비스워커**: `public/sw.js` (FCM 백그라운드 알림 + 앱 쉘 캐시)
- **FCM SW**: `public/firebase-messaging-sw.js`
- **Firestore 규칙**: `firestore.rules`
- **Firestore 인덱스**: `firestore.indexes.json`

## 배포
- **Vercel 프로젝트**: `readaloud-app` (harry-kims-projects-2eb6982d)
- **URL**: `https://raloud.vercel.app`
- **GitHub**: `https://github.com/harry3085/ReadAloud` (브랜치: `master`)
- **배포 명령**: `npx vercel --prod --yes`
- GitHub `master` push → Vercel 자동 배포 (Production Branch = master로 설정됨)

## Firebase 프로젝트
- **Project ID**: `readaloud-51113`
- **Auth / Firestore / Storage / FCM** 모두 사용
- Firebase SDK: v10.12.0 (ES Module, CDN)

## 주요 Firestore 컬렉션 (2026-04-23 기준)
| 컬렉션 | 설명 |
|--------|------|
| `users` | 학생/관리자 계정 (role: 'admin'/'student', status: 'active'/'pause'/'out') |
| `groups` | 반(클래스) 정보 |
| `scores` | 시험 점수 — `mode` 표준 키(`vocab`/`fill_blank`/`unscramble`/`mcq`/`subjective`/`recording`) 사용 |
| `notices` | 공지사항 |
| `hwFiles` | 숙제 파일 |
| `userNotifications` | 사용자 알림 |
| `fcmTokens` | FCM 토큰 |
| `payments` | 결제 |
| `savedPushList` | 저장된 푸시 목록 |
| `pushNotifications` | 푸시 알림 목록 |
| `genBooks` | Generator 교재 (관리자 전용) |
| `genChapters` | Generator 챕터 (관리자 전용) |
| `genPages` | Generator 페이지 (관리자 전용) |
| `genQuestionSets` | AI 생성 문제 세트 (관리자 전용) |
| `genTests` | AI 시험 배정 — `testMode` 표준 키 사용. 하위 `userCompleted/{uid}` 에 학생 응시 스냅샷(questions/answers) — 최고점 통과 시에만 저장 |
| `apiUsage` | Gemini API 호출 일일 카운트 (문서 ID = `YYYY-MM-DD`). admin read / signed-in write |
| `tests` | (레거시) 관리자 시험목록 병합 조회용으로만 유지 — 쓰기 중단, 상세 스냅샷 없음 |

### Phase 6E에서 삭제된 컬렉션 (규칙·코드 모두 제거)
- `recHw` / `recSubmissions` / `recFeedbacks` / `recContents` / `recFolders` — AI 녹음숙제(`genTests.testMode='recording-ai'`)로 완전 이전
- `books` / `folders` / `units` (top-level) — 시험지 출력 제거로 더 이상 사용 안 함 (규칙은 현재 유지 중, Phase 6F 후보)

## 관리자 앱 구조 (`public/admin/js/app.js` ~6609줄)

### 핵심 유틸
```js
esc(str)          // XSS 방지 HTML escape — innerHTML 모든 곳에 적용
showToast(msg)    // 토스트 알림
showConfirm(title, sub) // Promise 기반 확인 모달 (confirm() 대신 사용)
showModal(html)   // 모달 열기
closeModal()      // 모달 닫기
```

### 페이지네이션 엔진
```js
// 데이터 → 테이블 렌더링 + 정렬 + 페이지네이션 통합
initPagination(tableId, dataArray, renderRowFn, paginationElId, pageSize)

// 내부 상태: _pageState[tableId] = { data, renderRowFn, page, pageSize, sortCol, sortDir }
renderPage(tableId)          // 현재 페이지 렌더링
refreshPagination(tableId)   // 데이터 유지하며 UI 갱신
```

**중요**: 정렬이 필요한 테이블은 반드시 `initPagination`을 사용해야 함.
직접 `innerHTML` 할당 시 `sortTable`이 작동하지 않음.

### 테이블 정렬
```js
// thead th에 onclick="sortTable('tableId', colIdx)" 추가 시 자동 작동
window.sortTable(tableId, colIdx)
```

### initPagination 적용된 테이블 목록
- `studentTableBody` / `pauseTableBody` / `outTableBody` — 학생관리
- `classTableBody` — 클래스관리
- `noticeTableBody` — 공지사항
- `testListBody` — 시험목록 (tests + genTests 병합)

## 관리자 앱 CSS (`public/admin/style.css`)

### 테마 색상
```css
--teal: #E8714A      /* 주 강조색 (코랄/오렌지) */
--teal-dark: #D85A30
--teal-light: #FEF2EC
--text: #222
--gray: #888
--border: #e0e0e0
```

### 테이블 셀 유틸리티 클래스
```css
td.td-link    /* 클릭 가능한 주요 컬럼 — 굵게, hover 시 teal */
td.td-main    /* 주요 컬럼 — 굵게, 검은색 */
td.td-sub     /* 날짜/보조 정보 — 12px, 회색 */
td.td-mono    /* 아이디 등 고정폭 — monospace */
td.td-center  /* 가운데 정렬 */
td.td-sm      /* 작은 글자 — 12px */
```

**규칙**: `<td>` 인라인 `style=""` 대신 위 클래스 사용.

### 테이블 정렬 헤더
```html
<th onclick="sortTable('tableId', 1)" class="sortable">컬럼명</th>
```

## Generator 페이지 (`public/admin/js/app.js` — loadGenerator 이하)

관리자 앱의 콘텐츠 생성 도구. 이미지 OCR → Firestore Book/Chapter/Page 구조로 저장.

### Firestore 스키마
```
genBooks   { name, createdAt, createdBy }
genChapters { name, bookId, bookName, order, createdAt }
genPages   { title, text, serialNumber, bookId, bookName, chapterId, chapterName,
             ocrConfidence, ocrProvider, imageUrl, edited, createdAt, createdBy }
```

### 상태 변수
```js
_genPages, _genChapters, _genBooks   // Firestore에서 로드한 전체 데이터
_genImages                           // 업로드된 이미지 배열
_genCheckedPages/Chapters/Books      // Set — 체크박스 다중 선택 (툴바 작업용)
_genActiveBook, _genActiveChapter, _genActivePage  // 행 클릭 네비게이션 상태
_genPageCur, _genPageSize=20         // Page 목록 페이지네이션
```

### 인터랙션 설계
- **체크박스 클릭**: `_genChecked*` Set 업데이트 → 툴바 버튼 활성/비활성
- **행 클릭**: `_genActive*` 상태 업데이트 → 필터 + 에디터 로드
  - Book 행 클릭 → Chapter/Page 목록 해당 Book으로 필터
  - Chapter 행 클릭 → Page 목록 해당 Chapter로 필터
  - Page 행 클릭 → 좌측 에디터에 내용 로드
  - 다시 클릭 시 선택 해제 (토글)
- 활성 행은 `var(--teal-light)` 배경 + `var(--teal)` 글자색으로 하이라이트

### 레이아웃
- 4컬럼 flexbox: [에디터(500px, 리사이저)] | [Page | Chapter | Book]
- 에디터 폭 드래그 리사이저: min 250px, max 60%, `localStorage('generator_editor_width')` 저장
- 높이: `calc(100vh - 280px)`

### OCR
- `POST /api/ocr` (api/ocr.js) — Google Cloud Vision DOCUMENT_TEXT_DETECTION
- 이미지 → base64 → API → genPages에 저장
- 환경변수: `GOOGLE_VISION_KEY` (JSON 또는 base64 인코딩 JSON)

### Firestore 규칙
```
match /genBooks/{id}    { allow read, write: if isAdmin(); }
match /genChapters/{id} { allow read, write: if isAdmin(); }
match /genPages/{id}    { allow read, write: if isAdmin(); }
```

## AI 문제 생성 (2026-04-19 추가)

관리자 앱의 두 번째 콘텐츠 생성 도구. Generator Page(본문) → Gemini로 객관식 4지선다 자동 출제.

### 관련 파일
- `api/generate-quiz.js` — Gemini API 호출 서버리스 함수
- `public/admin/quiz-test.html` — 독립 API 검증 페이지 (관리자 앱과 분리)
- `public/admin/js/app.js` 하단 (~6083줄 이후) — `loadQuizGenerate` / `loadQuestionSets` 등 UI 코드

### 관리자 앱 메뉴 2개 (콘텐츠 생성 섹션)
- **AI 문제 생성** (`goPage('quiz-generate')`): Page 선택 → AI 호출 → 미리보기/제외 → 세트 저장
- **문제 세트 목록** (`goPage('quiz-sets')`): `genQuestionSets` CRUD (이름변경/삭제/상세보기)

### API: `POST /api/generate-quiz`
```
body:   { pages: [{id,title,text}], count?: 1~20, type?: 'mcq' }
return: { success, model, questions: [{type,question,questionKo,choices[4],explanation,sourcePageId,difficulty}] }
```

### Gemini 모델 폴백 체인
```js
const GEMINI_MODELS = [
  'gemini-3.1-flash-lite-preview',  // 1순위 (빠르고 저렴)
  'gemini-2.5-flash',                // 2순위 (Preview 실패 시 폴백)
];
```

### 프롬프트 구조 (`api/generate-quiz.js`)
- **시스템 프롬프트** (`SYSTEM_PROMPTS.mcq`): 한국 중·고등 독해 퀴즈 역할 부여 + 5가지 규칙 (문제 유형, 본문 근거, 선택지 4개/정답 1개, easy30/medium50/hard20 난이도 분포, JSON 출력 형식)
- **유저 프롬프트** (`buildUserPrompt`): `[Passage 1] ID/Title/본문` 형식으로 선택된 페이지들 나열 + 문제 수 지시
- **generationConfig**: `temperature:0.7`, `maxOutputTokens:8192`, `responseMimeType:'application/json'` (JSON 강제 모드)

### 제약
- 본문 최대 3000자/페이지 (초과 시 slice), 최소 20자
- 한 번에 최대 10 Page, 1~20문제
- 문제 타입 추가 시: `SYSTEM_PROMPTS`, `validators`, `buildUserPrompt.typeInstructions` 3곳 + 관리자 UI `<option>` 추가 (객체 키 구조라 확장 용이)

### 환경변수
- `GEMINI_API_KEY`: Google AI Studio에서 발급. `.env.local` + Vercel 대시보드 양쪽 등록 필요

### Firestore 규칙
```
match /genQuestionSets/{id}  { allow read, write: if isAdmin(); }
match /genTests/{testId}     { allow read: if isSignedIn(); allow write: if isAdmin(); }
```

### genQuestionSets 스키마
```
{ name, sourceType:'mcq', sourcePages:[{pageId,pageTitle,bookId,chapterId}],
  questions:[...], questionCount, aiModel, aiGeneratedAt, createdAt, createdBy, updatedAt }
```

### 상태 변수 (app.js 하단)
```js
_qgSelectedPageIds  // AI 생성 화면에서 선택된 Page IDs
_qgGenerated        // AI 생성 결과 (미리보기용)
_qgExcluded         // 미리보기에서 체크 해제된 문제 인덱스
_qsList             // 문제 세트 목록 (Firestore에서 로드)
```

### 알려진 TODO (3단계 — 다음 세션)
1. **학생 앱에 신규 `testMode: 'reading-mcq'` 화면 필요** — 현재 학생앱은 단어 meaning 모드만 있고 본문 독해 객관식 UI 없음
2. `genTests` 스키마 확정 + "시험 배정" 관리자 UI (반/학생 선택 → genTests 생성)
3. 학생앱 `tests + genTests` 병렬 조회
4. `qgRenameSet`이 `prompt()` 사용 중 — 기존 규칙(`confirm()` 금지)에 어긋남, `showModal`로 교체 필요

## Firestore 복합 인덱스 (`firestore.indexes.json`)
```json
recSubmissions: hwId(ASC) + uid(ASC)
```
배포: `firebase deploy --only firestore:indexes`

## 서비스워커 (`public/sw.js`)
- **캐시명**: `kunsori-v13` (대규모 배포 후 버전 bump 관례)
- **전략**: 앱 쉘(HTML/CSS/JS) = 네트워크 우선 → 캐시 fallback
- **자동 리로드**: 새 SW 활성화 시 `SW_UPDATED` 메시지 → 클라이언트 자동 리로드

## 보안 규칙 (`firestore.rules`)
- `isAdmin()`: Firestore에서 users/{uid}.role == 'admin' 확인
- `isOwner(uid)`: request.auth.uid == uid 확인
- 대부분 컬렉션: read = 로그인 사용자, write = 관리자 전용

## 주요 버그 수정 이력

### 학생앱 녹음숙제 (2026-04-18)
- **원인**: `recSubmissions` 규칙이 `isOwner`만 허용하는데, 쿼리에 `uid` 필터 없이 `hwId`만으로 조회 → Firestore가 쿼리 전체 거부
- **수정**: `loadRecHwList`, `openRecHwDetail`, `updateRecBadge` 3곳에 `where('uid','==',myUid)` 추가
- **파일**: `public/js/app.js`

### 학생앱 스펠링 시험 모바일 키보드 자동완성 (2026-04-18 / 최종 2026-04-20)
- **원인**: 숨겨진 `<input type="text">`에 `autocomplete="off"` 등을 설정해도 iOS/Android 키보드의 예측 텍스트(QuickType)는 HTML 속성으로 차단 불가
- **최종 해결 (Phase 6C)**: `type="password"` + `autocomplete="new-password"`. iOS 예측 텍스트·비밀번호 관리자 둘 다 안정적으로 차단됨 (commit c0fb278 기준)
- **적용 범위**: `#vqSpellInput` (v2 단어시험), `#spellInput` (레거시 — Phase 6D에서 제거됨), `fb-input-*` (빈칸채우기)

### 관리자앱 모듈 로드 실패 (2026-04-21, Phase 6D 회귀)
- **증상**: `Uncaught SyntaxError: Unexpected token '}'` → `goPage`/`toggleNav` ReferenceError
- **원인**: Phase 6D 대규모 코드 제거 중 `updateNotice` 함수 뒤에 고아 `};`가 남아 ES 모듈 전체 파싱 실패
- **교훈**: 함수 블록 단위 삭제 후 반드시 `node --check *.mjs`로 모듈 모드 파싱 검증 필요 (일반 `node -c`는 module-aware하지 않을 수 있음)

### 학생앱 renderRanking 조기 종료 (2026-04-21, Phase 6E 회귀)
- **증상**: 녹음숙제 랭킹 탭 삭제 시 `if(tab==='score'){...}` 블록의 닫는 `}`도 함께 삭제되어 함수가 파일 끝까지 "삼켜짐"
- **탐지 방법**: `node --check` 자체는 통과 (파일 전체 brace 균형 맞음) — 함수 경계 스캐너 스크립트로 발견
- **수정**: if-score 블록 닫는 `}` 복구

## 작업 규칙 (중요)
1. **XSS**: 모든 사용자 데이터는 `esc()` 필수
2. **confirm/alert 금지**: `showConfirm()` / `showToast()` 사용
3. **테이블**: 반드시 `initPagination` 사용, 직접 innerHTML 할당 금지 (Generator는 커스텀 렌더 사용)
4. **색상**: 테이블 글자는 `var(--text)` (검은색), 보조 정보는 `var(--gray)`
5. **배포**: 변경 후 `git add → git commit → git push origin master → npx vercel --prod --yes`
6. **관리자 모달 레이아웃 패턴** (시험배정 모달 기준 — 2026-04-23 확립):
   - `showModal(html)` **기본 모드는 modalBox의 padding을 0으로 초기화**한다 (`box.style.padding = ''`). 콘텐츠 자체가 header/body/footer 섹션별 padding을 제공해야 함.
   - 표준 구조:
     ```html
     <div style="width:min(XXXpx,92vw);max-height:88vh;display:flex;flex-direction:column;">
       <div style="padding:18px 22px;border-bottom:1px solid var(--border);">
         <!-- 헤더: 타이틀 + 부제 / 우측 배지 등 -->
       </div>
       <div style="padding:16px 22px;overflow-y:auto;flex:1;">
         <!-- 본문: 섹션 헤더 "font-weight:700;font-size:13px;margin-bottom:8px;" 로 구분 -->
       </div>
       <div style="padding:14px 22px;border-top:1px solid var(--border);display:flex;gap:8px;justify-content:flex-end;">
         <!-- 풋터 버튼: 취소/닫기 = btn-secondary, 주 액션 = btn-primary 우측 -->
       </div>
     </div>
     ```
   - 폭 가이드: 단순 상세/확인 → 560px, 폼/배정 → 640px, 복잡한 에디터 → `showModal(html, {fullFlex:true})` 로 860px flex 모드
7. **userCompleted 스냅샷 규칙** (`_writeUserCompleted`):
   - 학생 시험 제출 시 `genTests/{testId}/userCompleted/{uid}` 에 기록
   - `latestScore`/`latestPassed`/`latestAt` 는 매번 업데이트
   - `questions`/`answers`/`score`/`passed`/`date` 등 상세 스냅샷은 **최고점 통과 시에만** 저장 (`passed && score > prevBest`)
   - → 관리자 상세 모달은 `s.score === comp.score && s.date === comp.date` 일 때만 상세 표시
8. **Gemini 모델 폴백 체인** (2026-04-27 유료 티어 전환): 모든 API 가 `2.5-flash-lite → 2.5-flash → 3.1-flash-lite-preview` 순으로 폴백. 같은 모델로 503/429 transient 시 1회 재시도(800ms) 후 다음 모델. 4xx 비-transient 는 즉시 502 반환 (다른 모델도 동일 결과 예상). 변경 시 3개 API 전부 동일 순서 유지: `api/generate-quiz.js` / `api/check-recording.js` / `api/cleanup-ocr.js`. 이전 단일 모델 정책(2026-04-23)은 무료 티어 RPD 한계 + preview 결과 편차 우려였는데 유료 티어로 둘 다 해소됨.
9. **Gemini API 호출 로깅**: 새 Gemini 호출 추가 시 반드시 `_logApiCall(endpoint)` 또는 `_geminiFetch()` 래퍼 경유 — `apiUsage/{YYYY-MM-DD}` 에 자동 카운트
10. **이모지 X, SVG 사용 (2026-06-03 강화)**: 새 UI 작성 시 이모지 직접 박지 말기. 학원장 app.js + 학생앱 app.js 의 `ICONS` 객체에 정의된 SVG 를 `${iconSvg('name')}` 헬퍼로 호출. HTML (`_app.html`) 정적 파일은 인라인 SVG 직접 박음 (헬퍼 못 씀). 새 아이콘은 Lucide 풍 stroke-only (`viewBox="0 0 24 24"`, `stroke-width="2"`, `currentColor` 상속), 학원장+학생앱 양쪽 ICONS 에 동일 추가. **예외**: showToast/showAlert 메시지 안 이모지 (짧은 시각 임팩트), 사용자 명시 요청. 옛 코드 광범위 일괄 변경 X (회귀 위험). 자동 변환 후 `node --input-type=module --check` 필수.

## 옛 세션 이력 (~2026-05-15)

2026-04-19 ~ 2026-05-15 의 세션 이력 (Phase 6 작업·멀티테넌시 전환·결제 v2·브랜딩·SSR·녹음숙제·말하기·AI 사용량 한도 재설계 등) 은 [docs/claude-md-archive/2026-04-19-to-2026-05-15.md](docs/claude-md-archive/2026-04-19-to-2026-05-15.md) 로 분리됨 (2026-05-23 컨텍스트 부담 완화). 옛 작업 맥락이 필요하면 Read 도구로 그 파일을 직접 fetch.

---

## 2026-05-16: 말하기 시험 userCompleted 미생성 버그 + 결과 표시 정비

당일 SW v529 → v535 (~8 commit). 학원장 "통과했는데 목록에 미완료" 보고에서
출발 → 근본 원인(말하기 answers undefined) 추적 → 차단·복구·표시 정비 종합.

### 1) userCompleted 미생성 근본 버그 (commit `d9faa59`)

**증상**: 문성미 '중1 마더텅 영어듣기 ch10' 말하기 통과(90점)했는데 학생앱·
학원앱 목록에 미완료 유지. 같은 시험 통과자 전원(용주영 등) userCompleted 0건.

**진단**: `scores` 는 `addDoc` 으로 정상 박힘(score=90 passed=true). 그러나
`_writeUserCompleted` 의 `setDoc` 가 throw → `_vqSubmit` 안쪽 `catch` 가
`console.warn` 으로 **조용히 삼킴** → userCompleted 미생성 → 목록 완료
판정(`userCompMap.get(t.id)?.score !== undefined`) 영영 false.

**throw 이유**: 말하기 answers 의 `spkAttempts` 등이 5/14~16 음성인식
대규모 변경(3차 MR 흐름, commit 18개)의 특정 경로에서 `undefined`.
Firestore 는 객체 어디든 undefined 있으면 setDoc **전체** 거부.

**수정 (옵션 1 — 공용 함수 방어, 음성인식 코드 무관)**:
- `questions`/`answers` JSON 왕복(`JSON.parse(JSON.stringify())`)으로 깊은 곳 undefined 제거
- top-level undefined 키 제거 (serverTimestamp sentinel 은 undefined 아니라 보존)
- setDoc 실패 시 `console.error` + 사용자 토스트 + **re-throw** (조용히 삼키지 않음)
- 모든 시험 유형(vocab/mcq/fill_blank/unscramble) 예방 보호. 다음 응시부터 적용

### 2) 누락 응시자 백필 (commit `37215c6`)

`scripts/migrate/backfill-usercompleted-from-scores.js` — scores `passed=true`
→ (testId,uid) 최고점 → userCompleted.score 없으면 최소 필드 백필.
- DRY-RUN/--apply. `_backfilledFromScores:true` 마커
- **default 9건 적용** (문성미 5·이성민·전지윤·정하윤·용주영, 전부 vocab/speaking)
- `questions`/`answers` 는 scores 에 없어 생략 → 목록·점수 정상, 상세는 작업규칙7 폴백

### 3) 다른 모드 영향 전수검사 — 말하기 전용 확인

백필 스크립트가 4모드(vocab/mcq/fill_blank/unscramble) 전부 스캔 → 대상
9건 전부 `vocab` + format 확인 결과 **전부 speaking**. mcq/fill_blank/
unscramble/일반vocab 누락 **0건**. → 말하기(speaking) 전용 버그 확정.

### 4) stale 전수검사 (commit `b5ee1c7`)

`scripts/diag/check-stale-usercompleted.js` — userCompleted 는 있지만
scores 최고점 > userCompleted.score (재응시 최고점 미반영, 김다윤 케이스).
- **stale 2건**: 문성미·김다윤 (둘 다 vocab/speaking, scores 100점인데
  userCompleted 83·90점 — 5/16 재응시가 setDoc 실패로 미반영)
- **옵션 A 채택** — 데이터 손대지 않음. 코드 수정(d9faa59) 완료라 재응시
  하면 자동 완전 복구. 점수만 갱신 시 questions/answers(유실) 불일치라 비권장

### 5) 결과화면 음역 멘트 누락 fix (commit `be6f990`)

`_cleanAiReason`(C 옵션, 5/15) 은 reason 텍스트만 정제. `aiHeard` 를 직접
출력하는 라인 5154 `"OOO처럼 들릴 수 있어요"` 는 별도 경로라 정답 검사
없이 무조건 출력 → 정답 말해도 떴음. 케이스2(AI 정밀통과)에서 aiHeard
소문자 trim 비교 → 정답(q.word)과 같으면 그 줄 숨김. 다른/동음이의어면 유지.

### 6) 상세 들린단어 spkAiHeard 우선 (commit `23d8a41`)

**발견**: AI 통과 시 `_vqSpkFinalize(true, q.word)` → `spkHeard = 정답
그 자체`. 상세 모달의 들린 단어가 정답으로 박혀 학생 실제 발음(AI 인식값
= spkAiHeard) 안 보임 — AI 정밀 결과를 버리는 셈.
- `_vqBuildDetail`(학생) + `_adminVocabBuildDetail`(학원장) 들린 단어를
  `spkAiHeard || spkHeard` 로. 학원장 동음이의어 매칭 판정도 spkAiHeard 우선
- 표시 레이어만 변경 — 데이터·저장 로직 무관. 옛 데이터는 spkHeard 폴백

### 7) 학원장 상세 — 정확도·시도횟수 표시 (commit `dd9f2f0`, `48048ee`)

`confidence`·`spkAttempts` 는 이미 받던 데이터(추가 비용 0). `_adminVocabBuildDetail`
speaking 줄에 학원장 전용 추가:
- **정확도 N%** (spkAiConfidence, 90+초록/70-89주황/<70빨강) — AI 추정값이라
  학생 비노출(시험점수와 혼동·혼란 우려). AI 경로만 (Web Speech 통과·옛 데이터 미표시)
- **N회** (spkAttempts — 단어 1개 내 1·2차 Web Speech / 3차 AI 중 통과 시도.
  객관 사실이라 혼란 없음)

### 8) 성적 상세 'N회 응시 중 X번째' (commit `f913e49`)

시험 전체 재응시 횟수 — 별도 카운터 없음, scores doc 건수로 계산.
`showScoreDetail` 에 그 학생·그 시험 scores `createdAt` asc 정렬 → 현재
기록 순번. 헤더에 보라색 `4회 응시 중 3번째` / `1회 응시`.
- 인덱스 `scores (testId+uid+createdAt)` 추가·deploy·빌드 완료 후 적용
- try/catch — 빌드 지연·실패 시 순번만 생략, 모달 정상

---

## 작업 규칙 추가 (2026-05-16)

신규:
- **Firestore undefined → setDoc 전체 거부** — 객체·배열 어디든 `undefined`
  하나면 그 doc write 통째 실패. 사용자 입력·동적 필드가 들어가는 공용
  저장 함수(`_writeUserCompleted` 등)는 `JSON.parse(JSON.stringify())` 또는
  재귀 sanitize 로 방어. (serverTimestamp sentinel 은 JSON 왕복 대상에서
  제외 — top-level 만 undefined 키 삭제, sentinel 은 보존됨)
- **안쪽 catch 가 조용히 삼키면 안 됨** — `try{ await write }catch(e){
  console.warn }` 패턴은 실패가 묻혀 "scores 는 있는데 userCompleted 없음"
  같은 비대칭 유발. 최소 사용자 토스트 + re-throw(호출자 인지). 디버깅 가능하게.
- **표시값과 저장값 분리 인지** — `spkHeard` 는 AI 통과 시 `q.word`(정답)
  가 박혀 "실제 발음"이 아님. 실제 발음은 `spkAiHeard`. 상세 표시는
  `spkAiHeard || spkHeard` 우선. 진단·통계 필드(spkAiConfidence/spkAttempts)는
  저장만 되고 표시는 별도 결정.
- **AI 자체 추정값(confidence)은 학생 비노출** — 객관 측정 아닌 주관 추정.
  절대 수치로 학생에게 보이면 시험점수와 혼동·혼란. 학원장 참고용으로만.
  객관 사실(spkAttempts 시도횟수, 응시 순번)은 노출 OK.
- **응시 횟수는 컬렉션 doc 건수** — 별도 카운터 필드 없음. scores 는 매
  응시 addDoc → (testId,uid) doc 건수 = 재응시 횟수. createdAt 정렬로 순번.
- **버그 영향 전수검사 = 백필 스크립트로 모드 전부 스캔** — 특정 유형 버그
  의심 시 4모드 다 스캔해서 실제 분포 확인(추측 X). 9건 전부 speaking →
  말하기 전용 확정. stale(있지만 미반영)은 별도 스크립트로 구분 검사.
- **admin SDK 진단은 Firestore Rules 우회 — 클라 Rules 영향 쿼리는 F12
  병행 필수** — `scripts/` 의 firebase-admin 진단은 Rules 무시(전권). 클라
  에서 권한 거부되는 쿼리도 admin 진단은 "데이터 정상"으로 나옴. 클라
  화면 버그는 admin SDK 진단만으로 단정 X — 학원장/학생 F12 콘솔 에러
  (`Missing or insufficient permissions`) 확인 병행. _srLoadTestMeta
  배지 전멸이 표본 (admin 진단 정상 → F12 로 permission-denied 확정).
- **academyId 검증 Rules 컬렉션은 쿼리에 academyId 정적 제약 필수** —
  `allow read: resource.data.academyId == myAcademyId()` 인 컬렉션
  (genTests 등) 을 `where(documentId(),'in',[...])` 또는 academyId 없는
  쿼리로 클라 조회하면 Firestore 가 "Rules 만족 보장 불가" → 쿼리 전체
  permission-denied. **해결: (a) 쿼리에 `where('academyId','==',MY)`
  동반, 또는 (b) 단일 `getDoc(doc(...))` — nested rule `match
  /{id}` 가 각 doc 의 academyId 평가 → 같은 학원 통과**. 2026-05-16
  수평 전개 결과 `_srLoadTestMeta` 가 유일 사례 (학생앱 genTests in
  쿼리는 academyId 동반·안전, collectionGroup userCompleted 는 uid
  정적 제약·Rules-aware 설계·안전, super adminLogs 는 isSuperAdmin only).

---

## 파일 크기 / SW 캐시 (2026-05-16)
- `public/js/app.js`: +~25줄 (sanitize + 실패토스트 + aiHeard 정답검사 + spkAiHeard 우선)
- `public/admin/js/app.js`: +~30줄 (spkAiHeard 우선 + 정확도·시도횟수 + 응시 순번)
- `firestore.indexes.json`: +1 (scores testId+uid+createdAt)
- 신규 스크립트: `backfill-usercompleted-from-scores.js` / `check-stale-usercompleted.js`
- SW 캐시: `kunsori-v535`

## 진행률 (2026-05-16)
- 말하기 시험 userCompleted 버그: **~100%** (근본 차단 + 9건 백필 + stale 2건 진단 + 전수검사)
- 말하기 결과·상세 표시 정비: **~100%** (음역 멘트·spkAiHeard·정확도·시도횟수·응시순번)
- 단어시험 채점 견고성·동음이의어·음성 인식: ~100% (변동 없음)
- 멀티테넌시·결제·브랜딩·super 앱: 변동 없음
- Phase 5 출시 준비: 0%

## 다음 세션 후보 (2026-05-16 갱신)
1. **stale 2건 학원장 안내** — 문성미·김다윤 재응시 시 자동 완전 복구 (코드 수정 완료)
2. **Phase 5 출시 준비** — 도메인 / 약관 / 결제 PG 연동
3. **학원장 대시보드 달력 보강** — 생일 카테고리
4. **v1.0 Polish 사이클** ([memory/project_v1_polish_cycle.md](memory/project_v1_polish_cycle.md))
5. **super 앱 reads P2** ([memory/project_super_reads_p2_after_billing.md](memory/project_super_reads_p2_after_billing.md)) — 결제 완성 후

**완료 (이 세션, 2026-05-16)**:
- ✅ userCompleted undefined sanitize + 실패 토스트 + re-throw (근본 차단)
- ✅ 백필 9건 복구 (문성미·이성민·전지윤·정하윤·용주영, 전부 말하기)
- ✅ 다른 모드 영향 전수검사 (말하기 speaking 전용 확정)
- ✅ stale 2건 진단 (문성미·김다윤, 옵션 A 재응시 복구)
- ✅ 결과화면 aiHeard 정답 시 음역 멘트 숨김
- ✅ 상세 들린단어 spkAiHeard 우선 (AI 실제 발음 반영)
- ✅ 학원장 상세 정확도·시도횟수 표시 (학생 비노출)
- ✅ 성적 상세 N회 응시 중 X번째 + scores 인덱스
- ✅ 작업 규칙 보강 — Firestore undefined / catch 삼킴 금지 / 표시값·저장값 분리 / AI 추정값 비노출 / 응시 횟수 doc 건수

---

## 2026-05-16 (이어서): 성적리포트 배지 Rules 버그 + 삭제시험 안내 + 결제 입금체크 + attemptLabel Rules

SW v537 → v540 (~6 commit). 학원장 보고 연쇄 진단 — 성적리포트 배지·상세 표시 + 결제 입금체크.

### 9) 성적리포트 배지 전멸 — _srLoadTestMeta Rules 충돌 (commit `fdfedbc`)

증상: 성적 리포트 시험명에 🎤 말하기 / 📐 문법 배지 **전부 누락**.
F12: `app.js:4841 test meta fetch: Missing or insufficient permissions`.

원인: `_srLoadTestMeta` 의 `where(documentId(),'in',chunk)` (genTests) 가
genTests Rules(`academyId == myAcademyId`)와 충돌 — in 쿼리에 academyId
정적 제약 없어 Firestore permission-denied → catch → speakingMap/grammarMap
**항상 빈 객체** → 모든 행 배지 false. 시험목록·시험관리는 genTests 직접
로드라 정상이었음(성적리포트만 별도 메타 in쿼리).
**admin SDK 진단은 Rules 우회라 "정상"으로 오판 → F12 로 확정.**

수정: in 쿼리 → `Promise.all(testIds.map(getDoc))`. 단일 doc read 는
`match /genTests/{testId}` 가 각 doc academyId 평가 → 같은 학원 통과. reads 동일.

수평전개: `documentId() in` / academyId 제약 없는 쿼리 전수 점검 →
`_srLoadTestMeta` 가 유일(학생앱 genTests in 은 academyId 동반·안전,
collectionGroup userCompleted 는 uid 정적 제약·Rules-aware·안전, super
adminLogs 는 isSuperAdmin only·안전).

### 10) 삭제 시험 vs 레거시 안내 문구 분기 (commit `201b237`)

성지율 mcq 90점 첫통과인데 "레거시 시험" 표시 → 학원장 혼란. 진단: 학원장이
그 시험 삭제 → userCompleted cascade 제거, scores 만 이력 보존. showScoreDetail
의 `!genTest` 분기가 삭제·진짜레거시 구분 없음. 수정: `s.testId` 유무로 분기
— testId 있는데 genTests 없음 = "삭제된 시험(점수 보존, 상세는 삭제 시 제거)",
testId 빈값 = 기존 "레거시" 문구.

### 11) scores 에 speaking/grammar 메타 보존 (commit `567b77b`)

배지가 genTests 메타에만 의존 → 시험 삭제 시 같은 학생 성적리포트에서
살아있는 시험은 배지 O, 삭제 시험은 X (불일치). 응시 저장 시 scores 에
메타 박음: vocab `_vqSubmit` → `vocabFormat`, mcq submit → `subType`.
`_srNormalize` 가 scores 자체 필드 우선 → 없으면 genTests 폴백.
앞으로 응시분은 시험 삭제돼도 배지 유지 + genTests fetch 줄어 reads↓.
이미 삭제된 옛 건은 소스 없어 복구 불가(불가피).

### 12) 결제관리 입금 체크 즉시 풀리던 버그 (commit `ffe9c8f`)

학원비 입금 체크 → 바로 지워짐. 원인: `_billingToggleChannel` 이 updateDoc
직후 `_renderBillingGrid()` (refetch=true 기본) → Firestore eventual
consistency 로 stale snapshot 받아 체크 풀린 상태로 덮음 + 메모리 캐시
미갱신. 2026-05-08 결제 패널서 고친 패턴이 이 토글 함수만 누락. 수정:
메모리 캐시(b — _billingsByMonth ref) 즉시 반영 + `_renderBillingGrid(0,{refetch:false})`.

### 13) attemptLabel Rules 충돌 — N회 라벨 안 뜸 (commit `3b4e369`)

문성미 '단어 Mr Brown' ~25회 응시인데 상세모달에 "N회 응시 중 X번째"
라벨 없음. 원인: `f913e49`(attemptLabel) 쿼리 `where(testId)+where(uid)+
orderBy(createdAt)` 에 academyId 정적 제약 없음 → scores Rules
(`academyId==myAcademyId`) 충돌 → permission-denied → catch → 라벨 ''.
**§9 _srLoadTestMeta 와 동일 함정을 신규 코드(f913e49)에 반복** — 수평전개는
기존 코드 대상이라 직후 작성한 신규에 작업규칙 미적용한 실수.
수정: 쿼리에 `where('academyId','==',s.academyId||MY)` + 인덱스
`scores(testId+uid+createdAt)` → `(academyId+testId+uid+createdAt)` 교체·
deploy·빌드. 옛 인덱스는 grep 전수 사용처 0건 확인 → `--force` 정리.

### 인덱스 무한 증가 우려 — 답변

Firebase composite index 한도 200/프로젝트(현 ~45). Console 에 사용 통계
**없음** — 사용 여부는 **코드 grep 으로 판별**(인덱스 정확 조합 ↔ 쿼리 대조,
prefix 규칙 고려). 애매하면 보존(인덱스 더 있어도 쿼리 안 깨짐, 삭제 실수가
더 위험). prefix 규칙: `[academyId,testId,uid,createdAt]` 1개가 academyId
단독·+testId·+uid 쿼리 모두 커버 → 잘 설계하면 인덱스 1개로 다수 쿼리 재사용.

---

## 작업 규칙 추가 (2026-05-16 이어서)

신규:
- **수평전개 후 작성하는 신규 코드에도 그 작업규칙 즉시 적용** — 수평전개는
  기존 코드만 점검. 직후 추가하는 코드가 같은 함정 반복 가능(f913e49 가
  _srLoadTestMeta 와 동일 Rules 함정 반복이 표본). 신규 쿼리 작성 시
  "academyId 검증 Rules 컬렉션이면 academyId 정적 제약" 체크리스트 적용.
- **Firestore 인덱스 사용 여부는 코드 grep 으로만 판별** — Firebase Console
  사용 통계 없음. 인덱스 정확 조합(academyId 유무·orderBy 유무·순서) ↔ 코드
  쿼리 1:1 대조 + prefix 규칙. grep 0건 = 안전 삭제 / 애매 = 보존. 쿼리
  교체 시 옛 인덱스 `--force` 정리(컬렉션 폐기 cleanup 3종 세트와 동일 맥락).
- **결제 등 토글 후 refetch:false + 메모리 캐시 즉시 반영** — `updateDoc`
  직후 `getDocs` refetch 는 Firestore eventual consistency 로 stale.
  토글류는 메모리 캐시(reference) 즉시 갱신 + `{refetch:false}` 로 캐시
  렌더(2026-05-08 결제 패널 패턴 — 신규 토글 함수마다 누락 없는지 확인).
- **`!genTest` 안내는 삭제 vs 레거시 분기** — `s.testId` 있는데 genTests
  없음 = 학원장이 삭제(scores 만 보존). testId 빈값 = 진짜 옛 레거시. 문구
  구분으로 학원장 "내가 삭제해서구나" 즉시 이해.

---

## 파일 크기 / SW 캐시 (2026-05-16 종료)
- `public/admin/js/app.js`: +~30줄 (_srLoadTestMeta getDoc·삭제문구·_srNormalize·결제토글·attemptLabel academyId)
- `public/js/app.js`: +~3줄 (vocab/mcq scores 메타)
- `firestore.indexes.json`: scores 인덱스 academyId+testId+uid+createdAt 교체 (옛것 --force 정리)
- SW 캐시: `kunsori-v540`

## 진행률 (2026-05-16 종료)
- 말하기 시험 userCompleted 버그·결과표시: ~100% (변동 없음)
- **성적리포트 배지·상세 표시: ~100%** (Rules 버그 fix·삭제문구·메타보존·attemptLabel)
- **결제관리 입금 체크: ~100%** (eventual consistency fix)
- 멀티테넌시·super 앱·브랜딩: 변동 없음
- Phase 5 출시 준비: 0%

**완료 (이 세션 이어서, 2026-05-16)**:
- ✅ 성적리포트 배지 전멸 fix (_srLoadTestMeta in쿼리 → getDoc, Rules 통과)
- ✅ documentId in / academyId 제약 없는 쿼리 수평전개 (유일 사례 확정)
- ✅ 삭제 시험 vs 레거시 안내 문구 분기
- ✅ scores 에 speaking/grammar 메타 보존 (시험 삭제돼도 배지 — 신규 응시분)
- ✅ 결제관리 입금 체크 즉시 풀리던 버그 (refetch:false + 캐시)
- ✅ attemptLabel Rules 충돌 fix (academyId 추가 + 인덱스 교체·옛것 정리)
- ✅ 작업 규칙 — 신규코드 규칙 적용 / 인덱스 grep 판별·prefix / 토글 refetch:false / 삭제·레거시 분기

---

## 2026-05-17: Gemini 403/503 진단 (앱 정상) + preview 모델 GA 교체

코드 변경은 모델 교체 1건(commit `ffe537d`). 나머지는 진단·메모리.

### 1) Gemini 403/503 오류 진단 — 우리 앱 정상 확인

학원장 "Google AI Studio 에 403 매일 꾸준" 보고 → 전수 진단:

- **403 다수의 정체 = `ModelService.ListModels` (v1·v1beta)** — 우리 코드
  `ListModels` 호출 **0건**(grep 확인). 6개 API 모두 특정 모델명으로
  `:generateContent` 직접 호출(목록 조회 안 함). ListModels 403 은 Google
  AI Studio 웹 접속·외부 부산물 → **학생·학원장 앱 무관 노이즈**.
- **우리 앱(`GenerativeService.GenerateContent`)** = Vercel 3일 로그에
  `[check-word] ... retryable fail: 503 high demand` **1건뿐**.
  `all models failed` 류 **0건** → 그 503 도 폴백(재시도/2.5-flash)으로
  흡수 = 학생 실제 영향 0. 작업규칙8 정상 작동.
- 결론: **앱 Gemini 호출 건강**. Cloud Console 오류율 그래프가 높아
  보이는 건 ListModels 노이즈 포함 — GenerateContent 만 필터하면 거의 200.
- IP 제한 아님(API 키 애플리케이션 제한 없음 확인). 코드 조치 불필요.

### 2) preview 모델 GA 교체 (commit `ffe537d`)

`gemini-3.1-flash-lite-preview` 2026-05-25 종료 안내 → 후속 정식
`gemini-3.1-flash-lite` 로 **6곳 일괄 교체** (폴백 3순위 그대로 유지,
작업규칙8 동일 순서). 잔존 preview 0 + 6파일 `node -c` 통과.
- `api/generate-quiz.js`·`check-recording.js`·`cleanup-ocr.js`·
  `growth-report.js`·`scoresnap-grade.js`(+주석) + `scripts/admin/recover-recording-errors.js`
- `check-word.js` 는 원래 2모델(2.5-flash-lite/2.5-flash) — preview 미사용·영향 없음
- 클라(public/) 무변경 → SW bump 불필요 (api 서버리스는 Vercel 배포 즉시 반영)

### 3) 가격 비교 + 폴백 2순위 재배치 (다음 달 보류)

WebFetch 로 ai.google.dev 가격 확인 (per 1M tokens, 유료):

| 모델 | input | audio | output |
|------|-------|-------|--------|
| 2.5-flash-lite (현 1순위) | $0.10 | $0.30 | $0.40 |
| 2.5-flash (현 2순위) | $0.30 | $1.00 | $2.50 |
| 3.1-flash-lite (현 3순위) | $0.25 | $0.50 | $1.50 |

- 사용자 추정("RPM 차이뿐")과 달리 **단가 차이 큼**. 3.1 main 전환은
  비용 2.5~3.75배 → 부적절(현행 유지가 비용 최적)
- 그러나 **3.1-flash-lite < 2.5-flash 전 항목 저렴 + 신모델** → 2순위를
  2.5-flash → 3.1-flash-lite 로 바꾸는 옵션 B 합리적. 단 GA 직후 안정성
  우려로 **5/25 종료 후 1~2주 확인 후** 진행 — 메모리
  [project_gemini_fallback_reorder.md](memory/project_gemini_fallback_reorder.md) 등록

---

## 작업 규칙 추가 (2026-05-17)

신규:
- **Gemini 오류율 진단 시 메서드 구분 필수** — `ModelService.ListModels`
  (v1·v1beta) 403/503 은 우리 코드 0건 호출(특정 모델명으로
  generateContent 직접). Google AI Studio 웹·외부 부산물이라 앱 무관
  노이즈. 추적 대상은 **`GenerativeService.GenerateContent` 뿐**. Cloud
  Console 오류율이 높아 보여도 메서드 필터하면 GenerateContent 는 거의 200.
- **Vercel Runtime Log 는 휘발성 — 매일 발생 추적엔 부적합** — 과거 누적
  검색 약함. cold-start 마다 뜨는 DEP0169 만 보이고 드문 에러는 안 보일 수
  있음. API별 에러 로그 prefix 다름(`[check-word]`/`[check-recording]`/
  `Gemini ${model} error:`) — "Gemini" 단일 검색은 누락. 누적 추적은 Cloud
  Console 측정항목(메서드·응답코드별)이 정확.
- **외부 서비스 가격·모델 spec 은 WebFetch 로 공식 확인** — knowledge
  cutoff 후 자주 변동(특히 GA 직후 모델). 추측 단정 X
  ([[feedback_confirm_specs_before_work]]). ai.google.dev/gemini-api/docs/pricing.

---

## 파일 크기 / SW 캐시 (2026-05-17)
- `api/*.js` 5개 + `scripts/admin/recover-recording-errors.js`: 모델명 1줄씩 교체
- 클라·SW 무변경 (SW 캐시 `kunsori-v540` 유지)

## 진행률 (2026-05-17)
- Gemini 인프라: ~100% (앱 호출 건강 확인, preview→GA 교체로 5/25 종료 대비)
- 성적리포트·결제·말하기·멀티테넌시: 변동 없음
- Phase 5 출시 준비: 0%

**완료 (이 세션, 2026-05-17)**:
- ✅ Gemini 403/503 전수 진단 — 앱 정상(403=ListModels 노이즈, 503=폴백 흡수)
- ✅ preview→GA 모델 교체 6곳 (5/25 종료 대비, 폴백 3순위 유지)
- ✅ 가격 비교 (WebFetch) + 폴백 2순위 재배치 메모리 등록 (다음 달 보류)
- ✅ 작업 규칙 — ListModels 노이즈 / Vercel 로그 휘발성 / 외부 가격 WebFetch 확인

---

## 2026-05-18: AI Generator 언스크램블 문장 직접 입력 (Wordsnap 패턴)

SW v540 → v541 (1 commit `b4516a4`). 단어시험 Wordsnap 처럼 언스크램블에도 본문 Page 선택 없이 영문장 직접 입력 → 청크 분할 + 한글뜻 자동 생성.

### 확정 spec (사용자 결정)
- 입력 형식: textarea **한 줄 = 1 영문장** (한글뜻 AI 자동 생성)
- 청크 분할: AI 호출 (원문 100% 보존 + 자연 청크 경계)
- 결과: 미리보기 모달 → 저장 (기존 언스크램블 흐름)
- "입력 내용 따라 다른 룰" = 줄바꿈 구분 영문장 리스트 형태 자동 처리

### 서버 ([api/generate-quiz.js](api/generate-quiz.js))
- 새 mode `'unscramble-from-text'` 분기 (homophones-only 패턴 동일 구조)
- `UNSCRAMBLE_FROM_TEXT_PROMPT` — VERBATIM 보존 강제, 청크 N±1, meaningKo 생성
- `handleUnscrambleFromText`:
  - sentences[] 검증 (3~400자, 최대 100문장, 중복 제거)
  - GEMINI_MODELS 폴백 체인 (작업규칙 8)
  - **원문 보존 검증** — chunkedSentence 의 `/` 제거 후 정규화(공백·대소문자) 비교
  - 변형 감지 시 **원문 단어 단위 N등분 강제** (AI 가 변형해도 원문 100% 보존)
  - 청크 누락 시도 동일 fallback

### 클라 ([public/admin/js/app.js](public/admin/js/app.js))
- `_qgBuildUnscrambleSnapSection` — UI (textarea + 📥붙여넣기 + 실행, Wordsnap 동일 디자인)
- `_qgParseSentences` — 줄당 1문장 (빈 줄 스킵, 중복 제거, 3~400자)
- `_qgUnscrambleSnapUpdateStatus` / `qgUnscrambleSnapPaste` / `qgRunUnscrambleSnap`
- type 분기 — 기존 `word→_qgBuildWordsnapSection` 에 `unscramble→_qgBuildUnscrambleSnapSection` 추가
- chunkCount 는 언스크램블 옵션 (`_qgCollectOpts('unscramble').chunkCount`) 값 사용
- `qgRunUnscrambleSnap` → `_qgShowResultModal` (기존 미리보기 흐름 재사용)

### qgSaveSet Book fallback (중요)
- 기존 `qgSaveSet` 의 sourcePages 는 `q.sourcePageId` 기반 — 직접 입력은 sourcePageId='' → bookId 빈값 → **미지정 폴더 저장 위험**
- fix: sourcePages 가 전부 빈 bookId/pageId 면 `_qgActiveBook` 단일 엔트리 생성 (Wordsnap qgRunWordsnap 패턴과 동일)
- 직접 입력 언스크램블 세트도 활성 Book 폴더에 저장됨

### 작업 규칙 추가
- **AI 직접 입력 원문 보존 패턴** — Wordsnap(단어) / 언스크램블(문장) 처럼 사용자 입력을 AI 가 가공할 때, AI 가 원문 변형하면 서버에서 검증(정규화 비교) 후 원문 기반 강제 재구성. AI 응답 신뢰 X, 입력이 ground truth.
- **AI Generator 직접 입력 = qgSaveSet Book fallback 필수** — sourcePageId 없는 직접 입력 세트는 sourcePages 비어 미지정 폴더로 빠짐. `_qgActiveBook` fallback 으로 활성 폴더 연결.

## 파일 크기 / SW 캐시 (2026-05-18)
- `api/generate-quiz.js`: +~130줄 (UNSCRAMBLE_FROM_TEXT_PROMPT + handleUnscrambleFromText)
- `public/admin/js/app.js`: +~150줄 (언스크램블 직접 입력 UI·파서·실행 + qgSaveSet fallback)
- SW 캐시: `kunsori-v541`

## 진행률 (2026-05-18)
- **AI Generator 직접 입력: ~100%** (단어시험 Wordsnap + 언스크램블 문장 입력)
- 음성 인식·동음이의어·AI 사용량·멀티테넌시: 변동 없음
- Phase 5 출시 준비: 0%

**완료 (이 세션, 2026-05-18)**:
- ✅ 언스크램블 문장 직접 입력 (mode 'unscramble-from-text' + UI + 원문 보존 검증)
- ✅ qgSaveSet Book fallback (직접 입력 세트 미지정 폴더 방지)
- ✅ 작업 규칙 — AI 직접 입력 원문 보존 / qgSaveSet Book fallback

---

## 2026-05-18 (이어서): AI Generator/OCR Book 클릭 race + Chapter 이동 모달 재구성

학원장 신고 "AI Generator/OCR 에서 Book 클릭해도 Chapter/Page 종종 안 뜸"
→ lazy fetch race 진단 → 이동 모달 UX 종합 재구성. SW v540 → v546 (~5 commit).

### 1) Book 클릭 lazy fetch race + 조용한 실패 fix (commit `0756d82`)

증상: AI Generator(`qgSelectBook`)/AI OCR(`genClickBook`) 에서 Book 클릭
시 Chapter·Page 목록 종종 안 뜸.

원인 (둘 동일 패턴):
- **race**: async lazy fetch 중 다른 Book 클릭 → 늦게 온 옛 응답이
  엉뚱한 시점 concat+render → 빈 목록 (빠른/연속 클릭 시)
- **catch 조용히 삼킴**: `catch(e){console.warn}` — fetch 일시 실패 시
  빈 화면, 사용자 에러 인지 0

수정: 공용 `_genBookFetchToken` 세대 가드 — `++tk` 후 fetch, 완료 시
`tk !== 현재토큰`이면 return (최신 클릭이 render 담당). catch →
`console.error` + 토스트 + 옛 응답 에러 무시. 인덱스·Rules 정상 확인
(genChapters/genPages `academyId+bookId+order/serialNumber` 존재,
where academyId 동반 → Rules 통과 — _srLoadTestMeta 함정 아님).

### 2) Chapter 이동 모달 Book→Chapter 2단 + inline 생성 (commit `38538a4`·`987880a`·`9bd58f1`)

문제: lazy 라 Book 안 고르면 `_genChapters` 비어 "Chapter 없음" alert
→ 사용자가 딴 화면서 **중복 Chapter 생성**. 전체 chapter 노출은 학원
커지면 긴 목록·reads 증가로 비효율(사용자 우려).

데이터 근거: default Book 10 / Chapter 29 / Page 156 — Page 가 대용량.

해결 (사용자 제안 — 상위 선택→그 하위만 lazy, 전체 fetch X):
- **`genMovePages` 2단**: ① Book 선택(`_genBooks`) → ② 그 Book chapter
  lazy fetch(where bookId, race 가드) 목록
- 양 단계 항상 **inline 새 생성** ([+ 새 Book], [+ 이 Book 에 새 Chapter])
  → 이름 입력(Enter) → addDoc(bookId/bookName 자동) → 생성+즉시 이동
  (별도 화면 X, 중복·미지정 차단). `_genDoMove` 공용 헬퍼
- **`genMoveChapters`**(Chapter→Book) 도 inline 새 Book (생성 즉시 그
  Book 으로 Chapter 이동 — 1단)
- "없음" alert 차단 → 안내 + 생성 버튼. onclick 인라인 따옴표 →
  `data-*` 속성 패턴 (특수문자 안전)

효과: lazy 유지(학원 커져도 목록·reads 일정 — 항상 한 Book chapter),
중복 생성 근본 차단, 초기 사용자 직관(모달이 Book→Chapter 흐름 안내).

### 3) 이동 모달 리스트 최근순 통일 (commit `3141a86`)

genMovePages ① Book(이름순)·② Chapter(order순) → `_genRecentSort`
(updatedAt/createdAt 최근순). genMoveChapters Book 은 이미 최근순.
3곳 통일 → 방금 작업·생성 항목이 맨 위 (스크롤 불필요).

---

## 작업 규칙 추가 (2026-05-18 이어서)

신규:
- **async lazy fetch 는 세대 토큰 race 가드 필수** — Book 클릭처럼
  사용자가 빠르게 연속 트리거하는 async fetch 는 `const tk=++_token`
  후 fetch, 완료 시 `tk !== _token` 이면 return(최신 트리거가 render).
  옛 응답이 늦게 와 엉뚱한 render → 빈 목록 방지. catch 도 옛 토큰이면
  무시 + 현재 토큰일 때만 사용자 토스트.
- **lazy fetch ↔ UX 충돌은 "상위 선택 → 그 하위만 lazy + inline 생성"
  으로** — 전체 fetch(학원 커지면 긴 목록·reads↑) 도 아니고 lazy
  방치(안 보여 중복 생성) 도 아닌, 모달에서 상위(Book) 고르면 그
  하위(Chapter) 만 lazy + 없으면 그 자리 생성. 확장성·중복 차단 동시.
- **모달 내 동적 단계 전환** — showModal 정적 HTML + `<div id=...>` 영역
  innerHTML 교체(`_genMoveRefresh`)로 2단 흐름. 상태는 모듈 변수
  (`_genMoveBook`). window 함수로 onclick 노출.

---

## 파일 크기 / SW 캐시 (2026-05-18 이어서)
- `public/admin/js/app.js`: +~120줄 (race 가드 + Chapter 이동 모달 재구성·inline 생성)
- SW 캐시: `kunsori-v546`

## 진행률 (2026-05-18 이어서)
- AI Generator/OCR 안정성: **~100%** (Book 클릭 race fix + 이동 모달 종합 재구성)
- AI Generator 직접 입력: ~100% (변동 없음)
- Gemini 인프라·성적리포트·결제·말하기: 변동 없음
- Phase 5 출시 준비: 0%

**완료 (이 세션 이어서, 2026-05-18)**:
- ✅ Book 클릭 lazy fetch race + 조용한 실패 fix (AI Generator·OCR)
- ✅ Chapter 이동 모달 Book→Chapter 2단 + 양 단계 inline 새 Book/Chapter 생성·즉시이동
- ✅ Chapter→Book 이동 모달 inline 새 Book
- ✅ 이동 모달 리스트 최근순 통일 (방금 작업 항목 우선)
- ✅ 작업 규칙 — async lazy race 가드 / lazy↔UX 충돌 해법 / 모달 내 동적 단계

---

## 2026-05-18 (이어서 2): 헤더 Version 표시 + 새로고침 버튼 7곳 토스트 피드백

학원장 "새로고침 눌러도 반응 없어 됐는지 모름" + "버전 보이게". SW v546 → v549 (~4 commit).

### 1) 헤더 Version 표시 (commit `8cb162f`·`565b89c`)

학원장 캐시 갱신 자가진단용 — 강력 새로고침 후 숫자 바뀌면 최신.
- `sw.js` message: `GET_VERSION` → MessageChannel 로 `CACHE_NAME` 회신
- `_app.html`: `#appVer` span — **우측 학원장 이름(`#adminName`) 앞**
  (default 학원장 계정명이 '큰소리영어'. 학원명은 좌측 로고 옆 별개)
- `admin app.js _showAppVersion()`: SW 질의 → `kunsori-v549` →
  `"Version 5.4.9"` (뒤1=patch / 그앞1=minor / 나머지=major).
  onAuthStateChanged 후 fire-and-forget
- SW 자체값 질의 → 클라 상수 어긋남 없이 정확

### 2) 새로고침 버튼 7곳 — 토스트 + 차등 캐시 무효화 (commit `7a8d40a`·`81d4723`)

증상: AI OCR/AI Generator/문제세트목록 등 ↺ 새로고침 눌러도 무반응.
+ 일부는 캐시 가드로 클릭해도 실제 갱신 안 됨 → 토스트만 달면 거짓 피드백.

**진단 후 차등 적용** (진입=기존 함수 직접·캐시활용 / 새로고침만 wrapper):

| 화면 | wrapper | 캐시 처리 (진단 근거) |
|------|---------|----------------------|
| AI OCR `genRefresh` | loadGenerator() | 항상 재fetch |
| AI Generator `qgRefresh` | `_genBooks/Chapters/Pages=[]` → loadQuizGenerate() | `_genBooks` 캐시 skip 이라 무효화 필요 |
| 문제세트목록 `qsRefresh` | `_qsInvalidateCache()` → loadQuestionSets() | 세트 캐시 `__initialized` 유지라 무효화 필요 |
| 진도체크 `progRefresh` | `delete _prog.testsByDate[date]` → progRenderByDate | 그 날짜 캐시 hit 라 무효화 필요 |
| 시험배정 `tpAssignRefresh` | _renderTestAssignDetail() | 매번 재fetch → 토스트만 |
| 대시보드 `dashRefresh` | initDashboard() | 학생수 getCount·시험·AI사용량·달력·공지 매번 fresh, 미납만 결제캐시(자동무효) → **토스트만, 광범위 무효화 X** (무효화 시 reads 폭증) |
| AI 사용량 `quotaRefresh` | loadQuotaUsage() | 매번 getDoc fresh → 토스트만 |

모두 "새로고침 중..." → "✅ 완료" 2단. _app.html 6곳 + app.js 1곳 onclick → wrapper.

핵심: 캐시 가드로 갱신 안 되던 곳만 무효화(거짓 피드백 방지), 이미
fresh 한 곳은 토스트만(불필요 reads 안 늘림 — 학원장 reads 정책).

---

## 작업 규칙 추가 (2026-05-18 이어서 2)

신규:
- **새로고침 버튼 = 진입 함수 그대로 쓰면 거짓 피드백 위험** — 캐시
  가드(lazy `__initialized` / `if(!_genBooks.length)` / `testsByDate[date]`)
  있는 함수는 새로고침 클릭해도 skip. 새로고침 전용 wrapper 가 해당
  캐시 무효화 후 재fetch + 토스트. **단 진단 먼저** — 이미 매번 fresh
  한 함수(getCount/매getDoc)는 무효화 불필요(reads 낭비), 토스트만.
- **대시보드류 복합 캐시는 광범위 무효화 금지** — 위젯별 캐시 정책
  상이(getCount 매번 / 결제 _billingsByMonth 캐시·자동무효 / 달력 매번).
  새로고침에서 통째 무효화하면 결제 fetch 폭증. 자체 무효(데이터 변경
  시) 에 맡기고 새로고침은 함수 재실행(거의 fresh) + 토스트만.
- **SW 버전 클라 노출 = SW 질의(MessageChannel)** — 클라 상수는 sw.js
  와 어긋남. `GET_VERSION` postMessage → `event.ports[0].postMessage
  (CACHE_NAME)`. 디버깅 목적엔 SW 실제값이어야 의미.

---

## 파일 크기 / SW 캐시 (2026-05-18 이어서 2)
- `public/admin/js/app.js`: +~60줄 (_showAppVersion + 새로고침 wrapper 7개)
- `public/admin/_app.html`: #appVer span + 새로고침 onclick 6곳 교체
- `public/sw.js`: GET_VERSION 핸들러
- SW 캐시: `kunsori-v549`

## 진행률 (2026-05-18 이어서 2)
- AI Generator/OCR 안정성: ~100% (변동 없음)
- 학원장 앱 UX 피드백: **~100%** (헤더 Version + 새로고침 7곳 토스트·차등 갱신)
- Gemini·성적리포트·결제·말하기: 변동 없음
- Phase 5 출시 준비: 0%

**완료 (이 세션 이어서 2, 2026-05-18)**:
- ✅ 헤더 Version 표시 (우측 학원장 이름 앞, SW 질의 → "Version 5.4.9")
- ✅ 새로고침 버튼 7곳 — "중...→✅완료" 토스트 + 진단 근거 차등 캐시 무효화
- ✅ 작업 규칙 — 새로고침 거짓 피드백 방지 / 복합 캐시 광범위 무효화 금지 / SW 버전 질의

---

## 2026-05-18 (이어서 3): 단어시험 출제 형식 옵션 인쇄 모달과 통일 (B안)

학원장이 단어시험 **출제(배정) 형식 옵션에 혼란** 보고 — 출제 모달은
형식+방향+비율 **3축**, 인쇄 모달은 형식+슬라이더 **2개**라 멘탈 모델
불일치. 검토 후 사용자 B안 확정. SW v549 → v552 (3 commit).

### 1) 통일 설계 (B안)

출제(배정) 모달을 인쇄 모달과 동일 UX 로:
- **형식**: 혼합(랜덤) / 혼합(객→주) / 혼합(주→객) / 말하기(음성 인식)
- **객관식비율** 슬라이더 (0~100, 10단위) — 0%=전체 주관식 / 100%=전체 객관식
- **영→한비율** 슬라이더 — 0%=전체 한→영 / 100%=전체 영→한
- 말하기 선택 시 두 슬라이더 자동 비활성 (한글→영어 발음 고정)
- 독립 "주관식(스펠링)"/"객관식"/"방향 드롭다운" **제거** (슬라이더
  0%/100% 로 흡수 — 인쇄 모달이 이미 한 단순화)

**객→주 / 주→객 동작** (사용자 확인): 객·주 선택 자체는 비율 기반
**랜덤 유지**, 결정되는 건 표시 순서뿐. 객관식 배정 묶음 먼저(내부
셔플 순서 유지) → 주관식 묶음 나중. 주→객은 반대.

### 2) 구현 (commit `fd2a865`)

- **출제 모달 UI** (`tpOpenPublishModal`): 형식 select 4개 + 슬라이더
  2개. `_tpVocabFormatChanged` 가 말하기 시 `tpVocabRatioRow` 비활성
- **`tpPublish` vocabOptions**: `direction` 제거, `en2koRatio`(0~100)
  추가. `isFinite` 패턴(0 함정 회피). `mcqRatio` 도 슬라이더값
- **학생앱 `startVocab`**: 객·주 선택은 비율 랜덤 유지 +
  `mixed_mcq_first`/`mixed_short_first` 는 배정 후 그룹 정렬(안정
  정렬로 같은 그룹 내부 셔플 순서 유지, **questions·answers 인덱스
  동기**). speaking → 전부 speaking
- **하위호환** (기존 genTests 무변경): 학생앱이 폴백 매핑
  · 옛 format `short`→mcqRatio 0 / `mcq`→100 / `mixed`·`speaking` 그대로
  · 옛 `direction` `en2ko`→en2koRatio 100 / `ko2en`→0 / `mixed`·미설정→50
  · 이미 배정된 시험도 새 학생앱에서 정상 동작

### 3) 컴팩트 한 줄 배치 (commit `48a8f4a` → `cbb5829`)

사용자 요청 — 형식 select 와 슬라이더가 다른 줄로 wrap 되던 문제.
시험출제 모달 폭 **640 → 720px** 확대 + 형식 행 `flex-wrap:nowrap`
+ 슬라이더 110→100px + 라벨 `white-space:nowrap` →
`형식: [▾]  객관식비율: [━] 50%  영→한비율: [━] 50%` 한 줄 고정
(좁은 화면은 모달 94vw 축소 안에서 한 줄 유지). `tpVocabRatioRow`
는 `<span>` 으로 래핑(말하기 시 슬라이더만 비활성, 동작 동일).

---

## 작업 규칙 추가 (2026-05-18 이어서 3)

신규:
- **출제 모달 ↔ 인쇄 모달 동일 도메인 옵션은 UX 통일** — 같은
  개념(단어시험 형식/비율)을 두 화면이 다른 입력 방식(드롭다운 3축
  vs 슬라이더 2개)으로 노출하면 학원장 혼란. 슬라이더 비율 모델로
  통일. 학생앱은 비율 기반 랜덤 배정이라 슬라이더와 자연 호환.
- **데이터 모델 변경 시 학생앱 하위호환 폴백 필수** — vocabOptions
  처럼 이미 배정된 genTests 에 옛 필드(format='short'/'mcq',
  direction) 가 박혀있음. 마이그레이션 대신 학생앱 읽는 쪽에서
  신필드 우선 → 없으면 옛 필드 매핑. 기존 시험 무변경·즉시 호환.
- **순서 정렬 시 questions·answers 인덱스 동기** — `_vqState`
  의 answers[i] ↔ questions[i] 대응. 그룹 정렬 시 둘 다 같은
  order 로 재배열(한쪽만 reorder 하면 상세·채점 어긋남). 같은 그룹
  내부는 0 반환 안정 정렬로 셔플 순서 유지.

---

## 파일 크기 / SW 캐시 (2026-05-18 이어서 3)
- `public/admin/js/app.js`: ~동일 (옵션 UI 교체 + tpPublish + 모달 폭)
- `public/js/app.js`: +~20줄 (startVocab 신모델 파싱·하위호환·그룹 정렬)
- `public/sw.js`: v549 → v552
- SW 캐시: `kunsori-v552`

## 진행률 (2026-05-18 이어서 3)
- **단어시험 출제 옵션: ~100%** (인쇄 모달과 통일, 학원장 혼란 해소)
- 학원장 앱 UX 피드백: ~100% (변동 없음)
- Gemini·성적리포트·결제·말하기·AI Generator: 변동 없음
- Phase 5 출시 준비: 0%

**완료 (이 세션 이어서 3, 2026-05-18)**:
- ✅ 단어시험 출제 형식 옵션 인쇄 모달과 통일 (B안 — 형식 4개 + 슬라이더 2개)
- ✅ 학생앱 startVocab 신모델 + 객→주/주→객 그룹 정렬 + 옛 데이터 하위호환
- ✅ 형식+슬라이더 한 줄 컴팩트 배치 (모달 폭 720, nowrap)
- ✅ 작업 규칙 — 출제↔인쇄 UX 통일 / 데이터 모델 변경 하위호환 / 인덱스 동기 정렬

---

## 2026-05-18 (이어서 4): 녹음숙제 상세 통일·최소화 + 말하기 3차 hang + check-word 503 대응 + Gemini 폴백 재배치

학원장 보고(녹음숙제 상세 경로마다 정보 다름) → 진단 → 통일·최소화.
이어 말하기 3차 먹통 진단·수정, 503 급증 대응, 폴백 재배치까지.
SW v549 → v558 (api 재배치는 SW 무관). 다수 commit.

### 1) 녹음숙제 학원장 상세 — 단일 공유 빌더로 통일 (`62c1c54`)

진단: 학원장이 보는 3경로가 각각 별도 렌더라 정보량 제각각 —
#1 시험관리/시험목록/진도체크학생별 풀카드(시간·말소리%·속도 O, note X),
#2 일자별 한 줄(요약), #3 성적상세모달(`_adminRecBuildDetail`, 시간만·
말소리%/속도 X). 학생앱 `_rv2RenderResult`(#4)는 점수 비공개 정책 준수 확인.

- `_adminRecBuildDetail(recordings, fullText, opts)` 단일 공유 빌더로
  통일 — 회차별 시간·말소리%·속도(WPM)·점수·note·AI피드백 모두 포함
- WPM용 fullText: `showScoreDetail` 이 `genTest.questions[0].fullText`
  → `comp._recFullText` 전달 / #1 은 `tqFullText` 전달
- `clickSafe` 옵션 — #1 카드는 부모 onclick(모달) 충돌 방지 stopPropagation
- #1 인라인 회차·AI피드백 코드 제거 → 공유 빌더 호출
- 부수: voiceActivity/duration 미보존 회차(재평가 등)는 "- · 말소리 -"
  로 3경로 동일 표시 (누락 투명화)

### 2) 진도체크·최근시험 녹음 카드 전체 최소화 (`8ac4729`)

사용자 결정(한 줄 유지+클릭 시 동일 모달): `isSubmittedWithRecs` 풀카드
폐기 — `opts.simpleRec` 조건 제거하고 **모든 녹음 제출 카드 한 줄**
(이름·📤제출됨·N점·날짜), 클릭 → #3 공유 모달. 다른 시험 유형 카드와
시각 통일. 옛 데이터·에러 카드는 이미 한 줄이라 변동 없음.

### 3) 성적 상세 모달 [🔁 재평가] 버튼 (`3f4b11e`)

한 줄 최소화로 카드의 재평가 진입점 소실 → showScoreDetail 풋터에
[🔁 재평가] 추가 (recording + recordings 있을 때만). 풋터
space-between (재평가 좌 / 닫기 우). `tpReEvaluateRecording` 호출.

### 4) 단어 말하기 3차 먹통 (안드로이드 SR→MR 핸드오프) — A·B

진단: 1·2차 SpeechRecognition(안드로이드 클라우드, 마이크 점유) →
3차 `getUserMedia`(MediaRecorder) 전환 시 마이크 해제가 150ms 안에
안 끝나 **getUserMedia 가 응답·reject 없이 hang** → 버튼 disabled +
busy=true + attempt=MAX → 영구 먹통·복구 불가. 안드로이드 고유(iOS 무관).

- **A안 (`a91ba7b`)**: `_gumWithTimeout` — getUserMedia 4초 타임아웃
  (늦게 도착 stream track stop). hang 시 busy 해제·버튼 복구·attempt
  롤백(재시도 허용)·finalize 안 함(잠그지 않음)
- **B안 (`a50f7dd`)**: 2차+ SR 마이크 해제 대기 150ms→400ms (hang 예방).
  SR 은 stream 미노출이라 abort()+대기로만 보장

### 5) check-word 503 대응 — 타임아웃 9초·B-1 (`75cab80`)

503 급증(Vercel 로그 30분 26건)으로 단어말하기 AI 폴백이 5초 초과 →
서버는 200 성공인데 클라가 5초에 포기 → **억울한 오답 다발**(로그 16초
케이스 확인). 단어말하기 5초 / 녹음숙제 30초 차이로 단어말하기만 취약.

- check-word fetch 타임아웃 5000ms → **9000ms**
- 5초 경과 시 진행 메시지 "AI 응답이 늦어지고 있어요" 추가 (3단계)
- 9초 초과(AbortError) → 오답 처리 X. `_vqSpkAllowRetry`: busy 해제
  + attempt 롤백 + aiTried 리셋 + blob 폐기 → "다시 시도" + 재녹음 가능
  (B-1, getUserMedia hang 복구 A안과 동일 패턴)
- AI 재배치는 이때 보류 결정 → 진단 후 §7 에서 실행

### 6) AI 의존도 진단 스크립트 신규

`scripts/diag/analyze-speaking-ai-dependence.js` — 말하기 답안 중
`spkSource`(webspeech/ai/ai-error) 비율 + `spkAttempts` 분포, 기간
필터(`--days` / `--from`/`--to` KST), `--academy`/`--top`. 데이터 한계:
userCompleted 통과 응시만(작업규칙7), 타임아웃 B-1 케이스 미기록.

측정 결과:
- 14일: AI 도달 11%(미상 48% 희석) / 3일: **AI 도달 21.6%**(미상 2.2%
  깨끗) / 당일: **22.1% + AI 서버오류 3→5% 상승**
- 결론: AI 의존도 ~22% 높은 편 + 503 악화 추세 → 폴백 재배치 실행 근거

### 7) Gemini 폴백 2순위 재배치 (`39ef11f`) — 보류 → 실행

사용자 결정("모두 해놓고 결과 판단"). 메모리
[project_gemini_fallback_reorder.md](memory/project_gemini_fallback_reorder.md)
옵션 B 를 503 급증 + 진단 근거로 앞당겨 실행:

- `2.5-flash-lite → 2.5-flash → 3.1-flash-lite`
  ⇒ **`2.5-flash-lite → 3.1-flash-lite → 2.5-flash`**
  (generate-quiz/check-recording/cleanup-ocr/growth-report +
  recover-recording-errors)
- check-word(2모델): `2.5-flash-lite → 2.5-flash`
  ⇒ `2.5-flash-lite → 3.1-flash-lite` (속도·비용 민감)
- scoresnap-grade(Vision): 1순위 2.5-flash 유지, 2·3 재배치
  ⇒ `2.5-flash → 3.1-flash-lite → 2.5-flash-lite`
- 근거: 3.1-flash-lite 가 2.5-flash 보다 전 항목 저렴+빠름. 2.5-flash
  3순위 강등(audio 비용 큼, 1·2 동시 장애 시만). api 전용 → Vercel
  즉시 반영, SW bump 불필요. 코드 주석 갱신(작업규칙8 본문은 차기 정리)

---

## 작업 규칙 추가 (2026-05-18 이어서 4)

신규:
- **여러 화면이 같은 데이터를 별도 렌더하면 단일 공유 빌더로** — 녹음
  상세 3경로(#1 카드/#3 모달)처럼 정보량 드리프트 발생. 한 빌더 +
  옵션(clickSafe 등)으로 통일. 요약 카드(#2)는 클릭→공유 모달로 일관.
- **안드로이드 SR→MediaRecorder 핸드오프 = getUserMedia hang 위험** —
  SpeechRecognition(안드로이드 클라우드)이 마이크 점유, 해제 비동기·느림.
  전환 전 abort()+충분한 대기(≥400ms) + getUserMedia 타임아웃 래퍼 필수.
  iOS(온디바이스 SR)는 무관 — 플랫폼별 별개 이슈 혼동 주의.
- **latency-critical 클라 타임아웃은 초과 시 벌점 금지** — check-word
  5초처럼 짧은 타임아웃은 503 폴백 초과 시 서버 성공분도 버려 억울한
  오답. 타임아웃 = "여기까지 기다림" 일 뿐, 정답/오답 가르는 선 X →
  초과 시 재시도(B-1)로 복구. 숫자 키우기보다 벌점 제거가 핵심.
- **503 = 구글측 모델 용량(transient), 우리가 못 고침** — 대응은
  재시도·폴백·타임아웃·벌점제거. 모델별 절대 빈도는 Vercel 로그/Cloud
  Console 이 정확(Firestore 는 통과분만이라 과소). 의존도 추세는 진단
  스크립트로 며칠 관찰 후 재배치 등 결정.
- **api 전용 변경은 SW bump 불필요** — Vercel 서버리스 즉시 반영.
  클라(public/) 무변경 시 SW 캐시 버전 안 올림 (5/17 preview→GA 선례).

---

## 파일 크기 / SW 캐시 (2026-05-18 이어서 4)
- `public/admin/js/app.js`: 녹음 상세 통일(공유 빌더)·최소화·재평가 버튼 (~-30 순감)
- `public/js/app.js`: 말하기 3차 hang A·B + check-word 9초·B-1 (+~50)
- `api/*` 6개 + `scripts/admin/recover-recording-errors.js`: 폴백 체인 재배치
- `scripts/diag/analyze-speaking-ai-dependence.js`: 신규 ~150줄
- SW 캐시: `kunsori-v558` (api 재배치는 SW 무관)

## 진행률 (2026-05-18 이어서 4)
- 녹음숙제 학원장 상세: **~100%** (3경로 단일 빌더 통일·전체 최소화·재평가 모달)
- 단어 말하기 안정성: **~95%** (3차 hang A·B + check-word 9초·B-1·폴백 재배치. 추세 관찰 중)
- Gemini 폴백: **재배치 완료** (후속 추세 관찰만)
- Phase 5 출시 준비: 0%

**완료 (이 세션 이어서 4, 2026-05-18)**:
- ✅ 녹음숙제 학원장 상세 3경로 단일 공유 빌더 통일 (`62c1c54`)
- ✅ 진도체크·최근시험 녹음 카드 전체 한 줄 최소화 (`8ac4729`)
- ✅ 성적 상세 모달 [🔁 재평가] 버튼 (`3f4b11e`)
- ✅ 말하기 3차 getUserMedia hang — A(타임아웃 복구) `a91ba7b` + B(핸드오프 대기 400ms) `a50f7dd`
- ✅ check-word 타임아웃 5→9초 + 늦음 안내 + 9초 초과 시 재녹음(B-1) `75cab80`
- ✅ AI 의존도 진단 스크립트 신규 + 측정 (3일 AI 도달 21.6%, 서버오류 3→5%)
- ✅ Gemini 폴백 2순위 재배치 (2.5-flash→3.1-flash-lite, 6+1곳) `39ef11f`
- ✅ 메모리 project_gemini_fallback_reorder 완료 갱신 + MEMORY.md 인덱스
- ✅ 작업 규칙 — 공유 빌더 / 안드 SR→MR hang / latency 타임아웃 벌점금지 / 503 본질 / api SW bump 불필요

---

## 2026-05-19: Firestore 색인 최적화 — 작업지시서 검증 후 1/3만 적용

외부 작업지시서 (`firestore-indexes-optimization-tasks.md`, 다른 LLM 이 Firebase 쿼리 통계 보고 작성) 받아 검증·진행. 코드 무수정 (색인 파일만). commit `91c40c4`.

### 지시서 검증 — 3개 진단 중 2개가 코드와 불일치

| 지시서 진단 | 실제 코드 검증 | 결정 |
|------|------|------|
| `(academyId, testId, reEvaluated)` 효율 128.10 | **운영 코드 아님** — `scripts/diag/test-length-vs-scores.js` 진단 스크립트 1회용. genTests 루프에서 시험마다 호출 ("12회 실행"=시험 12개). scores.reEvaluated 는 adminAction.js:199 재평가 시 박힘이나 adminAction 은 add only·쿼리 X | **제외** |
| `(academyId, mode, userName)` 효율 71.70 | **userName where 0건** (grep). 성적 리포트 이름검색은 `_srBuildConstraints` 가 academyId+date+mode 로 fetch (색인 매칭 ✓) 후 클라 측 `scoreSearch` 필터. userName server-side X. Firebase 통계 추론 표기 오류 | **제외** (영원히 미사용) |
| `(academyId, testId)` 효율 9.10 | **운영 실재** — 6619 `tpToggleTestProgress` (시험 진행현황 펼침), 6409 `_tlLoadScoresForTests` (진도체크/시험목록 통계). `academyId==+testId==` (orderBy 없음) 정확 매칭 색인 부재 → Firestore 가 academyId 단일 색인 선택 → 학원 점수 전체 받아 testId 메모리 필터 (664 받아 73 사용) | **적용** |

→ 지시서대로 3개 다 넣었으면 2개는 죽은 색인 (슬롯 낭비 + 혼란). 지시서의 "운영 read 5,000/주" 는 `scripts/diag/` 진단 스크립트 1회 실행분 포함으로 부정확.

### 적용 — scores `(academyId, testId)` 2-field 색인 1개

```json
{ "collectionGroup": "scores", "queryScope": "COLLECTION",
  "fields": [
    { "fieldPath": "academyId", "order": "ASCENDING" },
    { "fieldPath": "testId", "order": "ASCENDING" }
  ]
}
```
- 기존 색인 #1 (`academyId+testId+uid+createdAt`, 4-field) 그대로 유지 — 5182·13226·5340 (uid 쓰는 쿼리) 가 계속 사용. 추가만, 삭제 X
- 효과: 시험 진행현황 펼침 / 시험별 통계 read **1/9 절감**. 학원 점수 누적 많을수록 격차 ↑
- `firebase deploy --only firestore:indexes` 완료 (46 인덱스, scores 9개)
- 검증 가이드: [docs/firestore-indexes-2026-05-19.md](docs/firestore-indexes-2026-05-19.md) (3일 후 쿼리 통계 재확인)

### pushNotifications (Step 2 — 보고만)
- 4516/4517 `getCountFromServer(academyId+sent)` — 페이지 진입당 2회 COUNT (비용 0, setInterval 아님)
- 168회/7일 = 24/일 = 학원장 대시보드+메시지 진입 빈도. 색인 충분 → 조치 불필요 (지시서 결론과 일치)

### 작업 규칙 재확인 (2026-05-02 TASK-4/10 정립분 강화)
- **외부 작업지시서 = 출발점, 검증 필수** — 청구 내용 vs 실제 코드/데이터 대조 → 불일치 보고 → 확정 후 진행. LLM 생성 지시서는 Firebase 통계 추론 오류·진단 스크립트 혼입·컬렉션 혼동 가능. 임의로 지시서 안 따름
- **사용자가 "현재 상황 고려" 명시 = 이 검증 단계 트리거** — 일반론을 이 프로젝트의 멀티테넌시 구조(academyId 격리·색인 prefix 규칙·scripts/diag 분리)에 대조
- **Firebase 쿼리 통계 ≠ 운영 비용** — `scripts/diag/` 진단 스크립트 실행분도 통계에 집계. 필드명도 색인 추론으로 실제 where 와 다를 수 있음. 통계 → 코드 grep 검증 필수

## 파일 크기 / SW 캐시 (2026-05-19)
- `firestore.indexes.json`: +1 색인 (scores academyId+testId), 총 46개
- `docs/firestore-indexes-2026-05-19.md`: 신규 결과 보고서
- 코드(public/·api/) 무수정 — SW bump 없음

## 진행률 (2026-05-19)
- **Firestore 색인 최적화: ~95%** (운영 실재 비효율 1건 해결. reEvaluated 진단스크립트용은 보류)
- 음성 인식·동음이의어·AI Generator·멀티테넌시: 변동 없음
- Phase 5 출시 준비: 0%

**완료 (이 세션, 2026-05-19)**:
- ✅ 작업지시서 검증 — scores 쿼리 10곳 + pushNotifications 전수 분석
- ✅ scores (academyId, testId) 색인 추가 + 배포 (운영 1/9 절감)
- ✅ 지시서 진단 2건 (reEvaluated/userName) 코드 불일치로 제외
- ✅ 결과 보고서 docs/firestore-indexes-2026-05-19.md
- ✅ 작업 규칙 — 외부 지시서 검증 필수 / Firebase 통계 ≠ 운영 비용

---

## 2026-05-19 (이어서): 단어 말하기 채점 — 닫힌후보 가드 + 발음코드 (인식 불만 대응)

Web Speech 인식 불안정으로 억울한 오답 불만 ↑. STT 도입 검토했으나
**STT 15초 최소과금 → 단어시험엔 Gemini-lite보다 오히려 비쌈**(월 수백$)
으로 폐기. Web Speech는 인식단계 편향 불가 → **채점(인정) 단계 편향**으로 해결.
commit `9f2a03a`, SW v558→v559.

### 적용 (1번 + 3번)

`public/js/app.js _spkGradeAnswer` 재작성:
- **1번 닫힌후보 가드**: 들린 단어를 "이 시험의 모든 단어(allWords)"와 비교.
  정답군 최고유사도(bestG)가 다른 시험단어 최고(bestO)를 마진 이상
  앞설 때만 인정. 강한매칭 `bestG≥임계 & gap≥0.15` / 임계미만 구제
  `bestG≥0.45 & gap≥0.30`. → 무의미·다른시험단어·충돌은 거부
- **3번 발음코드**(metaphone-lite `_spkPcode`): cereal≈serial 등 철자
  달라도 소리 같으면 가드 안 유사도(0.92)로 반영. **단독 통과 불가**
  (cat/cot 등 false positive 억제 — 가드가 최종 관문)
- 정확일치 후보(동음이의어/발음변형)가 **다른 시험단어와 겹치면 후보
  제외** — light/right 같은 충돌 false positive 원천 차단 (검증 핵심)
- 호출부: `_vqState.questions` 단어목록을 allWords 로 전달.
  accentVariants(2번) 인자는 받되 데이터는 후속
- 검증: `scripts/diag/test-spk-grading.js` **14/14 통과** (억울한 오답
  해소 + 엉뚱/다른단어/충돌 전부 차단 — 사용자 핵심 우려 확인)

### 비용 비교 검증 (WebFetch 공식 확인)

- Gemini 25토큰/초, lite 오디오 $0.30/1M·출력 $0.40/1M → check-word
  1회 ≈ $0.0001 (거의 공짜)
- STT 실시간 ~$0.016/분 **15초 최소 올림과금** → 단어 3초도 15초 청구
  → STT 1회 ≈ Gemini 40배. **단어시험엔 STT가 비싼 안** (직관과 반대)
- 결론: 현행(거의 $0) < A(전부 AI ~$15, 503 6~7배) ≪ B(STT ~$수백)
  → 비용·안정성 모두 채점 가드(이번 작업)가 정답

### 후속 (미완)

- **2번 AI 발음변형(accentVariants)**: homophones 생성 파이프에 얹어
  단어별 "한국식 ASR 오인식 변형"(R/L·F/P 등) 생성·저장. 1번 가드 위에서만
  작동(겹치면 제외)이라 안전. 1번 효과 데이터 보고 필요시 추가
- 효과 관찰: `analyze-speaking-ai-dependence.js` / `analyze-speaking-errors.js`

### 작업 규칙 추가 (2026-05-19)

- **인식 불안정은 인식단계 아닌 채점단계에서 편향** — 브라우저 Web Speech는
  phrase hint 불가. "정답이 시험 내 다른 단어보다 확실히 더 닮았는가"
  닫힌후보 비교가 STT phrase-hint 효과를 채점에서 무료·안전하게 모사
- **인정 편향 추가 시 닫힌후보 가드 필수** — 동음이의어·발음변형·발음코드
  어느 것도 단독 통과 X. 다른 시험단어와 겹치는 후보는 제외. false
  positive("엉뚱한 답 정답처리")는 합성 테스트(test-spk-grading.js)로
  배포 전 검증
- **STT 15초 최소과금** — 짧은 단어 음성판정엔 Gemini-lite보다 비쌈.
  "STT가 싸다"는 일반 통념이 단어시험엔 반대 (공식 가격 WebFetch 확인)

---

## 2026-05-19 (이어서 2): 말하기 부적합 단어 출제 게이트 + 1음절 휴리스틱 폐기

roll·up 류 ASR 한계 단어는 1번 가드로도 한계 → 출제(배정) 단계에서
걸러내는 게이트 도입. SW v558→v567 (commit `83239e5`·`a0598f5`).

### 1) 말하기 부적합 단어 배정 전 게이트 (`83239e5`, SW v566)

vocab+speaking 출제 시 [배정하기] 클릭 → 배정 전 부적합 단어 목록 모달
(단어·사유·🗑삭제). 학원장이 삭제하면 `questions` in-place splice 후 진행.
- `api/generate-quiz.js` `mode === 'speaking-unfit-check'` 분기 +
  `SPEAKING_UNFIT_PROMPT` + `handleSpeakingUnfit` (homophones-only 패턴).
  quota/increment 는 상단에서 모든 mode 공통 — **generator 카운터**
- 클라 `_tpSpeakingUnfitReasons`(휴리스틱) + `_tpSpeakingUnfitGate`(모달,
  `_geminiFetch`, `_tpUnfitDel`/`_tpUnfitClose`), `tpPublish` 주입
  (`_fillMissingHomophones` 뒤, vocab+speaking 만). 0개 남으면 배정 차단

### 2) 1음절 휴리스틱 폐기 → AI hardForASR (`a0598f5`, SW v567)

`wild`·`soft`·`feel`·`claim` 등 정상 단음절이 '1음절' 휴리스틱에 과다
표시(roll(나쁨) vs wild(좋음) 구분 불가) → 사용자 결정 "1":
- 휴리스틱 = **`3글자 이하`만** (up·be·go 객관 극단)
- `SPEAKING_UNFIT_PROMPT` 에 `hardForASR` boolean 추가 — 한국 학생 발화
  시 ASR 오인식 위험 높은 음향적 빈약·모호 단어(roll·up·be·err·owe)만
  true, wild·soft·claim = false, **애매하면 false(과다표시 금지)**
- 모달 사유 라벨 `ASR 인식 어려움` (의성어·사전없음과 함께)

### 작업 규칙 추가 (2026-05-19 이어서 2)

- **휴리스틱이 정상/위험을 못 가르면 폐기하고 AI 판단으로** — '1음절'
  처럼 정상 단어(wild)와 위험 단어(roll)가 같은 특징을 공유하면 그
  휴리스틱은 노이즈. 객관적 극단(3글자 이하)만 코드, 미묘한 판단은 AI
  (애매하면 false 로 과다표시 억제 지시 필수)
- **부적합 단어는 채점 보정 아닌 출제 단계 차단** — roll 류 ASR 한계
  단어는 채점 가드(1번)·발음변형(2번)으로도 한계. 출제 시 학원장이
  보고 빼는 게이트가 가장 확실. 채점 편향과 별개 레이어

### 후속 / 보류

- 2번 accentVariants — [memory/project_speaking_accent_variants.md] 등록.
  말하기 인식 불만 재발 시 트리거 (1번+게이트 효과 관찰 후)

## 파일 크기 / SW 캐시 (2026-05-19 이어서 2)
- `api/generate-quiz.js`: +~80줄 (speaking-unfit-check + hardForASR)
- `public/admin/js/app.js`: +~90줄 (게이트 모달·휴리스틱·tpPublish 주입)
- SW 캐시: `kunsori-v567`

---

## 2026-05-19 (이어서 3): 진도체크 학생제외 재조회 0회 + 출제옵션 기본값 변경 + 폴백 진단

SW v567→v569 (commit `bac6bd8`·`376b576`). 운영 점검 + UX/기본값 정비.

### 1) 진도체크 학생 제외 시 재조회 0회 — A-1 (`bac6bd8`, SW v568)

증상: 일자별 반별 진도체크에서 학생 카드 ✕(제외) 누를 때마다
`tpExcludeStudent` 끝의 `tpToggleTestProgress` 2회(닫고 다시 = Firestore
재조회) → 한 명 지울 때마다 그 시험 학생현황 전체 재조회.

사용자 결정 흐름: 방법 A(재조회 폐기·카드만 처리) vs B(배치 확정) →
A 선택 → "삭제 대기로 희미하게" 제안 → A-1(즉시 삭제 쓰기 + 카드 희미)
vs A-2(진짜 대기·일괄 확정) → **A-1 확정**.

- ✕ 클릭 → 삭제 쓰기(excludedUids·userCompleted·scores) 즉시 실행(현행),
  확인 모달 학생당 1번(현행)
- `tpToggleTestProgress` 2회 호출 **폐기** → 그 카드만 opacity 0.4 +
  취소선 + grayscale + pointer-events:none, ✕ 버튼 제거
- ✕ 버튼 3곳(일반/통과/녹음 카드) `tpExcludeStudent(...,this)` 로 btnEl
  전달 → `btnEl.parentElement` 직접 dim. btnEl 없을 때만 옛 재조회 폴백(안전망)
- 효과: 여러 명 지워도 **재조회 0회**. 통계·목록은 그 시험 다시 펼칠 때 갱신
- tp(시험관리)·tl(시험목록)·pd(진도체크 일자별) 3경로 공통 적용

### 2) 출제옵션 기본값 변경 (`376b576`, SW v569)

신규 배정분에만 적용 (사용자 컨펌: 이전 출제분은 그때 옵션대로 유효, 폴백 미변경):
- 녹음숙제 **최소시간** 기본 `?? 60` → `?? 20` 초 ([app.js:13514](public/admin/js/app.js))
- 단어 말하기 **엄격도** 기본 `normal selected` → `lenient selected` +
  저장 폴백 `|| 'normal'` → `|| 'lenient'` ([13586/13729](public/admin/js/app.js))
- 미설정/옛 시험 클라 폴백(`speakingStrictness || 'normal'` 4곳,
  cfg `minDurationSec:60`)은 **그대로** — 기존 시험 채점기준 불변(의도)

### 3) 진단 — Vercel 로그 분석 (코드 변경 없음)

- check-word 503 1건: `2.5-flash-lite` 503 → `3.1-flash-lite` 200, 4.2s
  < 9s 타임아웃 → 5/18 폴백 재배치·타임아웃이 받쳐준 **정상 성공** (조치 X)
- 녹음숙제 24h 42건 중 14건 폴백, **503 없음** → 원인은
  `gemini-2.5-flash-lite` 200 응답인데 **JSON 파싱 실패**(스키마 무겁고
  `maxOutputTokens:3000`+`temperature:0.9` 로 출력 잘림 → `_salvageTruncated`
  복구 실패 → 같은모델 1회 재시도 → 다음 모델). 첫 200 호출 비용 청구·폐기
- check-recording 폴백 조건 = (a) isRetryable HTTP(503/429/404) 또는
  (b) 200 인데 parse 실패. 503 없으면 (b) 가 지배
- **보류(컨펌 대기)**: 옵션 A(`maxOutputTokens` 3000→5000~6000, 잘림 근본
  해소·temperature/피드백 정책 불변) 권장. 14건 첫 상태 200 확인 후 진행 예정

### 작업 규칙 추가 (2026-05-19 이어서 3)

- **재조회 없이 카드만 처리 패턴** — 행 단위 삭제/제외 후 전체 재조회
  (`tpToggleTestProgress` 2회 등) 대신 btnEl→parentElement 직접 dim.
  여러 건 처리 시 reads 0. 통계는 다음 펼침 때 갱신(허용). btnEl 폴백 유지
- **기본값 변경은 신규 배정분만, 폴백 불변이 기본 안전** — 출제 모달
  default 만 바꾸고 미설정/옛 데이터 클라 폴백은 두면 기존 시험 채점기준
  불변. 폴백까지 바꾸려면 별도 컨펌(이미 응시분 영향)
- **폴백 ≠ 실패, 503 ≠ parse-fail** — Vercel 로그 폴백 진단 시 첫 호출
  상태코드 필수 확인. 503/429=구글 용량(우리 못 고침), 200 후 폴백=출력
  잘림(우리가 maxOutputTokens 로 고침). 최종 200 이면 학생 영향=속도뿐

## 파일 크기 / SW 캐시 (2026-05-19 이어서 3)
- `public/admin/js/app.js`: +~15줄 (tpExcludeStudent dim + 기본값)
- `public/sw.js`: v567 → v569
- SW 캐시: `kunsori-v569`

---

## 2026-05-20: 공지 다중 첨부·만료일 + 언스크램블 난이도 재정의 (정정 1회)

SW v569→v570 (1회 bump, 공지). 언스크램블 변경은 api 전용 — SW bump 없음.

### 1) 공지관리 파일 첨부 (commit `ea4d6b5`, SW v570)

자료실·메시지와 동일 정책으로 공지에도 첨부 — 단 다중·만료일 사용자 지정.

**학원장 (공지 작성·수정 모달):**
- 📅 만료일 `<input type="date">` (기본 오늘+30일) — 학원장 자유 변경
- 📎 다중 첨부 — 드래그&드롭 또는 클릭. 파일별 ✕ 제거
- 검증: 파일당 20MB / PDF·Office·한글·이미지·텍스트 화이트리스트
- 수정 모달은 기존 첨부 prefill (status:'done' + url, 새 파일만 저장 시 업로드)
- 안내문 박스: 허용 형식·Storage 1년 자동삭제 명시

**학생 (공지 화면):**
- 공지 상세에 첨부 다운로드 버튼 N개 (📄 파일명·크기·↓)
- "📎 N개 · YYYY-MM-DD 까지 다운로드 가능" 안내
- **만료 후**: "🔒 첨부 파일 보관 만료 (YYYY-MM-DD 까지였음)" — 다운로드 차단
- 목록(홈/전체): 제목 옆 📎 (만료면 🔒)

**Storage·인프라:**
- `notices/{academyId}/*` 경로 (storage.rules 에 이미 깔려있음 — 2026-05-02 미리 보강)
- `scripts/admin/set-notice-attachments-lifecycle.js` 신규 (365일 GCS lifecycle 안전망)
  - 사용자 지정 만료일은 학생앱 표시·차단용. Storage 자체는 1년 안전망. 객체별 정확 만료는
    GCS Lifecycle 로 불가(일률 N일 룰만 가능) — cron 별도 인프라 필요. 베타엔 1년 안전망 단순
- `--apply` 적용 완료 (기존 lifecycle: recordings 60일·messageAttachments 10일 + notices 365일 → 3개)

**데이터 모델 (notices doc):**
- `expiresAt: Timestamp` 추가
- `attachments: [{ url, name, sizeKB }, ...]` 추가 (배열 — 메시지는 단수 `attachment` 와 분리)
- 옛 공지(이 필드 없음)는 그대로 표시 — 첨부 영역 안 보임, 호환

**구현 — 메시지 패턴 재사용**:
- `_msgAttachAllowed(type)` 화이트리스트 검증 헬퍼 그대로 재사용
- 헬퍼 5종 (`_noticeRenderAttaches`/`_noticeAcceptFile`/`_noticeUploadAll`/`_noticeClearAttaches`/`_noticeAttachBoxHtml`) + window 4종 (`noticePickAttach`/`noticeRemoveAttach`/`noticeDragOver`/`noticeDragLeave`/`noticeDrop`)
- 학생앱: `_noticeAttExpired(n)`/`_noticeAttExpYmd(n)`/`_noticeAttachmentsHtml(n)` 3종 신규

### 2) 언스크램블 난이도 한 단계씩 쉬운 쪽으로 재정의

학생들이 어렵다 평가 → 사용자 결정: 현재 중→상, 하→중, 새 하=쉽고 고빈도. 단 처음
7개 유형 모두 적용했다가 사용자 정정("언스크램블만") 으로 6개 원복, 언스크램블만 유지.

**최종 언스크램블 새 정의** (commit `60f66ab` → `b805afb` 정정 후, [api/generate-quiz.js:421](api/generate-quiz.js#L421)):
- 하 (NEW) = ≤8단어, 초등~중1 고빈도 단어만 (800-1000 word range)
- 중 (NEW) = 8-12단어, 단순 문법 + 일상 기본 단어 (기존 하)
- 상 (NEW) = 10-14단어, 일반 문법 + 일상 어휘. **관계절·분사구문 금지, 희귀/고급 단어 금지** (기존 중)
- 옛 상(긴 문장 + 복잡 구조) **폐기**

UI 라벨(상/중/하) 그대로 유지 — 학원장 같은 select 로 자동 한 단계 쉬운 출제 적용.
옛 출제분(이미 박힌 difficulty 필드)은 그때 정의대로 표시·풀이됨, 무관.

**나머지 6개 유형(vocab Type B / recording / MCQ-content / MCQ-grammar / subjective / fill_blank) — 원래 정의 그대로 유지** (사용자 의도).

**원인 — 사용자 의도 오해**:
- 사용자 흐름: "언스크램블 난이도?" → "어휘 수준은 어떻게 판단?" → "단어는 본문 안에만?" → "추천한 방식을 하로"
- 직전 추천(B안)은 "5개 유형 모두 강화"였으나 사용자는 줄곧 언스크램블 맥락
- "추천한 방식" = 'B안의 정신(쉬운/고빈도 단어)을 언스크램블 하에 적용'으로 봐야 했음. **맥락 끝까지 명확히 — 한 도메인 안 결정인지 전체 일괄인지 컨펌 필수**

### 작업 규칙 추가 (2026-05-20)

신규:
- **"전체 유형 일괄 적용" vs "현재 맥락 한 유형만" 구분** — 사용자가 특정 유형(언스크램블)
  맥락에서 난이도·옵션 변경을 지시하면 그 유형만으로 한정. "B안 추천=5개 유형 강화" 같이
  내가 직전에 제시한 옵션이 광범위해도 사용자 채택 시점 발화 ("추천한 방식을 하로") 가 좁은
  맥락(언스크램블)이면 좁게 해석. **확장 적용은 별도 컨펌**. [feedback_confirm_specs_before_work]
  의 강한 적용 사례.
- **객체별 정확 만료는 GCS Lifecycle 로 불가 — cron 필요** — `notices/expiresAt` 같이 doc 별
  사용자 지정 만료일은 일률 lifecycle 룰(`age: N`)로 못 따라감. 정확 정리 원하면 Vercel cron
  + admin SDK 가 만료된 doc 의 storage path 를 deleteObject. 베타엔 1년 안전망 단순 정책.

## 파일 크기 / SW 캐시 (2026-05-20)
- `api/generate-quiz.js`: 언스크램블 difficulty 정의 +3줄
- `public/admin/js/app.js`: 공지 첨부 헬퍼·UI +~180줄
- `public/js/app.js`: 공지 첨부 표시 +~30줄
- `storage.rules`: notices/* 경로 이미 적용됨(2026-05-02) — 변경 없음
- `scripts/admin/set-notice-attachments-lifecycle.js`: 신규 ~80줄
- SW 캐시: `kunsori-v570`

---

## 2026-05-21: 언스크램블 학생 화면 긴 문장 글자 단계 축소 + 안전망 스크롤

학원장 보고 — 언스크램블에서 문장이 길면 청크 선택지문이 화면 밖으로
밀려 안 보이고 스크롤도 안 됨 ('Captain awesome · 1 Captain awesome
CH5 · 5월 20일' 재현). SW v570 → v571 (commit `e6b32e0`).

### 원인
`unscrambleQuiz` 화면이 flex column 인데 합체카드(한글뜻+완성중)와
청크 영역 둘 다 `flex-shrink:0` + 그 아래 `<div style="flex:1">`
spacer → 콘텐츠가 길어지면 청크가 잘리고 footer 가 화면 밖으로 밀림.

### 옵션 검토 → D 채택 (사용자 정교화)
A(청크만 스크롤) / B(합체카드도 축소+청크 스크롤) / C(화면 전체
스크롤) / **D(자동 글자 축소 + 안전망 스크롤)**. 사용자가 D 채택 후
정교화 — 청크만이 아니라 한글뜻·완성중·청크 3개 영역 **동시 축소**
(시각 일관성), 최소 13px (12px 부담), 13px 에도 안 들어가면 청크만 스크롤.

### 구현
- `_app.html` — 청크 영역 부모 `id="uqChunkArea"` + `flex:1;
  min-height:0; overflow-y:hidden`. 그 아래 `flex:1` spacer 제거
  (청크 영역이 잔여 공간 차지). 합체카드·footer 위치 불변
- `app.js` `_uqRenderStep` 끝 `requestAnimationFrame(_uqFitContent)`
- `_UQ_FONT_TIERS` 3단계 — `{15,14,15} → {14,13,14} → {13,13,13}`
  (한글뜻/완성중/청크)
- `_uqApplyFontTier` — meanEl·builtEl·모든 청크 button fontSize 일괄 set
- `_uqFitContent` — tier 순차 적용 후 `scrollHeight > clientHeight+2`
  측정. 13px 까지 줄여도 overflow 면 청크 영역만 `overflow-y:auto`

### 작업 규칙 추가 (2026-05-21)
- **긴 콘텐츠 단계 글자 축소 + 안전망 스크롤 패턴** — 화면 고정 영역에
  가변 콘텐츠가 넘칠 때, 관련 영역들 글자를 단계표(tier)로 동시 축소
  → 각 단계 후 `requestAnimationFrame` 으로 `scrollHeight` 측정 →
  최소 단계에도 overflow 면 해당 영역만 `overflow-y:auto`. 인쇄
  `_tpApplyFitToPage`(zoom) 와 같은 정신, 화면용은 font-size 단계.
  시각 일관성 위해 단일 영역만 줄이지 말고 관련 영역 동시 축소.

## 파일 크기 / SW 캐시 (2026-05-21)
- `public/_app.html`: 청크 영역 flex:1 + spacer 제거 (~-2줄)
- `public/js/app.js`: `_uqFitContent`/`_uqApplyFontTier`/`_UQ_FONT_TIERS` +~33줄
- SW 캐시: `kunsori-v571`

---

## 2026-05-22: 단어 말하기 평가 방식 재검토 + 검증 페이지 2종 (spk-test / spk-exam)

단어 말하기(vocab speaking) 의 AI(check-word) 의존 문제 재검토.
**진행 중 — 베타 평가(여러 명 테스트) 후 방향 결정.** 학생 앱 코드 변경 없음
(독립 검증 페이지만 신규), SW 캐시 변동 없음.

### 1) AI 503 진단 — durationMs 로 클라 타임아웃 초과 확인
- Vercel `/api/check-word` 로그: 503 폴백이 떠도 최종 200 (폴백 정상 작동)
- 단 상세 로그 `durationMs`: 30분 33건 전부 9.7~52초 — 클라 타임아웃 9초 초과
- 결론: 503 자체보다 Gemini 응답 지연이 학생 체감 문제. AI 의존을 줄이는 방향 검토

### 2) 음성 평가 방식 검토 (STT vs LLM)
- `check-word` 는 Gemini(LLM) 오디오 멀티모달 — 무겁고 느림. 단어 채점에 부적합한 도구
- 빠른 음성 인식 = STT (Web Speech API 와 같은 계열). 우리 1·2차가 이미 STT
- 단어 1개는 STT 에 가장 불리 (문맥 보정 0). 문장이면 문맥으로 인식률 ↑
- 검토 안: 발음평가 API(Azure) / Capacitor 네이티브 앱 / 빈칸 문장 / 1·2·3차 단계 흐름
- 사용자 채택 방향(검토 중): 1차 영어 STT → 2차 한국어 STT → 3차 빈칸 문장.
  응시 시점 AI 호출 0, 출제 시점에만 데이터 생성

### 3) 검증 페이지 2종 신규 — 학생 앱과 분리된 독립 정적 페이지
- `public/spk-test.html` — 음성 인식 방식 **단건 검증**. 1·2·3차 버튼 + 실시간 문자화
  + alternatives·신뢰도·유사도·이벤트 로그. 최근 7일 AI 도달 단어 72개 드롭다운 +
  정답 발음 듣기(TTS)
- `public/spk-exam.html` — **실제 시험 형태 모의**. 1·2·3차 자동 흐름(영어 STT →
  한국어 STT → 빈칸 문장) + 힌트(스펠링 2글자) + 정답/오답·SKIP. 학생 앱 시험 화면
  디자인(코랄·합체 카드) 적용
- 둘 다 Firebase·로그인·서버 무관 순수 클라이언트. `quiz-test.html` 선례와 동일 패턴
- 접속: `raloud.vercel.app/spk-test.html` · `/spk-exam.html`

### 4) 진단 스크립트 fix
- `analyze-speaking-ai-dependence.js` — wordToAi 집계가 answer 객체에서 word 를
  찾던 버그 → `questions[i].word` 참조로 수정 (말하기 단어는 questions[] 에 있음)

### 작업 규칙 추가 (2026-05-22)
- **음성 인식 도구 선택 — STT vs LLM** — 음성→텍스트/점수는 STT 전용 엔진이 빠름
  (실시간). LLM 오디오 멀티모달(check-word)은 무겁고 느려 단어 채점엔 부적합.
  Gemini 음성 대화가 빠른 건 LLM 이 아니라 앞단 스트리밍 STT 덕분.
- **단어 1개 STT 는 가장 불리** — STT 는 문맥(앞뒤 단어)으로 보정해 정확해짐.
  단어 하나만 떼면 보정 0. 문장(빈칸 문장)으로 읽게 하면 인식률 자체가 올라감.
- **가설 검증은 독립 정적 페이지로** — 앱 통합 전 `spk-test.html`/`spk-exam.html`
  처럼 Firebase·로그인 무관 단독 페이지로 모바일 실측. 학생 영향 0, 빠른 반복.

## 파일 크기 / SW 캐시 (2026-05-22)
- `public/spk-test.html`: 신규 ~310줄 (음성 인식 단건 검증)
- `public/spk-exam.html`: 신규 ~430줄 (모의 시험 — 1·2·3차 흐름)
- `scripts/diag/analyze-speaking-ai-dependence.js`: wordToAi fix
- SW 캐시: `kunsori-v571` (학생 앱 무관 — 변동 없음)

---

## 2026-05-22 (이어서): AI OCR 스크롤 fix + 메시지 관리 정비 + 빈칸 문장 난이도 하향

SW v571 → v576. spk-test/spk-exam 검증 페이지는 학생 앱과 분리된 독립 페이지라
SW 무관.

### 1) AI OCR 화면 전체 스크롤 제거 (SW v572)
- `genGrid` 가 `height:calc(100vh - 280px)` 고정 — 위 요소(헤더·업로드 카드·여백)
  실제 합이 280px 초과 시 미세 스크롤·하단 버튼 가림
- `#page-generator.active` 를 flex column + 화면 콘텐츠 높이 고정, `genGrid` 는
  `flex:1` 로 남은 공간 정확히 차지 (calc 추정 폐기)

### 2) 메시지 관리 정비 (SW v573~v576)
- **행 2줄 압축** — 제목줄 오른쪽에 받는사람·날짜 배치 (제목/내용/받는사람·날짜 3줄 → 2줄)
- **날짜 필터** — '메시지 관리'·'발송 이력' 글자 옆 date input. 기본 어제,
  변경 시 그 날짜분만 (server-side `createdAt` 범위, 추가 인덱스 불필요)
- **검색** — 날짜칸 옆 검색 input. 메시지 관리=제목·내용·반 / 발송 이력=+이름.
  학원 메시지 저장 한도(`draftsPerAcademy`/`sentMessagesPerAcademy`)만큼 1회
  fetch·캐시 후 클라 부분일치 필터 (debounce 300ms). 한도=저장 최대치라 누락 0
- **삭제 시 상태 유지** — `delMsg`/`delDraftMsg` 가 `loadMessages()` 통째
  재호출 → 어제 날짜 리셋되던 문제. 현재 날짜·검색 유지하며 그 카드만 목록·캐시
  제거 + 한도 -1 + 부분 재렌더. 연이어 삭제 가능

### 3) spk-test/spk-exam — 빈칸 문장 단어 초등 저학년 수준으로
- 목표 단어 외 단어를 가장 흔한 500단어 수준으로 8개 교체:
  carpet→paper, storm→wind, blanket→bed, fur→hair, frost→winter,
  library→room, caves→a cave, daily→every day
- 근거: STT 문맥 보정에 필요한 건 "풍부한 문맥"이 아니라 "STT 가 정확히 인식하는
  주변 단어". 쉬운 고빈도 단어가 STT 인식 안정적 → 쉬운 문맥이 오히려 보정 유리.
  추상적 목표 단어만 일부 한계 (speaking-unfit 게이트로 처리)

### 작업 규칙 추가 (2026-05-22 이어서)
- **항목 삭제 UI 표준** — 삭제 시 전체 재조회·필터 리셋 X. 현재 상태(필터·검색·
  페이지·펼침) 유지하며 그 항목만 화면·캐시에서 제거 + 카운트 -1 + 해당 영역만
  재렌더. 사용자가 특별히 다르게 언급할 때만 예외. ([memory/feedback_delete_keeps_state.md](memory/feedback_delete_keeps_state.md))
- **AI 에 난이도 지시는 측정 가능한 기준으로** — "쉽게"는 주관적이라 AI 가 매번
  다르게 해석. "가장 흔한 500단어 이내", "CEFR A1", 단어 수 제한, 예시(few-shot)
  같은 측정 가능 기준 필수. 초등 1~2학년 ≈ 가장 흔한 300~500단어 수준.

### 단어 말하기 진행 상황
- spk-test/spk-exam 으로 베타 평가 중. **몇 차례 더 평가 후 학생앱 통합 예정** —
  미완료. 1·2·3차 흐름(영어 STT → 한국어 STT → 빈칸 문장) + 응시 시점 AI 0 방향.

## 파일 크기 / SW 캐시 (2026-05-22 이어서)
- `public/admin/style.css`·`_app.html`·`js/app.js`: AI OCR flex + 메시지 관리 정비
- `public/spk-test.html`·`spk-exam.html`: 빈칸 문장 8개 단어 교체
- SW 캐시: `kunsori-v576`

---

## 2026-05-23 ~ 24: 단어 말하기 신 흐름 통합 + AI 프롬프트·클린업 프리셋 동기 모델 통일

SW v577 → v588 (~13 commit).

### 1) 단어 말하기 1·2·3차 흐름 전면 개편 (응시 시점 AI 0)

옛: 1·2차 영어 STT → 3차 MediaRecorder + check-word AI (503 9.7~52초 지연 위험).
신: **1차 영어 STT (en-US, 닫힌후보 가드 유지) → 2차 한국어 STT (ko-KR, 한글 발음표기
매칭, 임계 0.7) → 3차 영어 빈칸 문장 STT (en-US, 목표 단어 부분 매칭, 임계 0.7)**.

- 출제 시점 `HOMOPHONES_PROMPT` 5필드 동시 생성 (homophones / koPron / sentence /
  sentenceKo / speakingTip) → 응시 시점 AI 호출 0 (check-word 폐기)
- check-word.js / MediaRecorder / silenceDetection / gumWithTimeout / blobToBase64 /
  cleanAiReason / _vqStartFinalAttemptMR — ~250줄 제거
- tpPublish 검증 게이트 — 4필드 누락 시 배정 차단
- 백필 스크립트 신규 — default 학원 vocab+speaking 161건 처리 (1500+ 단어, 4필드 자동 채움)
- 옛 시험 + 백필 안 된 단어는 학생앱 폴백 (1·2·3차 모두 영어 SR)
- 학생 힌트 UI — 스펠링 2글자, 점수 영향 없음. footer 위치(마이크 zone 움직임 방지)
- 학원장 베타 피드백 2차례 반영:
  · 1차: 힌트 footer 이동 / 2차 안내 단순화 / 3차 빈칸 회색 박스 가림 / 3차 라이브 STT /
    3차 정답 시 문장 노출·자동 발음·클릭 재생
  · 2차: 2차 통과 "한국식 발음" 멘트 제거 → speakingTip(5번째 필드, 단어별 발음 코칭
    25자, 예: "R 발음 — 혀 끝 말지 말기") / 정답 카드 글자 22px / 🔊 30px
- 3차 채점 완료 후 vqSpkLive 박스 유지 — 학생이 자기 발음 인식 결과 확인

신규 spkSource 값: `webspeech-1` / `webspeech-2` / `webspeech-3`. 학원장 상세 /
학생 상세 / analyze-speaking-ai-dependence.js 모두 신 값 대응.

iOS Safari Web Speech 정상 동작 확인 (아이패드 spk-test.html 테스트). 회귀 우려 해소.

### 2) AI 프롬프트·클린업 프리셋 동기 모델 통일

학원장 "AR 1.5 본문에 'consequence' 같은 어른 어휘" 보고 → mcq(본문이해)
**VOCABULARY MIRRORING 규칙** + 난이도 사고 단계(FACT-FINDING/COMPREHENSION/INFERENCE)
명확화. 그 과정에서 코드 default vs Firestore super 글로벌 갈라짐 발견:

7개 AI 프롬프트 중 4개(mcq/mcq_grammar/subjective/unscramble)가 2026-05-10 super
편집분 그대로. 그 이후 코드 변경 4건이 운영 미반영.

**자동 sync 도입 시도 + 즉시 철회** (902e4bd → dad90e8):
- 단방향 자동 sync(코드 → Firestore) 도입했더니 super 편집 후 코드 박기 전에
  학원장 출제 한 번에 super 편집 손실 → 자동 sync 제거

**최종 정책** — Firestore(super 글로벌) = 진실 출처. 양방향 동기는 명시 요청 시에만.
```
[학원 커스텀] academies/{id}.customPrompts / customCleanupPresets  ← 영구, 우선
   ↓ 없으면
[super 글로벌] appConfig/aiPrompts / appConfig/cleanupPresets       ← 진실 출처
   ↓ 글로벌 비었을 때만
[코드 default]                                                     ← emergency fallback
```

**클린업 프리셋 모델 변경** — 옛 "학원 본인 컬렉션이 진실 출처(genCleanupPresets) +
super 글로벌은 시드만" 구조 → AI 프롬프트와 동일 구조로 통일:
- super 글로벌이 진실 출처 + 학원 추가/수정만 `academies/{id}.customCleanupPresets`
- 마이그레이션 4개 학원 학원 커스텀 보존 (default "객관식 문법문제 생성" 1771자 등)
- `_cleanupLoadPresets`/Save/Duplicate/Delete 재작성. `_cleanupSeedDefaults` 폐기
- Firestore 규칙 `academies/{id}` update 키에 `customCleanupPresets` 추가

**도구**:
- `scripts/admin/push-aiprompt-to-firestore.js` — 코드 → Firestore 박기 (AI 프롬프트)
- `scripts/migrate/cleanup-presets-to-academy-custom.js` — 옛 학원 컬렉션 → 학원 커스텀
- `scripts/diag/check-aiprompts-sync.js` — 코드 vs Firestore 차이 진단
- 향후 사용자 요청 시 admin SDK 로 양방향 동기

**super 앱 화면 안내** (public/super/index.html): AI 프롬프트 + 클린업 프리셋
헤더에 동기화 필수 빨간색 경고. 클린업 "시드값" 표현 폐기.

### 작업 규칙 추가 (2026-05-23~24)

- **응시 시점 AI 호출 0 패턴** — 학생 응시 흐름은 AI 호출 없이 동작하도록 설계.
  필요한 AI 데이터는 **출제 시점**에 미리 생성·박음. 503/타임아웃 운영 리스크 제거.
  tpPublish 게이트로 데이터 누락 시 배정 차단.
- **단방향 자동 sync 는 의도와 반대로 작동할 수 있음** — 코드를 진실 출처로 가정한
  단방향 sync(코드 → Firestore)는 super 앱 편집을 즉시 손실시킴. 코드 vs Firestore
  양쪽 모두 의미 있는 변경 경로일 때는 **자동 sync 제거 + 명시 요청 트리거**가 정답.
  Firestore 가 진실 출처면 코드는 fallback 만.
- **시드 모델 vs 진실 출처 모델 구분** — 옛 클린업 프리셋처럼 "글로벌 default 는
  시드만, 학원 본인 컬렉션이 진실 출처" 구조는 super 편집이 기존 학원에 반영 안 됨.
  AI 프롬프트처럼 **글로벌 진실 출처 + 학원 커스텀 별도** 가 일관성 ↑.
- **본문이해 mcq 어휘 mirroring** — AI가 본문 어휘 풀에서 벗어난 어른 단어
  (consequence/demonstrate 등) 끌어 쓰면 학생 당황. **본문 어휘 우선 + 학년 흔한
  단어 보조**로 강제. 난이도 상/중/하는 **사고 단계**(사실/이해/추론) 한 축만 조절.
- **양방향 동기 자동화는 코드 측 제약으로 불가** — 코드는 git/deploy 만 변경 가능.
  Firestore → 코드 자동 동기 안 됨. 가장 안전: 사용자 명시 요청 시에만 양방향 처리.

## 파일 크기 / SW 캐시 (2026-05-23~24)
- `public/js/app.js`: 단어 말하기 1·2·3차 흐름 + 헬퍼 (~+300줄, check-word 등 ~-250줄)
- `public/admin/js/app.js`: tpPublish 게이트 / _fillMissingHomophones 5필드 / 클린업
  Save·Duplicate·Delete 신 모델 + _cleanupSeedDefaults 폐기 / 학원장 상세 차수 라벨
- `public/_app.html`: sentence area + 라이브 STT live area + 힌트 버튼 footer
- `api/generate-quiz.js`: HOMOPHONES_PROMPT 5필드 + mcq MIRRORING + subjective 갱신
- `public/super/index.html`: 동기화 필수 빨간색 안내 (AI 프롬프트 + 클린업)
- `firestore.rules`: academies update 키에 customCleanupPresets 추가
- 신규 스크립트:
  · `scripts/migrate/backfill-vocab-speaking-data.js` (백필)
  · `scripts/migrate/cleanup-presets-to-academy-custom.js` (클린업 이동)
  · `scripts/admin/push-aiprompt-to-firestore.js` (코드 → Firestore 동기)
  · `scripts/diag/check-aiprompts-sync.js` (진단)
- SW 캐시: `kunsori-v577` → `kunsori-v588`

## 진행률 (2026-05-23~24)
- **단어 말하기 신 흐름 통합: ~100%** (Phase 1~5 + 학원장 베타 피드백 2차례 + iOS 정상 + AI 호출 0)
- **AI 프롬프트·클린업 프리셋 동기 모델 통일: ~100%** (Firestore 진실 출처, 학원 커스텀,
  코드 fallback, super 앱 안내, 양방향 동기 도구)
- 본문이해 mcq 어휘 mirroring + 사고 단계 명확화: ~100%
- 멀티테넌시·super 앱·결제: 변동 없음
- Phase 5 출시 준비: 0%

**다음 세션 후보**:
- 옛 `genCleanupPresets` 컬렉션 안전 삭제 (운용 1~2일 안정 확인 후, 명시 트리거)
- 단어 말하기 베타 결과 관찰 + 추가 튜닝 (speakingTip 품질·정답 문장 노출 UX 등)
- 백필 안 된 시험 단어 (sent 검증 실패 ~7%) 재호출 검토
- Phase 5 출시 준비 (도메인·약관·결제 PG)

---

## 2026-05-24 (이어서): 부적합 단어 일관성 + 해석하기 옵션 + UX 정비

SW v588 → v593 (~6 commit).

### 1) advertisement TV 케이스 fix (단어말하기 출제 차단 해소)
출제 시 "advertisement" 가 누락 단어로 차단됨. 진단: sentence "I saw an advertisement on TV." 의 'TV' 영문 약자 때문에 sentenceKo Korean-only 검증 실패 → 빈값 유지. 매번 같은 응답이라 영영 안 채워짐.
- HOMOPHONES_PROMPT sentence 규칙에 RULE 6 신규 — 영문 약자/이니셜리즘 금지 (TV/USA/OK/FBI/NASA/iPhone/WiFi 등). "television" 같은 full word 사용
- backfill 스크립트 HOMOPHONES_PROMPT 도 동기
- admin SDK 로 advertisement 4건(sets 2 + tests 2) sentence/sentenceKo 수동 박음 ("I saw an advertisement in a magazine." / "나는 잡지에서 [광고]를 보았다.")
- 다른 영문 약자 의심 케이스 진단 — 0건 (advertisement 가 유일)

### 2) 부적합 단어 판정 일관성 (handleSpeakingUnfit)
보고: 같은 단어 세트 출제해도 매번 다른 부적합 단어 목록. 원인: callGemini 가 모든 task 에 temperature 0.7 사용 + hardForASR 기준 "highly likely" 가 AI 주관.
- A. callGemini 에 `opts.temperature` 추가 — handleSpeakingUnfit 만 0 호출 (분류 결정성). 출제 task 는 default 0.7 유지
- B. SPEAKING_UNFIT_PROMPT 강화 — "DEFAULT IS ALWAYS FALSE" + ">90% confident" 신뢰도 임계. hardForASR TRUE 는 ≤4글자 + 음향 모호만, wild/soft/right/light/world/fast 같은 흔한 단어 명시적 FALSE

### 3) 해석하기_주관식 옵션 — 문장 변형 / 문장 유지
학원장 요청. 옵션 2개 신설:
- 문장 변형 (paraphrase, default) — 기존 동작
- 문장 유지 (verbatim) — 본문 문장 그대로

구현:
- `SYSTEM_PROMPTS.subjective_verbatim` 신규 (2218자) — 본문 verbatim 강제
- POST handler `subjectiveMode` 파라미터 처리, promptKey 분기
- buildUserPrompt typeInstructions.subjective 모드별 분기
- validateSubjective 모드별 검증 — verbatim 은 본문 substring 매칭 강제, paraphrase 는 30% 단어 매칭
- 각 q.subjectiveMode 박음 + 세트 doc 메타 + 세트 목록에 라벨 (`✍️ 문장변형` / `📄 문장유지`)
- super 앱·학원장 앱 프롬프트 편집 모달에 subjective_verbatim 탭 추가
- push-aiprompt-to-firestore.js ALL_TYPES 에 추가 + Firestore 시드

### 4) UX 정비 (학원장 요청)
- AI 프롬프트 편집 모달 — subjective_verbatim 탭 추가 + 8개 탭 순서 재배열 (단어→빈칸→언스크램블→객관식(본문이해)→객관식(문법)→해석(변형)→해석(유지)→녹음). super 앱·학원장 앱 라벨 통일 ("해석하기 (문장변형/문장유지)")
- AI Generator 결과 모달 — 세트 이름 입력란을 문제 목록 위로 이동 (스크롤 불필요)
- 학생관리 검색 — 페이지네이션 로드된 학생만 검색하던 문제 → 학원 전체 학생 1회 fetch (limit 1000, academyId+role+status 만 필터, 반 무관) + 캐시 + debounce 300ms

### 5) 단어 말하기 — 3차 채점 후 라이브 STT 박스 유지
정답/오답 표시될 때 vqSpkLive 박스를 그대로 남겨 학생이 자기 발음 인식 결과 확인 가능 (1·2차는 변동 없음).

### 작업 규칙 추가 (2026-05-24)
- **분류 task vs 출제 task temperature 분리** — 모든 task 에 동일 temperature 사용하면 분류(yes/no)에서 결정성 손실. callGemini 에 opts.temperature 매개변수 두고 분류는 0, 출제는 default 0.7
- **AI 프롬프트 영문 약자 금지 (sentence/koTranslation 짝)** — 영어 sentence 에 TV/USA/OK 같은 약자 들어가면 한글 번역 측 영문 금지 규칙과 충돌해 검증 실패 무한 반복. 프롬프트에 명시적 금지 + full word 권장
- **모드 옵션 추가 시 응답 메타 + 세트 doc 메타 동시 박음** — subjective sentenceMode 처럼 옵션 추가 시 (1) validate 단계에서 각 q 에 메타 박고 (2) 응답에 모드 포함하고 (3) 세트 doc 에 모드 메타 박아 목록 라벨에 표시 — 3중 일관성

## 파일 크기 / SW 캐시 (2026-05-24 이어서)
- `api/generate-quiz.js`: HOMOPHONES sentence RULE 6 / SYSTEM_PROMPTS.subjective_verbatim / SPEAKING_UNFIT_PROMPT 강화 / callGemini temperature 옵션 / validateSubjective 모드 분기
- `public/admin/js/app.js`: subjective sentenceMode 옵션 / qgSaveSet 메타 / _qsBuildOptionsSummary 라벨 / 프롬프트 탭 8개 순서 + alias / 결과 모달 세트 이름 상단 / 학생 검색 학원 전체
- `public/super/js/app.js`: PROMPT_TYPES/LABELS 학원장과 동일 순서·라벨
- `scripts/admin/push-aiprompt-to-firestore.js`: ALL_TYPES 에 subjective_verbatim 추가
- SW 캐시: `kunsori-v589` → `kunsori-v593`

---

## 2026-05-24 (이어서 2): 시험명 편집 + 단어시험 한글·특수문자 검증 게이트

SW v593 → v597 (~4 commit).

### 1) 시험명 편집 ✏️ (오타 fix)
학원장 보고: 출제된 시험 제목 오타 발견 시 수정 UI 없음. 신규.

핵심 함수 `tpEditTestName(testId, currentName)`:
- 입력 모달 (`_showInputModal`) — Enter 확인 / ESC 취소
- `genTests/{testId}.name` update
- `scores` where(testId) 일괄 update — `testName` 필드 (`writeBatch` 450건/회)
- `userCompleted` 일괄 update — `testName` 필드
- 점수·정답·통과여부·questions/answers 등 모든 데이터 **보존**
- 화면 자동 갱신 (활성 화면 따라)

✏️ 노출 위치 (3곳):
- 통합 시험목록 행 (`_tlRenderRow`, 진도체크 → 시험별 진도체크 탭)
- 유형별 시험관리 행 (`_tpRenderTestRow`, 단어/녹음/등)
- 일자별 반별 진도체크 카드 (`progRenderByDate`)

스타일: opacity 0.4 → hover 1.0 (행 클릭 방해 최소화). `event.stopPropagation` 로 펼침 동작과 충돌 방지.

펼침 카드 중복 제거 (학원장 보고):
- 펼친 카드 상단 메타 라인 (시험명+✏️+응시통계) **통째 삭제** — 행/카드 헤더에 이미 있어 중복
- 학생 카드 색상으로 응시/미응시 한눈 구분 가능

### 2) 단어시험 세트 한글·특수문자 검증 게이트
학원장 보고: 단어/숙어에 한글이나 특수문자 섞이면 학생 답안 입력 단계에서 한글 입력 제한으로 제출 불가.

신규 검증 함수 `_qsValidateWordChars`:
- 한글 (가-힣ㄱ-ㅎㅏ-ㅣ) / 한자 (一-龯) / 일본어 (ぁ-んァ-ヶ) 검출
- 허용 문자: `a-zA-Z` 공백 `'` `-` `.` `/` `>` `~` `,` 숫자
  (변화형 표기 `/` `>` `~` 는 기존 데이터 보호 위해 허용)
- 허용 외 특수문자 → `'특수문자 (예시 5개)'`

게이트 모달 `_qsCharsGate` (말하기 부적합 게이트 패턴 차용):
- 부적합 단어 목록 + 사유 + **inline 수정 input + 🗑 삭제** 버튼
- [↻ 재검증] → [계속 진행 ▶] / [취소]
- 모든 단어 삭제 시 저장 불가 안내

호출 위치 (모든 vocab 저장 경로):
- `qgSaveSet` — AI Generator 단어시험 결과 모달 저장
- `qsSaveEdits` — 기존 세트 수정 저장
- `qgRunWordsnap` — Wordsnap 직접 입력 저장 (직전 fix 0b38b0c — qgSaveSet 우회 경로 누락 발견)

### 작업 규칙 추가 (2026-05-24 이어서 2)
- **데이터 일괄 update 시 `writeBatch` 450건/회 패턴** — Firestore batch 한도 500. 안전 마진 + 큰 시험 (학생 500명+재응시) 대비. testName update 처럼 단일 필드 일괄 변경에 적합.
- **저장 경로 우회 가능성 — 단일 진입점 가정 금지** — `qgSaveSet` 에 게이트 박았더니 Wordsnap (`qgRunWordsnap`) 이 직접 `addDoc` 호출해 게이트 우회. 데이터 무결성 검증은 **모든 저장 경로 grep 으로 확인 + 각 경로마다 명시적 호출** 필요. 단일 진입점 가정하지 말 것.
- **펼침 카드 ↔ 행 헤더 중복 = 행 우선 + 펼침은 본문만** — 동일 정보가 두 군데 노출되는 UX 는 사용자 혼란. 행/카드 헤더에 메타 (제목·편집·통계) 모으고 펼침은 본문(학생 카드 등) 만.

## 파일 크기 / SW 캐시 (2026-05-24 이어서 2)
- `public/admin/js/app.js`: tpEditTestName + _showInputModal / ✏️ 3곳 / 펼침 카드 메타 라인 제거 / _qsValidateWordChars + _qsCharsGate / 3경로 호출 / writeBatch import
- SW 캐시: `kunsori-v594` → `kunsori-v597`

---

## 2026-05-29: 녹음숙제 음성대역·단조로움 거부→안내 + 학원장 상세 수치 + 일시정지 경고

SW v599 → v602 (3 commit). OCR 2단 재정렬·cleanup 등은 직전 세션, 이번은 녹음숙제 정비.

### 1) 음성대역(C)·단조로움(D) 거부 폐기 → 안내 + 수치 노출 (`3b0d6c7`, v600)

`_rv2PreCheckRecording` 의 `voiceBandRatio < 0.40`(말소리 대역 에너지 비율) /
`monotony > 0.55`(1=완전 단조) 거부를 **폐기** — 정상 녹음(저가 마이크·또박또박
읽기·조용한 발음)도 막던 부작용 > 부정 차단 이득. 대신:
- 거부 아닌 **학생 안내**(warning, [저장] 그대로 가능) — `_rv2.alertMessage = check.warning`
- 회차별 수치를 **저장**해 학원장 상세 참고용 노출. `_rv2PreCheckRecording` 가
  `{ok:true, voiceActivity, voiceBandRatio, monotony, warning}` 반환
- 저장 경로 전부 carry: `_rv2AfterStop` currentTake / `_rv2Submit` recordingsDetail
  2곳 / `_rv2UploadRound` inProgress / `_raStartV2` 복원 (세션 간 이어하기 보존)
- 학원장 상세 `_adminRecBuildDetail` 에 `명료도 N%`(>=40 초록/미만 빨강) ·
  `단조로움 N%`(<=55 초록/초과 빨강) + 마우스 오버 툴팁. **학생 비공개·학원장 참고용**

**중요**: 수치는 genTests 설정이 아니라 **학생 제출 시점 클라가 계산** → 출제 시점
무관. 이미 출제된 시험도 v600+ 로 **제출하면** 반영. v600 이전 제출분은 필드 없어
미표시(소급 불가).

### 2) 수치를 말소리·속도 옆 같은 줄로 통합 (`ae17708`, v601)

별도 줄(`acousticLine`) 폐기 → 회차 헤더 `N초 · 말소리 N% · 속도 N WPM` 뒤에
` · 명료도 N% · 단조로움 N%` 이어 붙임 (학원장 요청). 라벨 "음성 명료도"→"명료도".

### 3) 일시정지 화면 이탈 미저장 경고 (`0dfed65`, v602)

녹음 일시정지 시 `[재개] 누르면 이어서 녹음됩니다` 안내 아래 빨간색
`⚠️ 이 화면을 벗어나면 저장되지 않습니다.` 추가 (currentTake 는 메모리라 화면
이탈 시 유실 — 1회차 저장 후 2회차 일시정지 중 이탈 케이스 명시).

### 직전 미반영분 (이번 세션 확정)

- 녹음 길이 일관성 검사(±30%, 옛 규칙 3) 이미 제거됨 — 일시정지 시 elapsedSec
  줄어 2회차 거부되던 버그. 600초 도달 = **자동 종료(녹음만 멈춤)**, 거부·자동제출
  아님. precheck 길이 거부는 `< minDur`(짧음) / `> maxDur+5`(엣지 안전망) 만 잔존

### 작업 규칙 추가 (2026-05-29)
- **AI 추정 음향 지표는 거부 아닌 안내로** — voiceBandRatio/monotony 같은 휴리스틱은
  정상 녹음 오탐 위험. 차단 대신 학생 안내 + 학원장 참고 수치 노출(학생 비공개,
  [[feedback_prompt_over_hardcode]] 정신 — 정상 사용자 막는 코드 게이트 지양).
- **제출 시점 계산 지표는 출제 무관·신규 제출분부터** — genTests 설정이 아니라 클라
  제출 시 계산되는 값은 이미 배정된 시험도 새 앱 제출분부터 반영. 소급 불가(과거 데이터 없음).
- **메모리성 currentTake 유실 경고는 UI 에 명시** — 일시정지 중 화면 이탈 시
  미저장(currentTake 는 blob 메모리). inProgress 는 회차 저장 후에만 박힘 → 진행 중
  회차는 보호 안 됨을 학생에게 안내.

## 파일 크기 / SW 캐시 (2026-05-29)
- `public/js/app.js`: precheck 거부→warning + 수치 carry(currentTake/recordingsDetail/inProgress/복원) + 일시정지 경고
- `public/admin/js/app.js`: `_adminRecBuildDetail` 명료도·단조로움 인라인 표시(색·툴팁)
- SW 캐시: `kunsori-v599` → `kunsori-v602`

---

## 2026-05-30: 시험명 ✏️ 따옴표 깨짐 + Book 이름 동기 누락 + CRUD 후 목록 즉시 반영 전수 정비

SW v602 → v608. 학원장 보고 3건 → 진단 후 작은 fix 묶음(✏️ data-attr, Book lazy fetch 동기, 송미정 백필) → 그 흐름으로 **CRUD 후 전체 reload 패턴 전수 조사·52건 surgical 전환**.

### 1) 시험명 ✏️ 버튼 따옴표 함정 (`f887983`, v603)

증상: 시험명에 따옴표(`'`/`"`) 들어가면 ✏️ 편집 버튼 클릭 무반응. 원인:
인라인 `onclick="...tpEditTestName('${esc(t.name)}')"` 의 `esc` 가 `'`→`&#39;`,
`"`→`&quot;` 로 인코딩 → HTML 파서가 속성값 엔티티 먼저 디코딩 → JS 문자열·
속성이 깨짐. CLAUDE.md(2026-05-18) 에 정립된 `data-*` 속성 패턴으로 3경로
공용 헬퍼 `_tpEditNameBtnHtml(t)` 통일. `data-eid`/`data-ename` →
`tpEditTestName(this)` 가 `el.dataset` 에서 원본 문자열로 안전하게 읽음.

### 2) Book 이름 동기 lazy 미로드 누락 (`9c6031f`, v605)

학원장 보고 — default 학원 "중2 예봉중 지학사 송미정" Book 의 2 chapter 중
"ch1" 의 `bookName` 만 옛 이름(`중2 지학사 송미정`)으로 남음. 진단(admin SDK):
chapter 1건 + 그 하위 page 3건 불일치. **원인:** `genDoEditBook` 이
`_genChapters.filter(c=>c.bookId===bid)` 메모리 캐시 기반 update 인데 AI OCR
은 **lazy fetch** — Book 펼치기 전엔 그 Book chapter/page 가 캐시에 없어
동기에서 통째 누락. **수정:** Firestore `where('bookId','==',bid)` 직접 쿼리
→ 캐시 상태 무관 전수 동기. 백필 스크립트(`backfill-book-name-sync.js`)로
기존 4건 정리 (DRY-RUN → 적용 → 검증). 전 학원 다른 불일치 0건.

### 3) CRUD 후 목록 즉시 반영 전수 조사·전환 (~52건, `8c5f0bd`·`054c101`·`60e76a1`, v606→v608)

`qsDeleteSet` 처럼 삭제 후 전체 reload 로 화면 상태(반 필터·검색어·페이지·
정렬·선택 Book·펼친 카드·스크롤) 리셋되던 케이스를 전수 조사 → 약 28건
처분 전 reload + 24건 이미 처리분 확인 → 전부 surgical 패턴 전환.

**공용 헬퍼 신설**:
- `_pageMutate(tableId, fn)` — `_pageState` data 배열 inline mutate, 현재
  page/sort 유지하며 `renderPage` (재fetch 0)
- `_stuSurgical(status, ids, opts)` — `allStudents` + `_stuSearchCache` 동기
  + 검색/페이지 모드 분기 재렌더 (반 필터·검색어 유지)

**전환 영역별 건수**:
- 학생: 일괄 작업 8건 (bulkAction pause/out/assign·doAssignClass·restore·
  outSelected·deleteSelected/Out) + 단건 2건 (saveStudent·updateStudent)
- 공지: 4건 (saveNotice unshift / updateNotice inline / deleteNotice ·
  deleteSelectedNotice filter)
- 자료실: 3건 (saveHwFileEdit · uploadHwFileAdmin unshift · deleteSelectedHwFile)
- 반 class: 4건 (saveClass push · updateClass inline · deleteClass ·
  deleteSelectedClass)
- 시험: 1건 (deleteSelectedTest — `_tlState.data` 동기)
- 문제세트: 2건 (qsRenameSet inline + qsSaveEdits inline + Book 폴더 변경
  시 옛/새 폴더 양쪽 정합)
- 메시지: 2건 (saveMessage faux snapshot unshift + 한도 +1, sendMessage
  캐시만 무효 — server 가 sent doc 생성이라 surgical insert 어려움)
- AI OCR Generator: 15건 (Book/Chapter/Page CRUD + 이동·제외·병합·OCR 배치).
  `loadGenerator({keepActive:true})` 폐기 → 메모리 캐시(_genBooks/Chapters/Pages)
  inline mutate + `_genRenderAll()`. 삭제·이동 시 `_genActiveBook`/`_genActiveChapter`/
  `_genActivePage` 정합 처리. genDoEditBook 의 chapter/page 동기는 §2 Firestore
  직접 쿼리 유지하고 결과를 메모리 캐시에도 반영.

**스킵·유지**:
- 결제 `markSelectedPaid`/`deleteSelectedPayment` — 호출처 0건·DOM 부재
  데드코드 (옛 payments 컬렉션 잔존). 라이브 결제는 `_billingToggleChannel`
  이미 surgical(2026-05-16)
- `_billingChangeTab`/`Month`/`TimelineJump`/결제 설정 마법사 — CRUD 아닌
  탭·월 전환·초기 설정 → loadPayments 풀 reload 유지(의도)
- `genRefresh`/메뉴 진입 `loadGenerator` 등 명시 새로고침 경로 유지

### 작업 규칙 추가 (2026-05-30)

- **인라인 onclick 인자에 esc 된 사용자 문자열 금지** — HTML 파서가 속성값
  엔티티(`&#39;`/`&quot;`)를 먼저 디코딩 → JS 문자열·속성이 깨짐.
  `data-*` 속성 + `el.dataset.xxx` 패턴 필수. 따옴표 들어간 사용자 데이터
  (시험명·반명·세트명 등) 핸들러 추가 시 즉시 적용.
- **lazy fetch 모델 동기 update 는 Firestore 직접 쿼리** — 메모리
  캐시(_genChapters/_genPages 등) 기반 cascade update 는 lazy 미로드 분이
  통째 누락. `where(...)` 직접 쿼리로 전수 잡고 그 결과를 메모리 캐시에도
  반영. 송미정 ch1 케이스가 표본.
- **CRUD 후 전체 reload 폐기 — surgical 갱신 기본** — 학원장 작업 후
  반·검색·페이지·정렬·스크롤·선택 Book 등 화면 상태가 리셋되면 매번
  되돌리는 비용 누적. 캐시(메모리·_pageState·_qsSetsByBook 등) 그 항목만
  mutate + 현재 화면 부분 렌더. CRUD 새 핸들러 작성 시 처음부터 적용.
- **검색 캐시는 별도 surgical 동기** — `_stuSearchCache` 같이 별도 fetch
  로 유지되는 캐시도 같이 mutate 안 하면 검색 결과에 stale 표시. 헬퍼
  (`_stuSurgical`)가 두 캐시 동시 처리.
- **server 생성 doc 은 invalidate 만** — sendMessage 처럼 server 가
  doc 생성하는 케이스는 정확한 id/필드 불명 → surgical insert 어려움.
  관련 캐시만 무효(`null`)로 두고 다음 fetch 에서 fresh. 검색·날짜 필터는
  유지 (loadX 풀 reload 의 reset 부작용 회피).
- **외부 작업지시서 / 단일 보고 검증 후 진행 (강화)** — Explore 서브에이전트
  보고("결제 일괄 처리 surgical 필요")가 실제로는 데드코드인 케이스 표본.
  보고 받으면 코드 grep·DOM 확인 후 적용. ([feedback_confirm_specs_before_work] /
  2026-05-19 외부 지시서 검증 흐름과 동일).

### 파일 크기 / SW 캐시 (2026-05-30)
- `public/admin/js/app.js`: +~200줄 순증 (_pageMutate/_stuSurgical 헬퍼 + 28개
  handler surgical 재작성). 전체 ~17k 줄 유지 (대부분 inline patch)
- `scripts/migrate/backfill-book-name-sync.js`: 신규 ~80줄
- SW 캐시: `kunsori-v602` → `kunsori-v608`

---

## 2026-05-31: 메시지 자동삭제 정책 + 녹음숙제/말하기 안내·구분 + 클래스 메모 + 단어시험 quota UI 제거

SW v608 → v614 (~7 commit). 사용자 보고·요청 6건 처리. 도중 검색 실수
1건(녹음숙제 안내 위치 오해)로 한 번 wrong place 수정 후 다시 정정.

### 1) 메시지 관리 — [전체] 버튼 + 자동삭제 정책 (`bfa12a3`·`f414775`, v609→v610)

`<input type=date>` 의 한국어 Chrome 클리어 버튼이 "삭제"로 표시돼 학원장 혼선.
브라우저 제어 라벨이라 직접 변경 불가 → 옆에 [전체] 명시 버튼 추가 (초안·발송
이력 양쪽). 툴팁 "날짜 필터 해제 — 저장 한도 내 전체 표시". [전체] 클릭은
limit(10) cursor fetch 라 가벼움 (검색 입력 시에만 학원 한도 100건 1회 fetch).

자동 삭제 정책 (학원장 결정 — 메시지 페이지 진입 시 fire-and-forget cleanup):
- pushNotifications.sent=true 60일 초과 → 한 번에 50건 batch 삭제
- userNotifications 학원 전체 30일 초과 → 한 번에 100건 batch 삭제 (Rules:
  admin only delete, 학원장이 학원 전체 처리)
- 메시지 관리(drafts sent=false) — 자동 삭제 없음, 학원장 수동 관리

발송 이력 헤더에 `⏳ 60일 후 자동 삭제 (학생 알림함 30일)` 표시 + 툴팁 풀버전.
인덱스 2개 신설·배포: `(pushNotifications, academyId, sent, createdAt ASC)`,
`(userNotifications, academyId, createdAt ASC)`.

### 2) 녹음숙제 출제 화면 안내 + 말하기 상세 오답 3분리 (`54cf4c7`, v611)

말하기 학원장 상세 오답 케이스 — 기존 `(음성 미감지/건너뜀)` 단일 라벨 → 3분리:
- `들린 단어: "..."` — `_heardRaw`(spkAiHeard||spkHeard) 있을 때
- `(음성 미감지)` — `spkSource` 있으나 _heardRaw 빈값 (3회 SR 무인식)
- `(건너뜀)` — `spkSource` 미설정 (SKIP 버튼만 누름, vqSkip 이 _vqSpkFinalize
  미호출). 옛 데이터는 spkSource 없으면 건너뜀, 있으면 미감지 자연 분류
각 라벨 툴팁으로 의미 부연 ("3회 시도했으나 음성이 인식되지 않음" / "학생이
SKIP 버튼을 눌러 시도 안 함").

### 3) 녹음숙제 화면 제목 아래 설명줄 정정 (`bb4c669`, v612)

직전 commit (`54cf4c7`) 에서 사용자가 준 기존 문구 "Page 단위 녹음숙제를
학생앱에 배정합니다..." 를 cfg.hint 자리(line 13682)에 새 문구로 교체했는데,
cfg.hint 는 `enabled:false` 잠금 화면용 dead code 라 rec-ai 화면에 안 보임.
사용자 지적("엉뚱한데를 찾아서 넣었어?") 후 정정:
- cfg.hint 원문 복구
- _app.html `#page-test-rec-ai` 의 page-sub 부분에 ' · 제출된 녹음은 60일간
  저장됩니다' 회색 보조 텍스트로 추가 (실제 시험관리 > 녹음숙제 페이지 헤더 자리)

**작업 규칙 (강화)**: 사용자가 화면에서 본 기존 문구를 정확히 인용해 줄 때 —
**그 문구로 literal grep 먼저** (`Page 단위 녹음숙제를 학생앱에 배정`). 부분
매칭("Page 단위 녹음숙제") 으로 검색하면 _heading 류 짧은 매칭이 잡혀 잘못된
위치로 갈 수 있음. cfg.hint 처럼 코드에 있지만 렌더 안 되는 dead code 도 있어
**실제 렌더 경로 확인**까지 추적 필요.

### 4) AI Generator 안내 갱신 + 클래스관리 메모 컬럼 (`2b2bed4`, v613)

- AI Generator 페이지 page-sub: '객관식 4지선다 문제' → '문제유형에 따라
  문제세트를 생성해' (실제로 단어/언스크램블/객관식/녹음숙제 등 여러 유형 지원)
- 클래스관리 테이블 — 사용 빈도 낮은 '앱표기숨김'(`g.hideApp`)·'All Books'
  (`g.allBooks`) 컬럼 제거, '메모' 컬럼 추가 (min-width:280px,
  white-space:pre-wrap+word-break:break-word 로 여러 줄 자연 표시)
- 생성·수정 모달에 textarea memo (rows=4), Firestore `groups.{id}.memo` 저장.
  기존 데이터는 memo 없이 '-' 표시 — 추가 마이그레이션 불필요
- 옛 hideApp/allBooks 필드는 Firestore 유지 (UI 미노출), 필요 시 별도 정리

### 5) 단어시험 quota UI 제거 (`3d1da64`, v614)

2026-05-23 단어 말하기 1·2·3차 Web Speech STT 전환으로 check-word AI 호출
폐기. `wordSpeakingCallsThisMonth` 카운터는 더 이상 증가하지 않으므로 UI 통째
제거 (사용자 요청).

- 학원장 앱 `loadQuotaUsage`·사용량 차트: 5분류 (OCR·Cleanup·Generator·
  녹음숙제·성장리포트) 로 통일. 6분류→5분류 주석 갱신
- super 앱: 학원별 사이드바 cell·Top10 테이블 🗣 단어 컬럼·aiSum/aiSumLimit
  계산·override label map·T6 한도 관리 PLAN_FIELDS 에서 모두 제거. colgroup
  폭 재조정 (남은 컬럼 비율 확대)
- **데이터·서버 측면 유지**: `api/check-word.js`·`api/_lib/quota.js` word-speaking
  config·Firestore plans/academies 의 wordSpeakingPerMonth/CallsThisMonth 필드.
  클라 호출 0이라 카운트 안 됨, 정리는 별도 마이그레이션 (현재 비우선)

### 작업 규칙 추가 (2026-05-31)

- **사용자 인용 기존 문구는 literal grep 먼저** — "Page 단위 녹음숙제를 학생앱에
  배정" 같이 사용자가 화면에서 본 정확한 문구를 줬을 때, 부분 매칭으로
  검색하지 말고 전체 문구 그대로 grep. 부분 매칭은 짧은 label/heading 류가
  먼저 잡혀 잘못된 위치로 갈 수 있음. 정확한 매칭 후 **실제 렌더 경로**
  (display chain) 까지 확인 — cfg.hint 처럼 코드엔 있지만 dead code 인 경우
  존재.
- **HTML 클리어 버튼은 브라우저 제어, 라벨 변경 불가** — `<input type=date>`
  의 한국어 Chrome '삭제' 버튼처럼 native UI 의 텍스트는 우리 코드로 못 바꿈.
  대안 명시 버튼 추가(예: [전체])로 의도 보조. iOS Safari 등 다른 브라우저는
  다르게 보일 수 있어 명시 버튼이 일관성 확보.
- **AI 카운터 폐기 시 데이터/서버 config 는 유지** — UI 만 제거하고 quota.js
  config·Firestore 필드는 그대로 두는 게 안전(클라 호출 0이라 카운트 안 됨,
  옛 누적값 보존). 정리(필드 삭제 마이그레이션)는 별도 작업으로 의도 분리.

### 파일 크기 / SW 캐시 (2026-05-31)
- `public/admin/_app.html`: 메시지 [전체] 버튼·녹음숙제 page-sub 보조 텍스트·
  AI Generator 설명·클래스 thead 재구성
- `public/admin/js/app.js`: 자동삭제 cleanup 함수 + 메시지 헤더 라벨 + 말하기
  3분리 라벨 + 클래스 saveClass/editClass/updateClass memo 필드 + 사용량 5분류
- `public/super/js/app.js`: 사용량 5분류·Top10 컬럼·PLAN_FIELDS 정리
- `firestore.indexes.json`: 메시지 자동삭제용 인덱스 2건
- SW 캐시: `kunsori-v608` → `kunsori-v614`

---

## 2026-06-01: 시험목록 월초 컷오프 폐기 + 말하기 부적합 게이트 강화 + 최근시험 스크롤 보존

SW v614 → v620 (~7 commit). 학원장 6/1 보고("최근시험 안 보임") 출발 → 월초 컷오프
2곳 패치 → 그 흐름으로 말하기 부적합 게이트 휴리스틱 3가지 확장 + 스크롤 UX 정리.

### 1) 시험목록 월초 컷오프 폐기 (`2168def`·`96f758b`·`232aa72`, v615→v618)

매월 1일 학원장이 보던 "최근시험 0건" 증상 — 두 곳에서 동일 원인.

- `_renderTestAssignDetail`/`loadMoreTpTests` (시험관리 > 각 시험 출제 화면 최근시험)
- `loadTestList`/`loadMoreTestList` (진도체크 > 시험별 진도체크)

기존 쿼리: `where(academyId)+where(testMode)+where(createdAt >= 월초)+orderBy(desc)+limit(20)`
→ 매월 1일 그 달 출제분 0건이면 빈 목록, 5월 시험 안 보임.

신: 날짜 필터 제거 → `where(academyId)+orderBy(createdAt desc)+limit(20)` + cursor 더보기.
`_tpTestsState.monthStartDate` / `_tlState.startDate` 필드 폐기. 라벨 '기간 내 모두
표시됨' → '모두 표시됨' (misleading 정리).

**전수 조사 결과**: 월초 컷오프 사용처는 이 두 곳이 전부 (`_ymdMonthStartKST()` 함수
호출 grep 0건). 결제/메시지/성적 리포트 등은 사용자 직접 날짜 선택(의도) 또는 rolling
N일 윈도우(의도)로 정상.

### 2) 말하기 부적합 게이트 휴리스틱 3가지 확장 (`81250d5`·`c5ea274`, v616·v619)

학원장 보고 "배정 불가 — 말하기 데이터 누락" (Hallelujah, keep ~ open, there 's)
계기. 자리표시·비정상 형식이 AI 채움 단계까지 가서야 차단되던 케이스를 게이트에서
선제 검출. `_tpSpeakingUnfitReasons`:

| 사유 라벨 | 검출 패턴 | 매칭 예 |
|----------|----------|---------|
| 자리표시 기호 포함 (신규) | `~`, `…`, `..` 이상 | `look ~ up`, `name ~ after`, `try...` |
| 비정상 띄어쓰기 (신규) | 공백+`'`/`'`+공백/연속 공백 | `there 's`, `it 's` |
| 특수문자/구분자 포함 (신규) | `/`, `>`, `,`, 괄호류, 통화기호 등 | `rice/fall`, `abandon, leave`, `name > called` |
| 3글자 이하 (기존) | 영문 추출 후 ≤3자 | `up`, `be`, `go` |

저장 게이트(`_qsValidateWordChars`)는 변화형 구분자(`/>~,`)를 단어장 표기 관행상
허용했는데, 말하기에선 발음 불가 — 게이트 두 개가 다른 정책 적용(저장 허용 ↔ 말하기
차단)으로 적절히 분리.

### 3) SPEAKING_UNFIT_PROMPT notRealWord 규칙 세분화 (`b13c507`)

학원장 질문 "Indianapolis, Mississippi 같은것도 사전에 없는 단어인가?" 계기. 이전
프롬프트 "Proper nouns (people/brands/countries) and abbreviations are TRUE" 가
지명·사전 등재 인물명까지 한 묶음으로 차단.

세분화 — Oxford/Merriam-Webster 사전 등재 여부 단일 기준:
- TRUE — 임의 인명(Tom·John) / 브랜드(Coca-Cola·iPhone) / 약자(FBI·USA·NASA) / 무의미(xqzy)
- FALSE — 일반 dict + **지명·국가·도시·강·산**(Mississippi·Tokyo·Korea·Paris) +
  사전 등재 역사/문학 인물(Einstein·Shakespeare)

HOMOPHONES_PROMPT 는 별도 수정 불필요 — RULE 2 "proper noun 대문자 유지" + RULE 4
"unusual 단어 best phonetic Korean approximation" 으로 이미 사전 등재 지명·인물의
koPron/sentence/sentenceKo 정상 생성. 통과 후 자동 채움 자연 동작.

### 4) 시험관리 최근시험 [+ 더 보기] 스크롤 위치 보존 (`ec96d19`, v620)

[+ 더 보기] 클릭 시 `_tpRender()` 재호출 → `innerHTML` 교체 → tests pane 스크롤이
처음(1번 시험)으로 리셋 → 학원장이 매번 다시 스크롤 내려야 하던 UX 불편.

`_tpRender` 가 sets pane(`tpSetsScroll`)만 prevScroll 캡처·복원했는데 tests pane
(하단 최근시험)은 누락. tests 스크롤 컨테이너에 `id="tpTestsScroll"` 부여 +
prevTestsScroll 캡처·복원 (기존 sets 패턴 동일). 이제 새 시험 append 만 되고 사용자
위치 그대로 유지.

### 작업 규칙 추가 (2026-06-01)

- **월초 컷오프 패턴은 학원장 혼선 일으킴 — 단순 최근 N개로** — 매월 1일 그 달 출제분
  0건이면 빈 목록 + 지난달 안 보임. 단순 cursor 페이지네이션이 직관적. 예외: 결제처럼
  월 단위가 본질인 영역만 유지.
- **저장 게이트 ↔ 말하기 게이트 정책 분리 OK** — 같은 데이터에 다른 정책 적용 가능.
  변화형 구분자(`/>~,`)는 일반 vocab 단어장에선 표기 관행상 허용, 말하기에선 발음 불가.
  게이트별 자기 기준으로 판단, 통일 강요하지 않음.
- **AI 프롬프트 boolean 분류는 단일 판정 기준 명시** — '모든 proper noun = TRUE' 같은
  광범위한 규칙은 합리적 케이스(지명·역사인물)까지 한 묶음으로 잘못 분류. 단일 객관적
  기준(예: 사전 등재 여부) 제시 + 카테고리별 examples 로 세분화 → AI 일관성·정확도 ↑.
- **재렌더 시 scrollTop 복원 패턴 — 사용자 위치 유지** — innerHTML 교체로 스크롤
  리셋되는 컨테이너는 prevScroll 캡처·복원 필수. 같은 페이지에 여러 스크롤 컨테이너가
  있으면 각각 따로 캡처(`tpSetsScroll` + `tpTestsScroll`).

### 파일 크기 / SW 캐시 (2026-06-01)
- `public/admin/js/app.js`: 월초 필터 제거 2곳 + 휴리스틱 3카테고리 확장 + tpTestsScroll
  스크롤 보존
- `api/generate-quiz.js`: SPEAKING_UNFIT_PROMPT notRealWord 4 카테고리 분리
- SW 캐시: `kunsori-v614` → `kunsori-v620`

---

## 2026-06-01 (이어서): 메시지 진입 UX + 성장리포트 녹음숙제 정성 분리

SW v620 → v623 (~3 commit). 학원장 보고 흐름.

### 1) 메시지 진입 시 최근 10개 + 발송 후 즉시 이력 표시 (`32509bb`, v621)

학원장 보고: 메시지관리 페이지 열면 초안·발송 둘 다 빈 칸 (default 날짜 필터
'어제' 라 어제 작성·발송 없으면 0건). + 메시지 발송 후 발송 이력에 즉시
안 보임 (loadMessages 재호출은 '어제' reset 부작용).

수정:
- `loadMessages` 날짜 필터 default `_msgYest` → `''` (빈 값 = 전체). `_msgFetchDrafts/Sent`
  가 날짜 필터 없을 때 academy+sent+orderBy createdAt desc + limit(MSG_PAGE_SIZE=10)
  자연 동작. 10개 초과 시 cursor 더보기.
- `sendMessage` 성공 후 캐시 무효만 하던 케이스를 `_msgSentState` 리셋 + `_msgFetchSent(false)`
  + `_msgRenderSentSection()` 로 변경. 현재 검색·날짜 필터 유지하며 최근 10개 재fetch.
  server 가 sent doc 생성 (정확 id/필드 불명) 이라 surgical insert 대신 재fetch.
  발송 한도 라벨 +1 도 즉시 반영. 옛 메모(server 생성 doc 은 invalidate 만) 케이스
  를 "현재 필터 유지 + limit 10 재fetch" 로 진화.

### 2) 성장리포트 녹음숙제 점수 분리 + 정성 코멘트 (`bbce155`·`d5874b4`, v622·v623)

학원장 요청: 성장리포트의 녹음숙제 점수는 객관성/신뢰도 문제로 학생 비공개
정책 (학생앱과 동일). 학원장 참고용으로만 수치 노출, 학부모 공유 PDF 에는
정성 표현만.

서버 (`api/growth-report.js`):
- 점수 집계에서 녹음숙제 제외 — `totalSum`/`totalAttempts`/`avgScore`/`passedCount`
  모두 `mode !== 'recording'` 만 합산. `modeBreakdown` 은 모드별 별도 표시용 유지.
- 녹음숙제 정성 데이터 수집 — 최근 10개 testId 의 `genTests/{testId}/userCompleted/{uid}`
  fetch → 최종 녹음의 `categoryScores`(발음·억양·속도·정확도) + `feedback.weakPronunciation`
  + `feedback.tips` 추출. `_aggregateRecordingQuality` 로 빈도순 약한 단어 Top 5 + 팁 Top 3.
- 출제/제출 카운트 — 최근 30일 학생 배정 recording 시험 수 (`recordingAssigned`)
  vs 제출 수 (`recordingSubmitted`=`scores.testId distinct`). target 필터 + excludedUids 제외.
- 응답 `recordingQuality` 에서 `avgCat` 제거 — `assigned`/`submitted`/`topWeakWords`/`topTips`
  만 클라에 전달 (학생 보호 정책).
- SYSTEM_PROMPT `recordingComment` 가이드 강화 — **출력에 절대 수치(점수·%) 금지**.
  AI 가 카테고리 점수 내부 추론용으로만 사용, 정성 표현("안정적·꾸준히 향상 중·
  더 다듬을 필요·흔들리는 부분") 만 출력. 약한 단어 예시(right·world)는 OK.
  좋은/나쁜 예시 명시. 응시 0건 시 안내 문구.
- `RESPONSE_SCHEMA` 에 `recordingComment` 필드 추가 (required).

클라이언트 (`public/admin/js/app.js`):
- 통계 카드(총 응시/평균/80점 이상)에서 "(녹음 제외)" 라벨 제거 — 점수 카드는
  비-녹음 모드만 집계되는 사실은 동일하나 표시 단순화 (혼선 방지).
- `modeBars` 녹음숙제 행 — 점수·평균 대신 `N회 출제 중 M회 제출 (X%)` 표시 +
  `(정성 평가)` 부연. 진행률 바 색상은 제출률 기준 (80%↑ 초록 / 50%↑ 호박 /
  그 외 빨강 / 0회 회색).
- 🎤 녹음숙제 정성 평가 카드 신규 (추세 아래 amber 톤) — `recordingComment` 만 표시.
  카테고리 평균 점수 부연 줄 제거.
- PDF/인쇄(`printGrowthReport`)는 모달 본문 `innerHTML` 그대로 사용 → 자동 반영.

### 작업 규칙 추가 (2026-06-01 이어서)

- **점수 비공개 정책은 PDF·정성 코멘트까지 일관 적용** — 녹음숙제처럼 점수
  객관성/신뢰도 문제로 학생 노출 제한이 있는 경우, 학원장 화면에서만 수치 노출.
  학부모 공유 PDF·AI 정성 코멘트에서도 수치 노출 금지. AI 가 내부 추론용으로는
  점수 사용 OK, 출력은 정성 표현만 하도록 프롬프트 강제.
- **default 날짜 필터 = 빈 값(전체) 이 직관적** — "어제" 같은 기본값은 그 날짜에
  데이터 없으면 빈 목록으로 학원장 혼선. 페이지네이션 + cursor 더보기가 있는 한
  default 는 전체로 두는 게 직관적.
- **server 생성 doc 의 즉시 표시 = 현재 필터 유지하며 limit N 재fetch** —
  surgical insert 가 어려운 server-created doc 케이스에서 loadX 전체 reload
  (필터 reset) 대신 cache 무효 + 부분 재fetch + 부분 재렌더 패턴.

### 파일 크기 / SW 캐시 (2026-06-01 이어서)
- `public/admin/js/app.js`: 메시지 default 날짜 + sendMessage 재fetch + 리포트
  카드 정성 카드·modeBars 녹음 분기
- `api/growth-report.js`: SYSTEM_PROMPT 수치 금지 강화 + recordingAssigned/Submitted
  + `_aggregateRecordingQuality` 헬퍼 + RESPONSE_SCHEMA recordingComment
- SW 캐시: `kunsori-v620` → `kunsori-v623`

---

## 2026-06-02: 학생 등록 진단 + 스펠 자동 제출 제거(회귀 1회) + 비번 입력 검증·토글

SW v623 → v627 (~5 commit + 진단 3 스크립트). 학원장 "엑셀 등록 학생 비번이
잘못 박힌다" / "스펠 오타 수정 못한다" / "비번 변경했다는데 안 된다" 보고
연쇄. 끝에 학원장 자가검증을 위한 비번 입력 UI 강화.

### 1) 학생 등록 진단 — 코드는 100% 정상, 학생 입력 측 원인

학원장 보고: 6/1~6/2 엑셀 등록 학생 5명 모두 비번 잘못 박혔다고 학생들이
보고. 학원장이 5명 전원 '123456' 으로 재설정.

**코드 점검 결과** ([app.js:6808](public/admin/js/app.js#L6808)):
- `importStudentExcel` 이 password `'123456'` literal 박음 — 분기·조건·변수
  없음. 엑셀의 어떤 컬럼도 password 자리에 들어갈 코드 경로 0
- `saveStudent` (단건) 은 학원장 모달 입력 그대로 전송, default 박는 코드 없음
- `api/createStudent.js:195` 는 클라가 보낸 password 그대로 `auth.createUser`
  에 전달, 강제 변환 없음

**진단 스크립트 3개 신규** (`scripts/diag/`):
- `check-students-2026-06-02.js` — 6/1~6/2 등록 학생 전수 진단 (Firestore +
  Auth + usernameLookup 일치 + 글로벌 username 충돌 + lastSignInTime)
- `check-password-resets-today.js` — 오늘(KST) `tokensValidAfterTime` 갱신된
  Auth user (비번 재설정 추정) 전수 검출
- `check-student-by-name.js <name>` — 이름으로 단건 진단 (재호출 가능 헬퍼)

**진단 결과** (6/1~6/2 5명 + 6/2 추가 등록 2명 + 5/12 등록 최하진):
- 5명 모두 Firestore + Auth + usernameLookup + customClaims 정상, providerData=password 박힘
- **3명 (최하진·목선우·홍주영) 재설정 전에 정상 로그인 성공 기록** → 등록 시점
  비번 '123456' 이 정상 박혔던 **확정 증거**. 김가윤·안유나는 재설정 후 로그인
  이라 직접 검증 불가지만 같은 createStudent 경로 + 다른 3명 정상 박힘 → 동일
  하게 박혔다고 강하게 추정
- 추가 김기헐 케이스: 등록 40초 만에 lastSignInTime 박힘 → 엑셀 등록 로직
  정상 작동 한 번 더 확인

**결론**: 엑셀 등록 코드 100% 정상. "비번 안 됨" 보고의 실제 원인은
학생 입력 오류(한글 자판 / 1↔l / O↔0 / 한 번 입력 후 잊음) 또는 학생이 본인
"내 정보" 에서 비번 변경 후 잘못 기억. Firebase Auth 는 비번 plain 값 어디
에도 저장 안 함(해시만) → admin SDK 도 검증 불가, **변경 이력 조회도 불가**
(`tokensValidAfterTime` 은 최신 한 번만 박힘).

**createStudent.js 의 createdBy 미기록 발견** (`api/createStudent.js:227`):
`callerUid` 를 idToken 검증 단계에서 알고 있는데 `users` doc 에 안 박음.
영향 — 등록 경로(엑셀 vs 단건) 추적 불가, 다중 학원장 시 책임 추적 0,
super 운영 분석 부재. 옛 데이터는 백필 불가(서버 로그 휘발). 즉시 수정
대신 멀티학원장(부원장) 도입 시점에 묶어서 진행하기로 보류.

### 2) 단어시험 스펠링 자동 제출 제거 + v624 회귀 fix

학원장 보고 "마지막 알파벳 입력하면 자동으로 다음 문제 — 오타 수정 시도조차
못 함". 옛 동작: input 이벤트 핸들러가 `_vqScheduleAutoSubmit` 400ms
debounce 로 자동 `vqNext()` 호출.

**v624** (`4a5a880`): input 핸들러의 자동 진행 트리거 제거 + 함수 정의
(`_vqScheduleAutoSubmit`/`_vqCancelAutoSubmit`/`_vqAutoSubmitTimer`) 삭제.
제출 버튼/Enter/타이머 만료 경로만 유지.

**v625 회귀 fix** (`5032de0`): `_vqAutoNext()` 안에 `_vqCancelAutoSubmit()`
호출 한 줄이 남아있어 ReferenceError → 모든 vocab 시험 유형(스펠/말하기/MCQ)
에서 제출 → 피드백 표시 → 버튼 disabled → throw → **다음 문제로 진행 안 됨**.
호출 잔존 1줄 제거. v624 운영 시간 ~30분~1시간 동안 응시 중이던 학생은 시험
도중 멈췄을 가능성.

회귀 표본 — Phase 6D 회귀(고아 `};`, 2026-04-21) 와 같은 부류. 함수 제거 시
호출처 grep 누락이 ReferenceError 회귀를 부름. 작업 규칙으로 정립.

### 3) 비밀번호 입력 검증·토글 강화 (v626 / v627)

학원장 "최하진이 본인이 '123456' 으로 바꿨다는데 안 됨" 추가 보고. 학생앱
"내 정보" 에 비번 변경 input 1개(`#myNewPw`, `type=password`)만 있어 학생도
자기 입력값 자가검증 불가. 학원장 [학생 추가/수정] 모달의 `#sPw`/`#euPw` 도
같은 상태라 학원장도 본인 입력 확인 불가.

**v626** (`da35412`): 비번 보기 토글 + 일치 검증
- 학생앱 [내 정보]: `#myNewPw` 👁 토글 + **비번 확인 input** (`#myNewPwConfirm`)
  + 일치 검증 (불일치 시 "비밀번호 확인이 일치하지 않습니다" 토스트). 비번 입력
  시 확인 칸 자동 노출, 비우면 다시 숨김
- 학원장 [학생 추가] `#sPw`: 👁 토글
- 학원장 [학생 정보 수정] `#euPw`: 👁 토글
- `togglePwVis(id, btnEl)` 함수 학생앱·학원장 앱에 각각 분리 등록 (별개 파일)
- 모바일 자동완성 차단: `autocomplete="new-password"` + `autocorrect=off`
  + `autocapitalize=off` + `spellcheck=false` (Phase 6C 학습 적용)

**v627** (`ea46529`): 학원장 피드백 — 위치 fix + 도식 SVG 교체
- 학생앱 토글 위치 — `form-row` 절대좌표(`top:38px`) → input wrapper
  `position:relative` + `top:50%/transform` → **input 정중앙** 정렬
- 👁 이모지 → **stroke SVG** (Heroicons/Lucide 풍 eye / eye-off, `color:#999`)
- 토글마다 eye ↔ eye-off 상태 전환 (현재 보기/숨김 상태 직관 표시)
- 학생앱 진입 시 `_SVG_EYE` 로 토글 아이콘도 리셋 (이전 진입 잔존 방지)

### 작업 규칙 추가 (2026-06-02)

신규:
- **함수 제거 시 호출처 전수 grep 필수** — 정의만 지우고 호출 잔존하면
  ReferenceError 회귀. 옛 함수명 grep → 0건 확인 후 commit. `node --check`
  만으로는 못 잡음(syntax 통과). Phase 6D 고아 `};` (2026-04-21) 와 같은
  부류의 함수 경계 회귀, 작업 후 grep 검증 routinize 필수.
- **Firebase Auth 비번 plain 조회 불가 — admin SDK 도 해시만** — `passwordHash`
  필드가 listUsers 에서 보이지만 해시고 plain 복원 불가. 비번 변경 이력도
  `tokensValidAfterTime` 한 번(최신 변경만) 만 박혀 옛 이력 추적 불가.
  학원장이 비번 알고 싶으면 **본인 재설정 + 직접 입력값 학생 안내** 외 방법
  없음. Cloud Audit Log 활성화 시 가능하나 기본 비활성.
- **`providerData=password` 박힘 = 비번 등록됨 (값 무관)** — admin SDK 진단
  `getUser.passwordHash` 는 일반적으로 미반환(listUsers 만 반환). 진단 시
  `providerData` 가 password 가지면 Auth 계정에 비번 박혀 있는 것 확정.
  passwordHash 부재로 "비번 안 박혔다" 오판 금지.
- **CRITICAL UI input 은 자가검증 가능해야** — `type=password`(●●●) 만으로는
  사용자(학생·학원장 모두) 입력 오류 자가검증 불가. 비번 같은 critical input
  은 👁 토글로 보기 옵션 + (새 비번 설정 경로면) 비번 확인 input 으로
  이중 입력 검증. 모바일 자동완성 차단 속성 동봉 필수.
- **inline onclick 의 `this` 전달로 button 상태 토글** — `togglePwVis(id, this)`
  처럼 button 자체를 전달하면 함수에서 `btnEl.innerHTML` 교체로 아이콘 상태
  토글 가능. data-* 패턴 안 써도 안전(`this` 는 엔티티 디코딩 영향 0).
  사용자 데이터 박는 경우만 data-* 필요.

### 진행률 / 파일 크기 / SW 캐시 (2026-06-02)

- 학생 등록 진단: ~100% (코드 정상 확인 + 진단 스크립트 3개 정착)
- 단어시험 스펠 UX: ~100% (자동 제출 제거 + 회귀 fix)
- 비번 입력 검증·토글: ~100% (학생앱·학원장 앱 3곳 적용, 위치·아이콘 정비)
- createStudent.js createdBy 미기록: **미해결** (멀티학원장 도입 묶음 대기)
- 멀티테넌시·결제·말하기·성장리포트·AI Generator: 변동 없음
- Phase 5 출시 준비: 0%

파일:
- `public/_app.html`: 학생앱 비번 변경 영역 input wrapper + 확인 칸 + SVG 토글
- `public/js/app.js`: 자동 제출 트리거·함수 제거(v624) + `_vqAutoNext` 호출
  잔존 제거(v625) + `togglePwVis` + 비번 검증·확인 칸 show/hide + SVG 상수
- `public/admin/js/app.js`: `togglePwVis` + sPw/euPw input wrapper + SVG 토글
- `scripts/diag/check-students-2026-06-02.js` / `check-password-resets-today.js`
  / `check-student-by-name.js`: 신규 진단 (untracked, .gitignore)
- SW 캐시: `kunsori-v623` → `kunsori-v627`

**다음 세션 후보** (변동):
1. `createStudent.js createdBy/createdMethod` 박기 — 멀티학원장 도입 묶음
2. 학생앱 비번 변경 후 재확인 안내 강화 (변경 직후 토스트에 마지막 시도값 안내 등)
3. Phase 5 출시 준비 (도메인·약관·결제 PG)
4. v1.0 Polish 사이클 / super reads P2 (변동 없음)

---

## 2026-06-03: 학생 등록·비번 변경 actor 추적 (users.passwordHistory + createdBy)

SW v627 → v628 (1 commit `366518c`). 6/2 작업 직후 학원장 재보고 — "6명
비번 재설정 후에도 또 안 된다는 보고가 반복돼 누가 어떻게 바꿨는지 모르겠음.
확인 수단이 부족함." → actor 추적 시스템 도입. 2026-06-02 미해결로 남겼던
`createdBy` 미기록 동시 해결.

### 1) 데이터 모델 결정 — A안(inline) vs C안(별도 컬렉션)

학원장이 알고 싶은 정보: 학생 단건 단위로 "누가 마지막으로 비번 바꿨나".

| 모델 | 저장 위치 | 학생 단건 진단 | 학원 전체 통계 | 무한 누적 | Rules |
|------|---------|-------------|-------------|---------|-------|
| **A** | `users/{uid}.passwordHistory` 배열 | 빠름 (1 read) | 학생 N명 fetch | 1MB doc 한계 | 변경 0 |
| C | `passwordChangeLog/{logId}` 컬렉션 | 1 쿼리 | 1 쿼리 + 인덱스 | 무제한 | 신규 |

C안 진짜 가치 영역(super 운영 분석·이상 패턴 탐지)이 현재 운영 규모(학원
4·학생 100·super 분석 비활성)에 비해 과한 인프라 → **A안 채택**. C안은
Phase 5 출시 후 멀티학원장 도입 시 검토.

### 2) 박는 위치 3곳 + actor 5종

`users/{uid}.passwordHistory: [{ ts, actor, actorUid, actorName, method }]`

| actor 라벨 | 배지 색 | 박는 위치 | 트리거 |
|-----------|--------|----------|--------|
| `admin_excel` 엑셀 일괄 등록 | 청록 | createStudent.js | body.method='excel' |
| `admin_single` 학원장 등록 | 청록 | createStudent.js | body.method='single' |
| `admin_reset` 학원장 재설정 | 호박색 | updateStudentPassword.js | 학원장 role |
| `super_reset` super 재설정 | 빨강 | updateStudentPassword.js | super_admin role |
| `student_self` 학생 본인 변경 | 보라 | 학생앱 saveMyInfo | client updatePassword 성공 후 |

`createdBy` (이전 미해결): createStudent.js 가 `caller.uid` 를 idToken 검증
단계에서 알고 있으나 users doc 에 안 박던 문제 — 동시 해결. 추가 박힘:
`createdBy: caller.uid` / `createdByName` / `createdMethod: 'excel'|'single'`.

학원장 앱 클라 측 method 필드 전달:
- `importStudentExcel` payload 에 `method: 'excel'`
- `saveStudent` payload 에 `method: 'single'`

### 3) 학원장 UI — [학생 정보 수정] 모달 이력 섹션

결제 이력 버튼 위에 "🔑 비밀번호 변경 이력 (최근 10건)" 섹션 신설.

- `_pwHistoryHtml(history)` 헬퍼 — 시각(YYYY-MM-DD HH:MM KST) + actor 라벨
  배지 + actor 이름. 최근 10건만(`slice(-10).reverse()`), 최신 우선
- `_PW_ACTOR_LABELS` 상수 — actor 5종 라벨·배지 색 매핑
- ts 정규화 — Firestore Timestamp(`toDate()`) 또는 `{seconds}` 또는 raw Date
  모두 처리
- 옛 데이터(2026-06-03 이전 등록분)는 history 없음 → "변경 이력 없음 (옛
  학생은 2026-06-03 이전 등록분이라 데이터 없음)" 안내

### 4) 기술 디테일

- **arrayUnion 안에서 serverTimestamp sentinel 불가** — `FieldValue.serverTimestamp()`
  는 top-level 만 가능, 배열 element 안에서는 throw. 대안: `new Date()`.
  서버시간 정확도는 100ms 이내 (Vercel function ↔ Firestore 같은 리전)라 운영 OK
- **Rules 변경 0** — `users/{uid}` update rule 이 이미 `isOwner(userId)` 허용
  → 학생 본인 own doc 의 passwordHistory 필드 update 가능. admin 도 같은 학원이면 OK
- **이력 기록 실패는 비치명적** — `try/catch` 로 감싸고 console.warn. 비번 변경
  자체는 이미 성공한 상태라 throw 하면 학원장이 "비번 변경 실패" 로 오해할 수 있음

### 5) 데이터 보존 한계 (정직 표시)

- **2026-06-03 이전 등록 학생**: history 비어 있음 — Firebase Auth 자체 로그도
  admin SDK 로 못 봐서 백필 불가
- **Firebase Console Cloud Audit Log** 활성화 시 옛 변경 이력 일부 추적 가능
  하나 기본 비활성이고 별도 GCP 설정 필요 — 현재는 보류
- 옛 학생 이력 필요해지면 별도 작업 (학원장이 학생당 한 번씩 [재설정] 누르게
  안내하면 그 시점부터 박힘)

### 작업 규칙 추가 (2026-06-03)

신규:
- **actor 추적이 필요한 사용자 액션은 시작부터 박기** — 안 박으면 옛 데이터
  백필 불가 (서버 로그 휘발). createStudent 의 createdBy 미기록을 두 달
  방치하다 학원장 보고 받고 도입한 사례. Phase 1 부터 박았으면 더 일찍 가능
- **inline 배열 vs 별도 컬렉션 결정 기준** — 사용 빈도(1명 단건 vs 전체 통계)
  + 운영 규모 + 운영 단계. 학생 단건 진단이 주 사용 패턴이면 inline 배열
  충분. 전체 통계·이상 탐지가 필요해지는 시점(super 분석 활성·멀티학원장
  도입)이 별도 컬렉션 트리거
- **arrayUnion 안 serverTimestamp 불가** — `FieldValue.serverTimestamp()` 는
  top-level field 만 가능. 배열 element 안에서는 throw. 운영용 actor 추적
  같은 ms 단위 정확도 불필요한 케이스는 `new Date()` 충분
- **`users/{uid}.isOwner` Rule 활용 — 학생 본인이 자기 doc 의 일부 필드 write
  필수 케이스** — 학생 본인 비번 변경 이력처럼 클라가 직접 박아야 하는 데이터는
  Rules 변경 없이 isOwner 허용 활용. 학생 본인이 own doc 임의 update 가능한
  설계라 신규 필드 추가만으로 작동

### 진행률 / 파일 크기 / SW 캐시 (2026-06-03)

- **학생 등록·비번 변경 actor 추적: ~100%** (3 박는 위치 + 학원장 UI + 5종 actor)
- **createStudent createdBy 미기록 해결** (2026-06-02 미해결 항목 해결)
- 멀티테넌시·결제·말하기·성장리포트·AI Generator: 변동 없음
- Phase 5 출시 준비: 0%

파일:
- `api/createStudent.js`: createMethod / createdBy / createdByName / passwordHistory 첫 항목
- `api/updateStudentPassword.js`: passwordHistory arrayUnion (actor=admin_reset/super_reset)
- `public/admin/js/app.js`: saveStudent/importStudentExcel payload method + editStudent 모달 이력 섹션 + `_pwHistoryHtml`/`_PW_ACTOR_LABELS`
- `public/js/app.js`: saveMyInfo updatePassword 후 passwordHistory arrayUnion (actor=student_self)
- SW 캐시: `kunsori-v627` → `kunsori-v628`

**다음 세션 후보** (변동):
1. 학생앱 비번 변경 후 재확인 안내 (직후 토스트에 마지막 시도값 안내 등)
2. 옛 학생(2026-06-03 이전 등록분) 비번 한번씩 재설정 안내 — history 시드
3. C안(별도 컬렉션) 도입 — Phase 5 후 멀티학원장 도입 시 검토
4. Phase 5 출시 준비 / v1.0 Polish 사이클 / super reads P2 (변동 없음)

---

## 2026-06-03 (이어서): 이모지 → SVG 점진 교체 Phase 1+2 (회귀 2회 + 회복)

SW v628 → v633 (~7 commit). 학원장 "이모지를 SVG 로 바꾸고 싶다" 요청
→ 빈도순 점진 교체 (B안). 87종 864회 중 상위 8종 95곳 변환. 도중
회귀 2회 (헬퍼명 충돌 / string literal 안 변환) 거치고 안정.

### 1) 사전 검토 — A/B/C 옵션 비교

- A: 일괄 전체 교체 (12~20h, 회귀 위험 큼)
- **B: 빈도순 점진 교체** (Phase 별 2~3h) ← 채택
- C: 구조 UI 만 + 토스트는 유지 (3~4h)

빈도순 점진 — Phase 별 시각 검수 가능 + 회귀 시 작은 범위 진단.
showToast 메시지·안내문·주석 안 이모지는 정책상 보류 (시각 임팩트 유지).

### 2) Phase 1 — ✏ 🗑 36곳 (`10a7b4f`, v629)

- 학원장 app.js: ICONS 객체 + icon() 헬퍼 추가 (기존 _SVG_EYE 옆 통합)
- ICONS 2종: edit / trash (Lucide stroke-only)
- 자동 변환 스크립트 — `>X ` / `>X<` 패턴 매칭 → `${icon('xxx')}` / SVG
- 학원장 app.js 17곳 + 학원장 _app.html 18곳 + 모달 헤더 1곳 = 36곳

### 3) Phase 2 — 🔍 💾 ⚙ 🎤 📋 📝 58곳 (`50707ac`, v630)

- ICONS 6종 추가: search / save / settings / mic / clipboard / pen
- 학생앱 app.js 에도 동일 ICONS + icon() 헬퍼 박음 (Phase 2 첫 도입)
- 학원장 app.js 42곳 + 학원장 _app.html 11곳 + 학생앱 app.js 2곳 +
  학생앱 _app.html 3곳 = 58곳

### 4) v630 회귀 1차 — 헬퍼명 `icon` ↔ 지역 변수 `icon` 충돌 (`cdbea3a` revert)

학원장 로그인 후 home 화면 안 뜸 (로딩 무한).

원인: 학원장 app.js 안 이미 `const icon = ...` **지역 변수 6곳** —
결제 상태별·시험 종류별·진행 상태별·문제 타입별·페이지 별칭 아이콘
분기. 그 함수 안에 Phase 2 변환으로 `${icon('save')}` 가 들어가면서
지역 변수 `icon`(문자열) 이 전역 헬퍼 `icon()` 함수를 가림 → 문자열을
함수 호출 → TypeError → home 렌더 throw.

Phase 1 만에서는 우연히 이 함수들 안에 ✏🗑 변환이 안 들어가 발현 X.
Phase 2 의 🎤📝📋 변환에서 발현.

대응: 학원장 app.js + _app.html 만 pre-Phase 1 상태 (5219930) 로 복귀,
학생앱은 정상이라 유지.

### 5) v632 재시도 — 헬퍼명 icon → iconSvg (`a93c06e`)

해결: 헬퍼 이름 `icon` → **`iconSvg`** 로 변경. 지역 변수 `icon` 6곳과
네이밍 스페이스 분리.

- 학생앱: `function icon` → `iconSvg` + 사용 2곳 + `window.iconSvg`
- 학원장 app.js: ICONS + iconSvg 헬퍼 재추가
- 학원장 Phase 1+2 변환 재실행 — `${iconSvg(...)}` 형태로 90곳
  (app.js 62 + html 28)

### 6) v632 회귀 2차 — string literal 안 `${iconSvg('mic')}` (`dcba5a3`, v633)

F12 콘솔 `SyntaxError: Unexpected identifier 'mic'`. 모듈 로드 자체 실패.

원인: 자동 변환 스크립트가 `>X<` 패턴을 컨텍스트 무관하게 변환했는데,
일부 위치가 **일반 string literal (`'...'`) 안**이었음. `'...>${iconSvg('mic')}</span>'`
형태에서 `'` 가 `iconSvg(` 의 `'` 와 만나 string 닫힘 → `mic` 가 식별자.

`node --check` 가 못 잡은 이유: script 모드 default 라 ES module 의
`import` 가 있어도 통과시킴. **`node --input-type=module --check` 필수**.

수정 4곳 모두 `condition ? '...emoji...' : ''` ternary 패턴:
- line 685: 캘린더 사이드 시험 list speaking 배지
- line 3476: 결제 템플릿 탭 편집 배지
- line 3690: 결제 템플릿 탭 변경 onclick 내 편집 배지
- line 5319: 시험 list 테이블 row speaking 배지

`'...'` → `` `...` `` 백틱 변경.

### 7) Phase 1+2 최종 완료 — 95곳

| 영역 | 변환 |
|------|------|
| 학원장 app.js | ✏ 10 + 🗑 10 + 🔍 2 + 💾 8 + ⚙ 1 + 🎤 9 + 📋 12 + 📝 10 = 62 |
| 학원장 _app.html | ✏ 8 + 🗑 9 + 🔍 1 + 💾 2 + ⚙ 1 + 🎤 1 + 📋 3 + 📝 3 = 28 |
| 학생앱 app.js | 🎤 1 + 📝 1 = 2 |
| 학생앱 _app.html | 🎤 2 + 📋 1 = 3 |
| **합계** | **95** |

### 작업 규칙 추가 (2026-06-03 이어서)

신규:
- **자동 변환 + import 파일 = `node --input-type=module --check` 필수** —
  기본 `node --check` 는 script 모드 default 라 ES module 전용 syntax
  (import 등 있어도) 우회. Phase 6D 회귀 (고아 `};`, 2026-04-21) 와
  같은 부류의 module-aware 검증 필요. sed/자동 변환 후 즉시 실행
  필수. 함수 경계 회귀와 함께 routinize 할 검증 단계.
- **자동 변환 패턴은 단일 string 컨텍스트 가정 X** — `>X ` / `>X<`
  같은 단순 패턴이 **백틱 안에 있을 거라 가정**하면 일반 string
  literal (`'...'`) 안 케이스에서 string 닫힘으로 syntax 깨짐. 변환
  전 pre-check 로 single quote / double quote 안 case grep 필요. 또는
  변환 결과를 백틱으로 강제 변환 (string → template literal).
- **헬퍼 함수명은 도메인 변수와 충돌 안 하는 식별자 선택** — `icon`,
  `data`, `item`, `result` 같은 일반 단어는 도메인 코드 곳곳에 지역
  변수로 박힐 가능성 높음. 헬퍼는 접두사(`_`, `app`, `svg`) 또는
  명사+동사 조합(`iconSvg`, `dataLoad`) 으로. 충돌 시 가까운 scope 의
  지역 변수가 우선 → 전역 헬퍼 호출 시 TypeError. node --check 통과
  도 안 잡힘 (runtime 만 발현). v630 회귀 표본.
- **점진 교체에서 헬퍼 헬퍼 추가 ↔ 변환은 같은 commit** — 헬퍼만 먼저
  박고 변환은 후속 commit 으로 미루면, 헬퍼 추가 commit 만 배포된
  상태에서 다음 commit 누락 시 변환된 곳이 헬퍼 호출 못 함. 또는
  reverse — 헬퍼 추가 안 한 상태에서 변환만 박혀도 헬퍼 없음 throw.
  같은 commit 으로 묶기. (이번 작업은 Phase 1 commit 에서 헬퍼+변환
  같이 박아 OK 했음, 다음 점진 교체 시도 유지)

### 진행률 / 파일 크기 / SW 캐시 (2026-06-03 이어서)

- **이모지 → SVG Phase 1+2: ~100%** (8종 95곳, 회귀 2회 거쳐 안정)
- Phase 3+ 후보: ✓ ✅ ⚠ ❌ ✕ 💡 📊 🤖 📤 📄 🔀 📚 🔊 🔒
- 학생 등록·비번 변경 actor 추적: ~100% (변동 없음)
- 멀티테넌시·결제·말하기·성장리포트·AI Generator: 변동 없음
- Phase 5 출시 준비: 0%

파일:
- `public/admin/js/app.js`: ICONS + iconSvg 헬퍼 + 변환 62곳 + ternary
  4곳 백틱 fix
- `public/admin/_app.html`: 인라인 SVG 28곳
- `public/js/app.js`: ICONS + iconSvg 헬퍼 + 변환 2곳
- `public/_app.html`: 인라인 SVG 3곳
- SW 캐시: `kunsori-v628` → `kunsori-v633`

**다음 세션 후보** (변동):
1. **Phase 3 — 빈도순 다음 8종** (✓ ✅ ⚠ ❌ ✕ 💡 📊 🤖) — string literal 안
   변환 회피 + 헬퍼명 충돌 검증 routinize
2. ✏️ 🗑 등 type icon map 안 단일 이모지 (vocab/fill_blank/recording)
   — 표시 위치에서 별도 SVG 처리
3. placeholder 안 이모지 — JS 로 별도 처리 (`<input placeholder>` 안
   SVG 박기 불가)
4. 학생앱 비번 변경 후 재확인 안내 / 옛 학생 history 시드 / Phase 5 출시
   준비 (변동 없음)

---

## 2026-06-03 (이어서 2): 이모지 → SVG Phase 3 (회귀 0)

SW v633 → v634 (1 commit `a21ddd1`). Phase 1+2 회귀 2회 교훈 다 적용해
회귀 0 으로 통과.

### 적용 — ✕ 📊 🤖 💡 42곳

ICONS 4종 추가: x / chart / bot / lightbulb (학원장 + 학생앱 동일).

| 영역 | 변환 |
|------|------|
| 학원장 app.js | ✕ 8 + 📊 4 + 🤖 4 + 💡 4 = 20 |
| 학원장 _app.html | 📊 4 + 🤖 1 = 5 |
| 학생앱 app.js | 📊 2 + 🤖 3 + 💡 4 = 9 |
| 학생앱 _app.html | ✕ 6 + 💡 2 = 8 |
| **합계** | **42** |

사전 fix 2곳 (학원장 single quote 컨텍스트):
- `status.innerHTML = '<span>🤖 동음이의어 분석 중...'` → 백틱
- `status.innerHTML = '<span>🤖 AI 청크 분할 + 한글뜻 생성 중...'` → 백틱

Phase 1+2 회귀 교훈 적용:
- ✅ 변환 전 single quote pre-grep → 2곳 발견·사전 백틱 변환
- ✅ 변환 후 `node --input-type=module --check` (학원장 + 학생앱) 둘 다
- ✅ 헬퍼명 iconSvg 유지 (지역 변수 충돌 회피)

Phase 3 제외 (의도):
- ✓ (74회) — 컨텍스트 다양해 다음 Phase 단독 묶음 권장
- ✅ ⚠ ❌ (총 111회) — showToast/안내문 비중 큼

### 누적 (Phase 1+2+3 완료)

| Phase | 종류 | 횟수 | 회귀 |
|-------|------|------|------|
| 1 | edit, trash | 36 | - |
| 2 | + pen, search, save, settings, mic, clipboard | 58 | 2 |
| 3 | + x, chart, bot, lightbulb | 42 | 0 |
| **합계** | **12종** | **136** | |

### 다음 Phase 결정 — 학원장 실사용 피드백 대기

학원장이 v634 사용해보고 미관·동작 확인하는 시간 확보. SVG 가 stat-icon
같이 size 가정이 있는 위치에서 모양·정렬 어색할 수 있어 실사용 피드백이
다음 Phase 디자인 보정에 더 유용. 다음 Phase 전 학원장 의견 수렴.

### 파일 크기 / SW 캐시 (2026-06-03 이어서 2)

- `public/admin/js/app.js`: ICONS 4종 추가 + 변환 20곳 + 사전 백틱 fix 2곳
- `public/admin/_app.html`: 인라인 SVG 5곳
- `public/js/app.js`: ICONS 4종 추가 + 변환 9곳
- `public/_app.html`: 인라인 SVG 8곳
- SW 캐시: `kunsori-v633` → `kunsori-v634`

**다음 세션 후보** (변동):
1. **학원장 v634 실사용 피드백 수렴** → Phase 4 디자인 보정 (size·정렬·
   stat-icon 등 CSS 가정 깨진 곳 확인)
2. Phase 4 후보:
   - ✓ 단독 (74곳, 작은 마커 다양)
   - 📤 📄 🔀 📚 4종 (~60곳, button 중심 안전)
3. type icon map 안 단일 이모지 / placeholder 안 이모지 (미해결)
4. 학생앱 비번 변경 후 재확인 안내 / 옛 학생 history 시드 (변동 없음)

---

## 2026-06-03 (이어서 3): AI Generator 운영 fix — 녹음 page 정렬·sn nextSerial·선택 카운트·신규 page 즉시 표시

SW v634 → v640 (6 commit). 학원장 6/1·6/2 Danny CH1·CH2 녹음숙제 보고
출발 → 정렬·serialNumber·UI 강조·캐시 갱신 4가지 영역 종합 정비.

### 1) 녹음 page 정렬 — chapterOrder/order → serialNumber → title 숫자 (v635→v636)

학원장 보고: Danny CH2 녹음 fullText 가 Page 3 시작 → Page 2 끝. 학원장
의도와 다른 page 순서로 결합. pageCount > 1 인 다른 의심 세트 9개 추가
발견 (DR Suess Mr Brown 3건 등).

#### v635 (`7906483`): 1차 수정 — chapterOrder/order → serialNumber

원인 발견: `_qgBuildRecordingSet` 의 정렬 키 (`chapterOrder × 10000 +
order`) 가 page 스키마와 어긋남. AI OCR 등록 코드 (line 8510) 가
`serialNumber` 만 박고 `order`/`chapterOrder` 안 박음 → 모든 page
정렬 키 0 → Firestore fetch 순서 (random).

수정: sort 키를 `serialNumber` 로 통일 (학원장 OCR 화면·다른 사용처와 일치).

#### v636 (`1c40acf`): 2차 수정 — serialNumber 도 부적합 → title 숫자

사용자 보고: v635 배포 후에도 Danny CH2 의 Page 3 가 먼저. 진단 결과
**chapter 내 sn 중복** — Page 1·2 sn=1·2, Page 3·4 sn=1·2 (다른
chapter 인데 sn 같음).

원인: OCR 의 `nextSerial = _genPages.filter(p => !p.chapterId).length`
가 "미배정 page 수" 기반. chapter 배정 후 미배정 0 → 다음 OCR 또 1
부터 시작. 같은 chapter 안에 sn 1, 2, 1, 2 중복.

최종 수정: `title` 안 마지막 숫자 추출 정렬 (학원장이 직접 입력·보는
라벨 기반).
- "Page 1" → 1
- "Page 14" → 14
- "CH1 본문 2" → 2
- "예봉 중 3 본문 CH1" → 1

학원장 OCR 한 순서·serialNumber·chapter 배정 순서 무관, 학원장이
화면에서 보는 라벨 그대로 정렬. 옛 잘못 결합된 11개 세트는 학원장이
[수정] → 재생성하면 즉시 복구.

### 2) nextSerial 학원 전체 max + 1 통일 (v637 `0636bdb`)

위 §1 의 nextSerial 결함 자체를 fix (B안). 박는 위치 3곳 통일:
- OCR (`runGenOcr`): `_genPages.filter(p => !p.chapterId).length` →
  Firestore 학원 전체 page max sn fetch
- 수동 추가 (`genDoCreatePage`): `_genPages.reduce(max)` (lazy 라 부정확) →
  Firestore fetch
- 병합 (line 8760): 위와 동일

신규 헬퍼 `_genFetchMaxSerialNumber()` — 학원 전체 page fetch + max 계산.
default 학원 156 page 라 1 read 부담 0. 1000+ 누적 시 인덱스
(`academyId+serialNumber DESC`) 추가 검토.

옛 156 page sn 재배치 안 함 (운영 위험). 새 page 부터 학원 전체 sequence
(157, 158, ...). 학원장 OCR 화면 표시 정렬은 옛 page 그대로 + 새 page
뒤로 추가.

### 3) AI Generator page 선택 카운트 배지 (v638 `a56e002`)

학원장 보고: chapter 바뀌고 다른 page 선택 시 이전 chapter 선택 누적된
채로 문제 세트 생성. 카운트 표시가 작아 인지 어려움.

수정:
- 선택 0개: 평범 (font 12px, weight 700, teal 텍스트)
- **선택 1개+: teal 배경 + 흰 글자 + bold 배지** (font 14px, weight 800,
  padding 2px 9px, border-radius 11px)

신규 헬퍼 `_qgSelCountStyle(n)` — `_qgRender()` 정적 HTML 과
`_qgUpdateSelCount()` 동적 update 양쪽에서 동일 적용.

chapter 클릭 시 자동 선택 해제 안 함 (학원장이 여러 chapter page 묶고
싶은 케이스 있음). 배지로 인지 강화만.

### 4) AI Generator page 작업 후 즉시 표시 (v640 `6aab665`)

학원장 보고: AI OCR 에서 active chapter/book 안 page 를 수정·병합·신규
OCR 후 결과가 화면에 즉시 안 보임. 새로고침 또는 화면 이탈 복귀 시에만
보임.

**중간 잘못된 fix 시도** (사용자 거부): chapter active 시 신규 page 를
그 chapter 에 자동 배정. 사용자 정정 — **미배정/active 분리는 의도된
분리**. 진짜 문제는 active 영역 안 page 작업 후 미갱신.

원인 (재진단): `addDoc` 시 `serverTimestamp()` 는 Firestore 서버 측에만
박힘. 클라 `_genPages.push` 시 createdAt 없는 객체 push → `_genRecentSort`
의 `t()` 가 `x?.createdAt?.toMillis?.()` 호출 → undefined → 0 fallback
→ 최근순 정렬 (default) 에서 가장 뒤로 밀려 페이지네이션 1쪽에 안
나옴. 새로고침 시 Firestore 의 실제 serverTimestamp 다시 fetch → 정상.

수정:
- `_genRecentSort` 의 `t()` 가 Firestore Timestamp / Date / number 모두 처리
- runGenOcr / genDoCreatePage / 병합: push 시 `createdAt: new Date()` 박음
- genSavePage (수정): `updatedAt: serverTimestamp()` + 클라 mutate 시
  `updatedAt: new Date()` 박음 → 수정한 page 가 최근순 맨 위로

`new Date()` 는 Firestore 가 다음 fetch 시 실제 serverTimestamp 로 자동
교체 (클라 캐시는 placeholder, doc 자체는 정확).

### 작업 규칙 추가 (2026-06-03 이어서 3)

신규:
- **`serverTimestamp()` 는 클라 캐시에 안 들어옴 — push 시 `new Date()`
  placeholder 필요** — addDoc 시 `serverTimestamp()` 는 Firestore 서버
  측에만 박힘. 클라가 그 직후 메모리 캐시에 push 하는 객체에는
  createdAt/updatedAt 이 sentinel 객체로만 남거나 아예 없음. 최근순
  정렬 함수가 millis 변환 못 해서 0 fallback → 가장 뒤로 밀림 → 화면에
  안 보임. push 시 `new Date()` 박아 Firestore 의 실제 serverTimestamp
  와 분리 (다음 fetch 시 자동 교체).
- **정렬 함수 `t()` 는 Firestore Timestamp + Date + number 모두 처리** —
  `_genRecentSort` 같은 정렬 헬퍼가 `x?.createdAt?.toMillis?.()` 만
  체크하면 클라 측 Date 객체 못 처리. `if (v instanceof Date) return
  v.getTime()` 분기 추가 필요. 다양한 데이터 소스 (Firestore fetch / 클라
  optimistic push) 모두 정상 처리.
- **`nextSerial` 학원 전체 max 기반** — `_genPages.filter(p =>
  !p.chapterId).length` 같은 미배정 수 기반은 chapter 배정 후 0 으로
  reset → 다음 OCR 같은 sn 시작 → chapter 내 중복. Firestore 학원 전체
  fetch + max + 1 이 정답 (read 1번이지만 _genPages lazy 라 정확). 옛
  중복 데이터는 둠 (학원 운영 영향 없음, 새 page 부터 정상 sequence).
- **정렬 키는 학원장 의도 라벨이 가장 안전** — `serialNumber` 도
  nextSerial 결함으로 chapter 내 중복 가능. 학원장이 직접 입력·보는
  `title` 안 마지막 숫자 (`/\d+/g` 매칭) 가 학원장 의도와 가장 일치.
  Page 등록 순서·chapter 배정 순서 무관, 라벨 그대로 정렬.
- **미배정/active 영역 분리는 의도된 분리 — 자동 배정 X** — 학원장이
  active chapter/book 에서 보지 않은 신규 page 를 그 영역에 자동 배정
  하면 학원장 흐름 깨짐. 미배정은 별도 영역으로 명확히 분리 (학원장이
  이동 작업으로 배정). 신규 page 가 화면에 안 보이는 진짜 원인은 sort
  결함 (위 항목).

### 진행률 / 파일 크기 / SW 캐시 (2026-06-03 이어서 3)

- **AI Generator 운영 정비: ~100%** (녹음 정렬·sn·UI·캐시 갱신)
- 옛 11개 잘못 결합 세트: 학원장 [수정] → 재생성으로 복구 (운영 측 작업)
- 이모지 → SVG Phase 1+2+3: 변동 없음 (12종 136곳)
- 학생 등록·비번 actor 추적: 변동 없음
- Phase 5 출시 준비: 0%

파일:
- `public/admin/js/app.js`: 녹음 sort 키 (title 숫자) + nextSerial fetch
  헬퍼 + 선택 카운트 배지 헬퍼 + _genRecentSort Date 처리 + push 시
  createdAt/updatedAt: new Date() 4곳
- SW 캐시: `kunsori-v634` → `kunsori-v640`
- 신규 진단 스크립트:
  · `scripts/diag/check-danny-recording.js`
  · `scripts/diag/check-recent-recording-tests.js`
  · `scripts/diag/check-recent-recording-sets.js`
  · `scripts/diag/check-recording-set-order.js`

**다음 세션 후보** (변동):
1. 옛 11개 잘못 결합 세트 학원장 재생성 안내 (운영)
2. Book/Chapter 도 같은 createdAt 캐시 패턴일 가능성 — 학원장 보고 받으면 동일 fix
3. 이모지 → SVG Phase 4 (학원장 v634 실사용 피드백 후)
4. Phase 5 출시 준비 / v1.0 Polish 사이클 (변동 없음)

---

## 2026-06-04~05: 메시지 리셋·진단·SW 자동 reload·1일 로그아웃 폐기

SW v640 → v642 (~3 commit + 진단 다수). 학원장 운영 보고 연쇄.

### 1) 메시지 [↻ 리셋] 버튼 (v641 `295a8d3`)

학원장 보고: [♻ 재활용] 으로 초안/발송 메시지 로드 시 발송 대상도
같이 박혀 다른 학급에 보낼 때 일일이 해제 번거로움.

새 메시지 작성 카드의 "📤 대상" 라벨 옆에 `[↻ 리셋]` 버튼 추가:
- `msgResetTargets()` — `_picker.targets = []` + box/summary 재렌더 + onChange 콜백

### 2) Gemini 결제 진단 + 6월 녹음 미평가 1건 재평가 (v640 그대로)

Vercel 로그 `[check-recording] prepayment depleted` 알림 → 학원장 충전.

진단 — 6월 녹음 73건 중 영향 단 1건 (나유안 6/5 07:59):
- 신규 진단 스크립트 `find-orphan-recordings-jun.js`,
  `check-recording-errors-jun.js` 두 종 추가
- 첫 진단(`inProgress.recordings` 만 확인)이 case C 분류 1건 놓침 —
  실제 필드는 `inProgress.rounds` 였음
- 재진단(`latestErrorMessage` 기준) — 결제 실패 케이스 1건만 명확 식별
- 영향 학생 1건은 학원장 앱 기존 [🔁 재평가] 버튼으로 처리 (코드 작업 불필요)
- 오은지(1/2 회차만 녹음 후 종료, 결제 무관) — 학생 재응시 안내

### 3) 진도체크 "Missing or insufficient permissions" — 삭제된 doc 의 Rules

학원장 보고: 진도체크 시험별 패널에서 일부 시험 학생카드 안 나오고
"로드 실패: Missing or insufficient permissions" 에러.

학원장 통찰("이름 바꾸거나 삭제했을지도") 으로 진단 — 옛 두 시험 doc
실제 Firestore 에서 사라짐 (학원장이 옛 시험 삭제 + 같은 이름에 "뜻시험"
추가하여 재출제). 시험 목록 메모리 캐시에 옛 testId 남아있어 클릭 시
`getDoc(genTests/{삭제된testId})` → Rules 거부.

**Firestore Rules 함정**: `allow read: if ... resource.data.academyId
== myAcademyId()` 의 컬렉션 doc 이 삭제되면 `resource.data` 는 `null`.
`null.academyId == 'default'` 평가 시 Firestore 는 **`not-found` 가
아니라 `permission-denied` 반환**. 학원장 화면에는 헷갈리는 권한 에러
표시. 해결책 후보 (안 함, 사용자 무시 결정):
- `tpToggleTestProgress` catch 에서 "Missing or insufficient permissions"
  메시지 감지 시 "시험이 삭제됨" 안내로 분기 (5분 작업)
- 시험 삭제 시 메모리 캐시 surgical 갱신 보강

### 4) SW 자동 reload + 1일 로그아웃 폐기 + footer 버전 (v642 `6191a1a`)

학원장 보고: 학생들이 시험 도중 강제 로그아웃되어 처음부터 다시.

원인: `onAuthStateChanged` 의 **24시간 자동 로그아웃 정책**
(`elapsed > ONE_DAY_MS`). 학생이 어제 ~20시 로그인 → 학생앱 PWA
백그라운드 유지 → 오늘 시험 시작 시 페이지 reload 발생하면 24시간
경계 도달 → 강제 로그아웃.

이민서 진단으로 패턴 확정 — Auth lastSignInTime 20:01:33 (재로그인),
첫 응시 20:09:57 (재로그인 8분 후). "튕김 → 재로그인" 흐름.

**옛 정책의 진짜 의도**: 베타 운영 중 SW 업데이트 강제. 1일 후
재로그인 흐름이 페이지 reload 발생시켜 결과적으로 SW 업데이트 적용.
**우회 메커니즘이 본질 (사용 중 로그아웃) 부작용 만든 케이스**.

해결 — 표준 PWA 패턴으로 전환:

**A. SW 자동 reload (시험 중 보호)**
- `sw.js` 의 activate 가 SW_UPDATED postMessage 보냄 (기존)
- 학생앱 listener 추가 — 시험 화면(vocabQuiz/unscrambleQuiz/recAiQuiz/
  readingMcq/fillBlank/result) 이면 `_pendingReload` 박고 대기
- `show()` wrapper hook — 시험 끝나 다른 화면 전환 시 자동 reload
- 첫 메시지 무시(`_swInitialMsg`) + sessionStorage 가드(`_swReloadDone`)
  무한 reload 방지

**B. 1일 자동 로그아웃 정책 폐기**
- `onAuthStateChanged` 의 `elapsed > ONE_DAY_MS` 블록 제거
- Firebase Auth 의 자동 토큰 갱신 + persistence 'local' 에 맡김
- 보안: 학원장 비번 재설정 → `tokensValidAfterTime` 갱신으로 강제
  invalidate 가능 (충분)

**C. Footer 버전 표시**
- 학원장 앱은 헤더 Version 이미 있음. 학생앱은 없었음
- `<body>` 끝에 fixed bottom-right `.version` div 박음 — 모든 화면
  우측 하단 자그마한 반투명 배지
- head 인라인 script 의 SW 캐시값 표시 로직 그대로 활용 (`document
  .querySelectorAll('.version')` 핸들러)
- 학원장이 학생 폰 잠깐 보면 최신 버전 여부 즉시 판단

### 5) 디바이스 정보 추적 부재 (별도 작업 보류)

학원장 보고: Galaxy Quantum 4 일부 학생 녹음 안 됨 (다른 폰으로 해결).
김재율 학생 디바이스 정보 요청 — 우리 코드가 user-agent/OS/브라우저
박지 않아 직접 진단 불가. Vercel 로그 30분 보존이라 사후 추적 어려움.

별도 작업 후보 (사용자 보류) — 학생앱 시험 진입 화면에 디버그 정보
(브라우저·OS·MediaRecorder 지원 형식·마이크 권한) 자그마하게 표시.
다음 보고 시 학원장이 학생 폰 잠깐 보고 핀포인트 진단 가능.

### 6) 메시지 긴급 발송 옵션 (v643 `11a1466`)

학원장 보고 (6/4 21:15 → 23:00 도착 진단의 후속): 평소엔 일반 알림으로
보내되 시험 안내·긴급 안내만 즉시 도착하도록 옵션화.

- 새 메시지 작성 카드 첨부 영역 아래 `[긴급 발송]` 체크박스 추가
- payload.urgent boolean 으로 서버 전달, 발송 성공 후 자동 해제
  (다음 메시지는 default 평소 정책)
- `api/sendPush.js` FCM message 분기:
  - urgent=true: `android.priority='high'` / `apns-priority=10` /
    `webpush Urgency=high` → 절전·doze 모드에서도 즉시 wake
  - urgent=false: 옛 동작 그대로 (Urgency=normal 명시)
- `pushNotifications.urgent` boolean 박음 (이력·통계용)

학원장 운영 가이드 — 긴급 발송 권장 케이스:
- 시험 시작 안내 / 약속 / 결석 통보 / 시간 변동 알림 등 즉시 인지 필요
- 일반 안내 (숙제·자료·일정 변경)는 체크 X — 평소 normal 우선순위로
  배터리 절약

### 작업 규칙 추가 (2026-06-04~05)

신규:
- **사용 중 강제 로그아웃은 정책 자체가 부적절** — 1일 자동 로그아웃
  같은 시간 기반 정책은 학생/학원장 활동 도중 끊김 부작용. PWA 표준
  은 토큰 자동 갱신 + 명시적 로그아웃 + 비번 재설정으로 invalidate.
  "비활성 N일" 같은 표현이 "마지막 로그인 후 N일" 보다 합당.
- **우회 메커니즘이 본질 부작용 만들면 표준 패턴으로 전환** — 1일
  로그아웃이 SW 업데이트 강제용 우회였던 케이스. 본질 해결책 (SW
  controllerchange/message 활용 자동 reload) 로 전환하면 부작용 0 +
  업데이트 자동 적용. 우회는 보통 임시 해결책 — 본질 fix 시점 미루지
  말기.
- **삭제된 doc 의 Rules 평가 = permission-denied (not-found 아님)** —
  `resource.data.academyId == myAcademyId()` 같은 Rules 가 doc 삭제
  후엔 `null.academyId` 평가하다 권한 거부 반환. 학원장 화면에
  헷갈리는 에러 메시지 노출. catch 에서 메시지 감지·분기로 친화 안내
  가능 (선택 작업).
- **카카오 인앱 브라우저 강제 외부 브라우저 변경 불가** — 카카오톡 측
  정책이라 보내는 측에서 못 바꿈. 대신 학생앱의 인앱 감지 배너
  (이미 구현) + [📋 주소 복사] 버튼으로 학생이 정식 브라우저로 옮기게
  유도. PWA 설치 후엔 인앱 문제 영구 해결.
- **Vercel 로그는 진단용 휴발 자원** — 30분 보존이라 사후 진단 어려움.
  중요 정보(에러·실패 케이스)는 Firestore (userCompleted.latestError-
  Message 등) 박아 영구 보존. 학원장 보고 즉시 진단 가능.
- **FCM 우선순위는 발송자 옵션으로 노출** — normal default + 학원장이
  케이스별 high 선택. 모든 메시지를 high 로 하면 "긴급 인상" + 디바이스
  배터리 부담. 평소는 normal (절전 우선) + 긴급은 high (즉시 wake) 분리가
  학원 운영 + 학생 배터리 둘 다 만족.

### 진행률 / 파일 크기 / SW 캐시 (2026-06-04~05)

- 학생앱 정책 정비: ~100% (1일 로그아웃 폐기 + SW 자동 reload + footer 버전)
- 메시지 관리 UX: ~100% (리셋 버튼)
- 6월 녹음 결제실패 진단·복구: ~100% (1건만 영향, 재평가 완료)
- 진도체크 삭제 시험 권한 에러: 사용자 무시 결정
- 이모지 → SVG Phase 1+2+3: 변동 없음 (12종 136곳)
- AI Generator 운영 정비: 변동 없음
- Phase 5 출시 준비: 0%

파일:
- `public/_app.html`: footer fixed `.version` div + 로그인 화면의 .version 제거
- `public/admin/_app.html`: 메시지 작성 카드 [↻ 리셋] 버튼 + [긴급 발송] 체크박스
- `public/admin/js/app.js`: `msgResetTargets` 함수 (picker reset) + sendMessage payload.urgent
- `public/js/app.js`: SW message listener + `_pendingReload` + `_trySwReload` +
  show() hook + 1일 로그아웃 블록 제거
- `api/sendPush.js`: FCM message priority 분기 (android/apns/webpush) +
  pushNotifications.urgent 필드
- SW 캐시: `kunsori-v640` → `kunsori-v643`
- 신규 진단:
  · `scripts/diag/find-orphan-recordings-jun.js`
  · `scripts/diag/check-recording-errors-jun.js`

**다음 세션 후보** (변동):
1. 디바이스 정보 표시 (학생앱 디버그) — Quantum 4 같은 보고 즉시 진단용
2. 진도체크 삭제 시험 친화 메시지 (catch 분기) — 학원장 보고 누적 시
3. 학원장 앱 SW 자동 reload + 모달 보호 — 학원장도 같은 표준 패턴
4. Book/Chapter `createdAt` 클라 캐시 (Generator) — 학원장 보고 받으면
5. Phase 5 출시 준비 / v1.0 Polish 사이클 (변동 없음)

---

## 2026-06-07: FCM 알림 — 학원 로고 + 알림 권한 배지

SW v644 → v646 (~2 commit). 학원장 보고 연쇄.

### 1) FCM 알림 아이콘 학원 로고 적용 (v645 `c619fc9`)

학원장 보고: default 학원(큰소리영어) 메시지 알림에 LexiAI 로고 박힘.

진단 (1차 오인): `academies/default.logo192Url` (top-level) 빈 값 →
"학원장이 로고 안 업로드" 추정. 학원장 정정 — "192/512 자동 생성
해 놓았는데?"

재진단 (2차): 로고는 **`academies/default.branding.logo192Url`** 에
정상 박혀있음 (Storage URL). 학원장이 자동 생성 코드 설계대로
원본 PNG 업로드 시 192/512 리사이즈 후 박는 흐름 — 데이터 정상.
**`sendPush.js` 가 그 필드를 안 봄** 이 진짜 원인.

수정:
- `sendPush.js` 가 발송 시 `academies/{callerAcademyId}` fetch
- `branding.logo192Url` 있으면 그것 사용, 없으면 absolute default
  (`https://raloud.vercel.app/icons/icon-192.png`)
- `webpush.notification.icon + badge` 둘 다 학원 로고로

옛 메시지 알림은 변경 없음 (이미 발송된 건). 다음 메시지부터 적용.

### 2) 알림 권한 상태 배지 (v646 `0ed4f36`)

학원장 후속 질문: "양쪽 폰 등록한 학생인데 한쪽만 알림 받는다는데
어떻게 확인하나"

설명: F12 로 `Notification.permission` 확인 가능 — 그러나 휴대폰엔
F12 없음. 학원장 정확 지적.

해결 — 학생앱 [내 정보] 화면에 알림 권한 상태 배지:
- 허용됨: 초록 "알림 ON · 학원 메시지 받음"
- 거부됨: 빨강 "알림 거부됨 · 폰 설정에서 켜기"
- 미설정: 노랑 "알림 미설정 [알림 받기]" 버튼 — 클릭 시
  `Notification.requestPermission()` + 허용 시 `doRegisterToken()`
  자동 호출 (FCM 토큰 즉시 등록)
- 브라우저 미지원: 회색

학원장이 학생/학부모 폰의 [내 정보] 화면 잠깐 보면 색상으로 즉시
판단. 미설정이면 그 자리에서 버튼 한 번으로 권한 요청.

### 작업 규칙 추가 (2026-06-07)

신규:
- **학원별 데이터는 `branding` 서브객체 우선 확인** — 학원 로고 같은
  브랜딩 데이터는 `academies/{id}.branding.logo192Url` 형식. top-level
  필드만 보면 빈 값으로 잘못 진단. sendPush 등 학원 데이터 사용하는
  서버 코드는 `branding.*` 경로로 fetch.
- **휴대폰 학생/학부모 진단 도구는 학생앱 내부에 박아야** — F12 같은
  PC 전용 도구는 휴대폰에서 못 씀. 학원장이 학부모에게 "설정 들어가서
  확인하세요" 안내하는 부담 줄이려면 학생앱 내부에 진단 정보 표시.
  [내 정보] 화면에 알림 권한 상태 배지가 표본. 향후 다른 진단 정보
  (디바이스 모델·브라우저·MediaRecorder 지원 등) 도 같은 패턴으로
  추가 가능.
- **FCM 알림 아이콘 URL 은 절대 URL** — `/icons/icon-192.png` 같은
  상대 URL 은 일부 브라우저에서 학생앱 origin 으로 해석 못 함.
  `https://raloud.vercel.app/...` 또는 Storage `https://storage.googleapis.com/...`
  같이 절대 URL 권장. 학원 로고는 Storage URL 이라 자동 절대.

### 진행률 / 파일 크기 / SW 캐시 (2026-06-07)

- 메시지 시스템: ~100% (긴급 발송 + 학원 로고 + 알림 권한 진단)
- 학생/학부모 진단 도구: ~50% (알림 권한 배지 추가, 디바이스 정보는 보류)
- 멀티테넌시·결제·말하기·성장리포트·AI Generator: 변동 없음

파일:
- `api/sendPush.js`: academies/{id}.branding.logo192Url fetch + iconUrl 분기
- `public/_app.html`: 내 정보 화면 #notifPermBadge div 추가
- `public/js/app.js`: `_renderNotifPermBadge` + `requestNotifPerm` +
  goMyInfo 에서 호출
- SW 캐시: `kunsori-v644` → `kunsori-v646`

**다음 세션 후보** (변동):
1. 학생앱 디바이스 정보 진단 표시 — 알림 권한 배지 옆에 브라우저·OS·
   MediaRecorder 지원 정보 (Quantum 4 같은 보고 시 즉시 진단)
2. FCM 토큰 만료 진단 — 학원장 앱에서 학생별 토큰 상태 확인 (없음/만료)
3. 발송 이력에 긴급 발송 배지 표시 (지금은 doc 에 박기만 함)
4. Phase 5 출시 준비 (변동 없음)

---

## 2026-06-08: 시험 도중 뒤로가기 보호 (3단계 진화) + 학생별 진도체크 fix

SW v646 → v650 (~5 commit). 학원장 보고: "학생이 시험 도중 휴대폰 하단
뒤로가기 누르면 처음부터 다시 풀어야 함" → 시험 종료 흐름 통일.
김유진 학생의 진도체크 표시 불일치 → 학원장 앱 결함 발견.

### 1) 뒤로가기 보호 1차 (v647 `4d5388d`) — 단순 confirm

popstate 핸들러에 `_isInExam(curId) && curId !== 'result'` 분기. 모달
"시험을 종료할까요?" → 확인 시 `history.back()`.

문제: 학생이 [확인] 누르면 history.back() 호출 → 다시 popstate 발화
→ 또 보호 분기 진입 → 또 모달. **무한 모달**. + result 화면 별도 id
아니라 시험 화면(vocabQuiz/recAiQuiz) innerHTML 만 결과로 교체하는
구조라 `curId === 'result'` 조건이 절대 true 안 됨.

### 2) 뒤로가기 보호 2차 (v648 `044ad3c`) — flag + dataset 마커

- `_examExitAllowed` flag — 학생 확인 시 true 박고 history.back()
  호출. 다음 popstate 시 1회 우회 후 false 리셋. 무한 루프 차단
- `screen.dataset.stage = 'result'` 마커 — `_vqRenderResult` /
  `_rv2RenderResult` 가 결과 렌더 시 박음. popstate 가 그 마커로
  결과 보기 중 판정. show() wrapper 에서 시험 시작 시 자동 리셋

문제: 모달 자체는 단순 confirm 1개. 그러나 학원장이 사용 중인 X 버튼
(quitVocab 등) 은 2단계 모달 — (1) 중단 확인 + (2) **저장 여부 확인**.
학생이 [중단]/[저장] 선택해서 진행 내용을 다음 진입에 이어풀 수
있어야 하는데 뒤로가기는 그게 빠짐.

### 3) 뒤로가기 보호 3차 (v649 `442bfb8`) — quit 함수 호출 (정답)

popstate 가 자체 모달 만들지 말고 **시험 유형별 quit 함수 호출**로
완전 위임:

```js
const _EXAM_QUIT_FNS = {
  vocabQuiz: 'quitVocab',
  recAiQuiz: 'quitRecAi',
  unscrambleQuiz: 'quitUnscramble2',
  readingMcq: 'quitReadingMcq',
  fillBlank: 'quitFillBlank',
};
```

quit 함수가 (1) 중단 확인 → (2) 저장 여부 확인 → `_vqSaveProgress()`
같은 진행 저장 + `goHome()` 까지 일괄 처리. `_examExitAllowed` flag
도 제거 (불필요 — quit 의 `goHome` 이 `show()` 호출이라 popstate
우회 분기 안 거침).

→ 뒤로가기와 X 버튼이 **완전 동일 흐름**. 학생이 어느 경로로 종료해도
같은 모달·같은 저장 옵션.

### 4) 학생별 진도체크 `excludedUids` fix (v650 `335ee50`)

학원장 후속 보고: "김유진 학생이 어스본반 시험에서 [✕ 제외] 했는데도
학생별 진도체크 화면에 그 시험이 표시됨. 학생앱에도 보이고 학원장
조회해도 보임 — 캐시가 아닌 다른 원인 같음."

진단:
- 시험 `excludedUids` 에 김유진 uid 정상 박힘
- 김유진 userCompleted/scores 없음 (진입 흔적 0)
- 학생앱 코드 (line 2152) 의 `excludedUids` 필터 정상
- 학원장 시험별 진도체크 `_progTestAssignedTo` (line 16871) 정상
- **학원장 학생별 진도체크 `_progFetchStudentTests` (line 16670)**
  의 dedup merge 단계에서 **`excludedUids` 필터 누락** ← 진짜 원인

학생앱 측은 fresh fetch 시 정상 (시뮬레이션 확인). 김유진 화면에서
보이는 건 옛 메모리 캐시 — 페이지 재진입 시 자동 해결.

수정: forEach 안에서 td.excludedUids 가 uid 포함하면 push skip. 세
화면 (학생앱·시험별·학생별 진도) excludedUids 정책 일관 적용.

### 작업 규칙 추가 (2026-06-08)

신규:
- **화면 id 만으로 화면 상태 구분 불가 시 명시 마커** — innerHTML 교체
  패턴 (`_vqRenderResult` 가 vocabQuiz 화면 안 결과 UI 박음) 에선
  curId 가 '시험 중' 과 '결과 보기 중' 모두 같음. `dataset.stage =
  'result'` 같은 명시 마커로 구분. 화면 진입 시 wrapper 에서 reset
  필수 (다음 시험 시작 시 마커 잔존 방지).
- **시험 종료 흐름은 quit 함수가 유일 진실** — quitVocab 등이 중단
  확인 + 저장 여부 모달 + 진행 저장 + goHome() 일괄 처리. 다른 종료
  경로 (뒤로가기·자동 종료) 도 그 함수 호출로 위임. popstate 가
  자체 confirm 만들면 X 버튼과 다른 흐름 → 학생 혼란 + 저장 흐름 누락.
- **excludedUids 체크는 3곳 동일 정책 필수** — 학생앱 시험 목록 /
  학원장 시험별 진도 / 학원장 학생별 진도 — 어느 한 곳 빠지면 화면별
  표시 불일치 (학생앱에선 안 보이는데 학원장 진도엔 보임 등). 신규
  진도/시험 표시 화면 추가 시 excludedUids 필터 routinize.
- **사용자 보고 → 점진 정확화 패턴** — 첫 fix (v647) 이 회귀로 추가
  fix 트리거 (v648, v649). 사용자가 "확인 누르면 화면 유지" / "저장
  여부 모달 안 뜸" 등 정확한 증상 알려주면 그것만 보고 추정 → 진단
  비용 최소. 1차 답변에서 "X 버튼이 quitVocab 호출하는데 뒤로가기는
  안 함" 같은 코드 측 비교가 빨리 정답으로 가는 길.

### 진행률 / 파일 크기 / SW 캐시 (2026-06-08)

- 학생앱 시험 보호: ~100% (3단계 진화 후 안정)
- 학원장 학생별 진도체크 excludedUids 정합성: ~100%
- 메시지 시스템 (긴급/로고/리셋/알림 배지): 변동 없음
- 멀티테넌시·결제·말하기·녹음숙제·AI Generator: 변동 없음

파일:
- `public/js/app.js`: popstate 핸들러 + `_EXAM_QUIT_FNS` 매핑 + `_isInExam`
  + dataset.stage 마커 (_vqRenderResult / _rv2RenderResult)
- `public/admin/js/app.js`: `_progFetchStudentTests` dedup merge 에
  excludedUids 필터 추가
- SW 캐시: `kunsori-v646` → `kunsori-v650`

**다음 세션 후보** (변동):
1. 학생앱 디바이스 정보 진단 표시 (Quantum 4 같은 보고 즉시 진단)
2. FCM 토큰 만료 진단 — 학원장 앱에서 학생별 토큰 상태
3. 발송 이력에 긴급 발송 배지 표시
4. 학원장 앱 SW 자동 reload + 모달 보호 (학원장도 표준 패턴)
5. Phase 5 출시 준비 (변동 없음)

---

## 2026-06-12: 로그인 비번 토글·친화 에러 + 시험 제한시간 학원장 설정

SW v650 → v655 (~5 commit). 학원장 보고 — minmini 학생 `auth/too-many-requests`
에러 + 단어시험 제한시간 학원장 조절 불가 → 일련 작업.

### 1) 학생 로그인 비번 토글 + 친화적 에러 메시지 (`a4b41ae`, v652)

minmini(이성민) 학생 `auth/too-many-requests` 에러 보고. 진단 결과 학생이 비번
잘못 입력 누적으로 Firebase 가 일시 차단. 학원장이 6/9 + 6/10 두 번 재설정 했는데
계속 보고 → 학생 비번 자가검증 수단 부재가 본질.

- 학생앱 `[로그인]` 화면 비번 input 에 👁 SVG 토글 추가 (myInfo 패턴 동일)
  · `togglePwVis('passwordInput', this)` 재사용
  · `autocomplete=current-password` 유지, padding-right:42px 로 토글 공간 확보
- `_friendlyAuthError(e)` 신규 — Firebase Auth 8개 code 한국어 매핑
  · `auth/too-many-requests` → "비밀번호를 여러 번 잘못 입력해서 일시 차단됐어요.
    30분 후 다시 시도하거나, Wi-Fi ↔ LTE 를 바꿔서 시도해보세요."
  · `auth/invalid-credential` / `wrong-password` → "비밀번호가 틀렸습니다."
  · `auth/user-not-found` / `network-request-failed` / `user-disabled` /
    `invalid-email` / `internal-error` 별도
  · default: "로그인 중 문제가 생겼어요. 잠시 후 다시 시도해주세요. (코드)"
- 옛 catch 분기 `e.code==='auth/invalid-credential'?'...':'오류: '+e.code` 폐기

### 2) minmini 학생 비번 진단 — 코드 변경 없음

학원장 후속 질문 3가지 진단:
- "이전 비번 알 수 있나" — 불가. Firebase Auth 는 SHA-256 + salt 해시만 저장,
  admin SDK 도 plain 복원 불가. `passwordHistory` 는 시각·actor 만 기록.
- "비번 잘못 입력 아닌 다른 요인?" — 가장 의심 1순위: **폰 비번 관리자 자동완성이
  옛 비번 박음**. 학원장 옆에서 1회 로그인 성공 (6/9 14:57:11) 후 학생이 본인 폰에서
  들어가려 할 때 자동완성이 옛 값 덮어쓰기 → 학생은 본인 입력으로 인지. **새 👁
  토글로 즉시 자가검증** 가능. FCM 토큰 0건 (학생 평소 [내 정보] 안 들어감) 확인.
- "다른 네트워크 시도하면 풀리는 이유" — Firebase 차단 기준 = IP + 계정 조합.
  Wi-Fi ↔ LTE 는 외부 IP 달라 차단 카운트 초기화. Credential Stuffing 방어용.
- "지금 차단 풀렸나" — 직접 조회 불가 (Firebase 가 rate limit 상태 노출 API 없음).
  비번 재설정은 보통 rate limit 도 해제 → 6/10 19:50 재설정으로 풀렸을 가능성 큼.

### 3) 단어시험 제한시간 출제 옵션 (회귀 1회 후 확정, `75f07e1` v653)

학원장 요청 — 출제 모달 시험명 뒤·통과점수 앞에 제한시간 input. 사용자 결정
과정 (3 iteration):
- 1차 추측: 단일 입력 default 10초, 범위 3~120
- 2차 정정: "기본값을 mcq 10초 스펠 30초로 수정, 범위는 5~120" → 형식별 2 입력
  으로 확장 (timeLimitMcqSec / timeLimitSpellSec)
- 3차 단순화: "mcq, 스펠 구분하지말고 동일하게 기본 30초로 하나로 통일" →
  단일 입력 default 30 으로 최종

확정 spec:
- 출제 모달 grid `1fr 90 85 95 140` (단어시험 5칸) — 통과점수·출제문제수 폭 축소
- `tpTimeLimit` input (number, default 30, min 5, max 120, title 툴팁)
- `vocabOptions.timeLimitSec` 저장 (이때 시점에는 vocabOptions 안에 박았음)
- 학생앱 `_vqStartTimer` — opts.timeLimitSec 우선, 미박힘 시 옛 룰 (MCQ 10/스펠 30)
  폴백. 형식 무관 단일 값.

### 4) startVocab opts carry 누락 버그 (`7b01e0e`, v654)

학원장 보고 — 60초 설정 출제한 시험이 학생앱(김기헌 본인 계정 Harry 테스트)에서
30초로 보임. 진단:
- Firestore 데이터 정상 (`vocabOptions.timeLimitSec: 60` 확인)
- 학생앱 `_vqStartTimer` 정상 (opts.timeLimitSec 읽음)
- **버그 위치**: `startVocab` 의 opts 객체 구성 (line 4751-4758) 이 명시적 픽킹
  패턴 (`format/mcqRatio/en2koRatio/shuffleQ/shuffleChoices/speakingStrictness`)
  인데 신규 `timeLimitSec` 추가 안 함 → `_vqState.opts.timeLimitSec = undefined`
  → 옛 룰 폴백 → 30초로 보임

원인: 이전 v653 작업이 `_vqStartTimer` 만 수정하고 그 위 단계 opts 구성 누락.
명시적 픽킹 패턴은 신규 필드 추가 시 반드시 위치 같이 갱신 필요.

수정: `opts` 객체에 `timeLimitSec: _raw.timeLimitSec` 추가.

### 5) 빈칸채우기·언스크램블 학원장 설정으로 확장 (옵션 A, `494773e`, v655)

학원장 "다른 시험은 어떻게 설정?" 질문 → 현황 정리 후 옵션 A 채택. 통합 작업:

- 출제 모달 — `cfg.testMode === 'vocab'` 단일 분기 → `['vocab','fill_blank',
  'unscramble'].includes(cfg.testMode)` 로 일반화. UI 패턴 동일
- **데이터 모델 통일 — doc 최상위 `timeLimitSec`** 으로 박음
  · 단어시험: vocabOptions 에서 빼고 최상위로 (학생앱은 vocabOptions.timeLimitSec
    폴백 유지 — 어제 출제분 호환)
  · 빈칸/언스크램블: 최상위 신규
- 학생앱 3개 타이머 동일 패턴:
  · `_vqStartTimer`: `test.timeLimitSec ?? opts.timeLimitSec` 우선
  · `_uqStartTimer`: `_uqState.test.timeLimitSec` 우선, default 30
  · `_fbStartTimer`: `_fbState.test.timeLimitSec` 우선, default FB_TIME_PER_Q(30)
- UI arc 비율 계산도 동적 total 받도록 — `_uqUpdateTimerUI(total)` / `_fbUpdateTimerUI(total)`

### 시험 유형별 제한시간 현황 (2026-06-12 종료 기준)

| 유형 | 제한시간 | 위치 |
|------|---------|------|
| 단어시험 (vocab) | 학원장 설정 (default 30) | test.timeLimitSec |
| 빈칸채우기 (fill_blank) | 학원장 설정 (default 30) | test.timeLimitSec |
| 언스크램블 (unscramble) | 학원장 설정 (default 30) | test.timeLimitSec |
| 객관식 본문이해 (mcq) | 없음 (무제한) | — |
| 객관식 문법 | 없음 (무제한) | — |
| 해석하기 (subjective) | 없음 (무제한) | — |
| 녹음숙제 (recording) | 학원장 설정 (min/maxDurationSec) | 별도 모델 |

### 작업 규칙 추가 (2026-06-12)

신규:
- **Firebase Auth `too-many-requests` = IP + 계정 + 디바이스 조합 차단** —
  Credential Stuffing 방어용. 다른 네트워크(Wi-Fi ↔ LTE 외부 IP 변경) 로 시도
  하면 우회. admin SDK 로 rate limit 상태 직접 조회 불가. 비번 재설정이 보통
  해제 트리거. 학원장 안내: 30분 대기 / 네트워크 변경 / 학생 폰 비번 관리자
  자동완성 옛 값 의심.
- **`type=password` input 은 자가검증 어려움 — 토글 필수** — 비번 입력은 사용자
  (학생/학원장 모두) 본인 입력값 자가검증 불가가 본질 문제. 폰 비번 관리자
  자동완성이 옛 값 덮어쓰기도 알 수 없음. 👁 토글 + (새 비번 설정 경로면) 확인
  input 으로 이중 입력. critical input 표준.
- **명시적 픽킹 패턴 opts/options 객체에 신규 필드 추가 시 반드시 위치 같이
  갱신** — `startVocab` 의 opts 구성 (`{format, mcqRatio, en2koRatio,
  shuffleQ, ...}`) 처럼 명시적 필드만 picking 하는 객체는 신규 필드 추가 시
  Firestore 데이터에 박혀도 carry 안 됨 → 학생앱이 신규 값 못 봄 → 출제 설정
  무용. 신규 필드 박는 위치 grep 후 모든 picking 경로 같이 갱신.
- **시험 옵션 데이터 모델 — doc 최상위 통일 우선** — vocab 만 `vocabOptions`
  서브객체 안에 박는 패턴은 다른 시험 유형 확장 시 비대칭. 시험 유형 무관 공통
  옵션 (`timeLimitSec` 등) 은 doc 최상위. 유형별 고유 옵션 (vocab format,
  recording recordingCount 등) 만 서브객체 유지. 학생앱은 폴백 분기로 옛 시험 호환.

### 진행률 / 파일 크기 / SW 캐시 (2026-06-12)

- 로그인 UX (비번 토글·친화 에러): ~100%
- 시험 제한시간 학원장 설정 (vocab·fill_blank·unscramble): ~100%
- minmini 진단: 코드 변경 없음 (학생 폰 자동완성 의심, 학원장 안내로 해결)
- 멀티테넌시·결제·말하기·녹음숙제·AI Generator·메시지: 변동 없음

파일:
- `public/_app.html`: 로그인 화면 비번 input wrapper + 👁 SVG 토글
- `public/js/app.js`: `_friendlyAuthError` + startVocab opts carry +
  `_vqStartTimer` / `_uqStartTimer` / `_fbStartTimer` 학원장 설정 우선
- `public/admin/js/app.js`: 출제 모달 grid 일반화 + `tpTimeLimit` input +
  tpPublish 최상위 timeLimitSec 박음
- SW 캐시: `kunsori-v650` → `kunsori-v655`

**다음 세션 후보** (변동):
1. minmini 학생 후속 관찰 (자동완성 정리 안내 효과 확인)
2. 학생앱 디바이스 정보 진단 표시 (Quantum 4 같은 보고 즉시 진단)
3. FCM 토큰 만료 진단
4. 객관식·해석에도 시간 제한 옵션 추가 (옵션 B — 학원장 요청 시)
5. Phase 5 출시 준비 (변동 없음)

---

## 2026-06-13 ~ 18: 메시지 다중 첨부 + 진도체크 캐시 surgical + 학생앱 시험목록 캐시·헤드체크

SW v655 → v658 (3 commit). 학원장 보고 3건 연쇄 — 메시지 첨부 1개 한계
+ 출제 직후 진도체크에 안 보임(5시간 stale) + Firestore reads 비용 절감.

### 1) 메시지 첨부 다중 파일 지원 (`f0fc480`, v656)

학원장 보고 — 메시지 작성 시 첨부 1개만 가능. 공지(다중)와 비대칭.

- 학원장 핸들러 — `_msgPendingAttach` (단일) → `_msgPendingAttaches[]` (배열).
  공지 패턴(`_noticePendingAttaches` / `_noticeUploadAll`) 그대로 mirror
- 신규 함수: `_msgRenderAttaches` / `_msgAcceptFile` / `msgRemoveAttach` /
  `_msgUploadAll` / `_msgClearAttaches`. 옛 `_msgUploadAttachIfAny` /
  `msgClearAttach` 폐기
- `_app.html`: `<input multiple>` + 첨부 리스트 영역 + 안내 문구 갱신
- `sendMessage`: payload `attachment` (단수) → `attachments` (배열)
- `api/sendPush.js`: `attachments[]` 우선, 옛 `attachment` 단수 폴백.
  pushNotifications / userNotifications doc 양쪽 박음. 최대 10개 안전망.
  옛 코드 호환 위해 첫 번째를 `attachment` 도 동시 박음 (이중 저장)
- 학생앱 표시 — `_notifAttachments(n)` 헬퍼: `n.attachments` 배열 우선,
  옛 `n.attachment` 단수 폴백. 알림 패널·showNotifModal 양쪽 다중 표시
- 학원장 발송이력 배지 — `첨부` → `첨부 N` (1개면 그대로, 2+ 카운트)

### 2) 출제 직후 진도체크 캐시 surgical insert (`a6df668`, v657)

학원장 보고 — 오후 5시 출제 시험이 10시 넘어도 진도체크 일자별·시험별
진도체크에 안 보임. 진단:
- `_prog.testsByDate[ymd]` / `_tlState.data` 가 **SPA 세션 유지 동안 영구
  캐시** — 시간 만료 로직 없음
- 학원장이 ↻ 새로고침 누르기 전엔 5시간 전 캐시 그대로
- 시험관리 > 최근시험은 매번 fresh fetch — 영향 없음

수정 — `tpPublish` 의 addDoc 후 surgical insert (학원장 reads 0):
- `await addDoc` → `docRef = await addDoc(...)` 로 결과 받기
- 새 시험 row 만들기 (박은 필드 + `_computeTestStats(t, [], allStudents)` 0값)
- `_tlState.data.unshift()` + `_pageMutate('testListBody', ...)` —
  정렬·페이지 상태 유지하며 재렌더 (다른 참조 안전망 동봉)
- `_prog.testsByDate[date]` 가 그 날짜 캐시 있을 때만 push.
  현재 일자별 화면이 그 날짜면 `progRenderByDate()` 자동 재렌더
- 시험관리 (`_renderTestAssignDetail`) 는 기존 fresh fetch 유지

학원장 1인 운영 default 학원에선 새 시험 즉시 반영 + reads 증가 0.
멀티학원장 도입 시 다른 학원장 출제는 여전히 ↻ 필요 (별도 작업).

### 3) 학생앱 시험 목록 SPA 세션 캐시 + 헤드체크 (`cf94640`, v658)

reads 분석 — default 학원 ~656k reads/월 추정. 가장 큰 부담:
- 학생앱 시험 목록 fetch: 학생 94명 × 일 2회 × ~50 reads = ~280k/월
- 같은 세션에서 학생이 시험 페이지 5회 왔다갔다해도 매번 fetch

수정 — SPA 세션 캐시 + 헤드체크 패턴:

- **Firestore Rules**: `academies/{id}` update 허용 키에 `lastTestUpdate` 추가
- **학원장 `tpPublish`**: 출제 시 `academies.lastTestUpdate =
  serverTimestamp()` 박음 (surgical insert 옆에 1줄 추가)
- **학생앱 헤드체크 (`_getAcademyLastTestUpdate`)**: 30초 TTL.
  같은 세션 여러 페이지 이동 시 학원 doc 1 read 만
- **`_loadTestListPage` 캐시 분기**:
  · `state.myTests` + `cacheKey === daysLoaded` + `fetchedAt > lastUpdate`
    → 캐시 사용 (fetch 0, 헤드체크 1 read 만)
  · 그 외 → 기존 3 쿼리 fetch + 캐시 저장
- **`_writeUserCompleted` 후 캐시 동기**: 응시 완료 시점에 모든
  `_testListState.userCompMap` 에 새 결과 박기. `myTests` 캐시 자체는
  유지 (시험 list 변동 X)
- `_invalidateTestListCache(type)` window 노출 — 강제 무효 가능

학원장 출제 → 학생 30초 안 진입 시 헤드체크 차이 감지 → 자동 fresh fetch.
30초 안엔 옛 캐시 가능 (트레이드오프).

### reads 절감 추정

| 시나리오 | 현재 | 적용 후 |
|---------|------|--------|
| 학생 진입 (변경 없음) | ~20 | 1 |
| 학생 진입 (변경 있음) | ~20 | 21 |
| 같은 세션 5회 페이지 이동 | ~100 | 1 |
| **월 추정** | **~780k** | **~225k (~71% ↓)** |

### 작업 규칙 추가 (2026-06-18)

신규:
- **SPA 세션 영구 캐시는 stale 위험** — `_prog.testsByDate[ymd]` /
  `_tlState.data` 처럼 시간 만료 없는 캐시는 학원장이 5시간 자리 비우면
  5시간 전 데이터 그대로. CRUD 후 surgical insert + 외부 변경은 헤드체크
  (`lastUpdate` 1 read) 로 보완. 영구 캐시 도입 시 무효화 경로 동시 설계 필수.
- **헤드체크 패턴 (academies.lastTestUpdate)** — 시험 목록 같은 자주
  read 되는 컬렉션은 학원 doc 의 timestamp 1 read 로 변경 감지. 전체
  fetch 대비 95% reads 절감. 단점: 학원 update 1 write 늘어남 (write 비
  read 보다 비싼 게 아니라 OK). 같은 세션 짧은 시간엔 헤드체크도 TTL (30초)
  로 추가 절감.
- **명시적 픽킹 객체 패턴 신규 필드 carry — 학생앱·학원장앱 양쪽 확인** —
  6/12 `startVocab` 의 `opts.timeLimitSec` 누락 회귀 표본. 신규 필드 추가 시
  Firestore 박은 곳·읽는 곳 모두 grep. 새 필드 carry 위치 누락이 가장 흔한
  회귀 패턴.
- **CRUD 후 캐시 동기 — myTests vs userCompMap 분리 대상 구분** — 시험 응시
  완료 후 `_testListState` 동기 시 `myTests` (시험 list) 자체는 변동 없고
  `userCompMap` 만 그 testId 결과 박으면 충분. 분리 처리가 캐시 효율 ↑.

### 진행률 / 파일 크기 / SW 캐시 (2026-06-13 ~ 18)

- 메시지 다중 첨부: ~100%
- 진도체크 캐시 surgical insert: ~100% (1인 운영 default 충분)
- 학생앱 시험 목록 reads 최적화: ~100% (헤드체크 + SPA 세션 캐시)
- 다른 학원장 변경 감지 / FCM 자동 알림: 미착수 (Phase 5 또는 멀티학원장 도입 시)
- 멀티테넌시·결제·말하기·녹음숙제·AI Generator: 변동 없음
- Phase 5 출시 준비: 0%

파일:
- `firestore.rules`: academies update 허용 키에 `lastTestUpdate` 추가
- `public/admin/js/app.js`: 메시지 다중 첨부 핸들러 + `tpPublish` surgical insert
  + `lastTestUpdate` 박기
- `public/admin/_app.html`: 메시지 입력 `multiple` + 첨부 리스트 영역
- `public/js/app.js`: `_notifAttachments` 헬퍼 + `_getAcademyLastTestUpdate` +
  `_loadTestListPage` 캐시 분기 + `_writeUserCompleted` userCompMap 동기
- `api/sendPush.js`: `attachments[]` 처리 + 옛 단수 호환
- SW 캐시: `kunsori-v655` → `kunsori-v658`

**다음 세션 후보** (변동):
1. 학생앱 reads 절감 효과 실측 (Firebase Console 며칠 추적)
2. 학원장앱 시험관리 (`_renderTestAssignDetail`) 도 같은 패턴 통일 — 2단계
3. 다른 학원장 변경 감지 + FCM 자동 알림 — 멀티학원장 도입 시 묶음
4. minmini 학생 후속 관찰 / 디바이스 정보 진단 / FCM 토큰 만료 (변동 없음)
5. Phase 5 출시 준비 (변동 없음)

---

## 2026-06-18 (오후 추가 작업): 알림 패널·뱃지 최적화 + 인덱스 회귀 교훈

오전 학생앱 시험 목록 캐시·헤드체크 작업 (cf94640) 후, Firebase 쿼리 통계
화면 진단으로 추가 비효율 발견 → 알림 패널·뱃지 reads 절감 작업. 도중
인덱스 누락 회귀 1회 (폴백 안전망 동봉으로 즉시 회복).

### 1) 알림 패널 페이지네이션 (`95c88d9`, v659)

학원장 통찰 — 알림 패널 열면 학생 한 달치 모두 fetch. `limit` 도 없고
`orderBy` 도 server-side 아님. 게다가 `readNotif`/`markAllNotifsRead` 가
끝에 `openNotifPanel()` 재호출 → 알림 5개 읽으면 전체 fetch 5번.

- `_notifPanelState` (lastDoc, exhausted, notifs[])
- `openNotifPanel`: `orderBy desc + limit 10 + cursor`
- `loadMoreNotifs`: `startAfter(lastDoc) + 10개 append`
- `readNotif`: 그 row 만 surgical (read:true 박고 색상·아이콘만 변경, 재fetch 0)
- `markAllNotifsRead`: 현재 노출된 row 모두 surgical, 재fetch 0
- `_renderNotifRow` 헬퍼 분리 (재사용)
- firestore import 에 `startAfter` 추가

**reads 절감 추정**: 학생 1회 사용 (알림 30개, 5개 읽기) 180 → 10 (~94% ↓)

### 2) 인덱스 누락 회귀 + 폴백 안전망 (`3cca68e`, v660)

학원장 보고 "알림함 불러오기 실패" — 신규 쿼리 `where('uid','==') +
orderBy('createdAt','desc')` 가 `userNotifications` 의 composite 인덱스
부재로 거부. [[feedback-index-before-filter]] 위반 (신규 쿼리 추가 시
인덱스 사전 deploy 필수).

긴급 대응:
- `firestore.indexes.json`: `userNotifications (uid, createdAt DESC)` 인덱스 추가
- `firebase deploy --only firestore:indexes` 배포 (빌드 1~5분)
- **코드 폴백**: try/catch 로 인덱스 에러 잡고 옛 방식 (where uid 만 +
  클라 정렬 + slice) 으로 자동 fallback — 빌드 중에도 학생 영향 0
- `_fallbackAll` 캐시로 더보기도 폴백 동작

### 3) 안 읽음 뱃지 aggregate count (`11692c2`, v661)

학원장 정확한 통찰 — "뱃지 카운트도 전체 fetch 후 size 만 쓰는 거 아냐?".
확인 결과 정확. `getCountFromServer` 로 교체:

```js
// before: getDocs(...).size → 10 reads
// after:  getCountFromServer(...).count → 1 read
```

- `updateNotifBadge`: snap.size → c.data().count + 폴백 안전망
- import 에 `getCountFromServer` 추가
- 학원장 admin 앱은 이미 광범위 사용 중 (대시보드·메시지 한도·시험 카운트)

**reads 절감 추정**: 학생당 일 ~10회 갱신 × 10 reads → 1 read = 월 ~282k → ~28k

### 4) 로그인 모달 aggregate count 수평전개 (`d0cb15b`, v662)

`checkUnreadNotifs` (학생 로그인 직후 자동 모달 띄움 함수) 도 동일 패턴
— `snap.size` 만 사용. updateNotifBadge 와 통일.

- 학생앱 `.size` grep 전수 검토 후 1건 발견 → 동일 패턴 적용
- 학원장앱은 이미 모두 적용됨 (수평전개 결과 무변경)

**reads 절감 추정**: 학생 94 × 일 2회 진입 × 10 reads → 1 = 월 ~56k → ~5.6k

### 5) 학원장 결정 — 학생 사용 중 변경 작업 회피

오후 작업 중 인덱스 회귀 1회 발생 → 폴백으로 즉시 복구됐으나 학원장
판단 "학생 많이 쓰는 시간엔 추가 변경 보류". 합리적 운영 결정.
보류된 다음 작업들은 메모리에 등록:
- 홈 뱃지 + 시험 목록 캐시 통합 (옵션 B) — reads ~89% ↓ 추가 가능
- 학원장 진도체크 헤드체크 확장 — [[project-admin-headcheck-extension]]
- 옛 schema 정리 Phase 1~3 — [[project-legacy-schema-cleanup]]

### 작업 규칙 추가 (2026-06-18 오후)

신규:
- **신규 쿼리 추가 시 인덱스 사전 deploy 필수** —
  [[feedback-index-before-filter]] 재강조. composite 인덱스
  (`where(eq) + orderBy(다른필드)`) 누락 시 "The query requires an index"
  로 학생 영향. 폴백 안전망 (try/catch + 옛 방식) 동봉이 회귀 복구
  표준.
- **`getCountFromServer` 는 카운트만 필요한 단일 조건 쿼리에 적용** —
  차집합 계산이나 `snap.docs` 도 같이 쓰는 케이스에는 효과 없음.
  `snap.size` 만 쓰는지 확인 후 교체. 폴백 안전망 동봉.
- **변경 작업은 사용자 사용량 적은 시간대에** — 학원 운영 시간·시험 시간
  회피. 새벽·심야·주말이 안전. 학원장 결정 존중. 회귀 시 영향 최소화.
- **수평전개는 결과 보고 → 사용자 컨펌 → 작업** 단계 분리 필수 —
  "확인해볼까", "스캔해볼까" 단계에서는 결과만 보고. 작업 진행은
  사용자 명시 동의 후. [[feedback-confirm-specs-before-work]] /
  [[feedback-answer-before-work]] 재강조.

### 진행률 / 파일 크기 / SW 캐시 (2026-06-18 오후)

- 알림 패널 페이지네이션 + surgical: ~100%
- 안 읽음 뱃지 aggregate count: ~100%
- 인덱스 회귀 복구 + 폴백 안전망 정착: ~100%
- 학생앱 reads 누적 절감 (오늘): ~1,340k reads/월
- 멀티테넌시·결제·말하기·녹음숙제·AI Generator: 변동 없음

파일:
- `public/js/app.js`: `_notifPanelState` + 페이지네이션 함수들 +
  `_renderNotifRow` 헬퍼 + `readNotif`/`markAllNotifsRead` surgical +
  `updateNotifBadge`/`checkUnreadNotifs` aggregate count + 폴백 안전망
- `firestore.indexes.json`: `userNotifications (uid, createdAt DESC)` 인덱스
- SW 캐시: `kunsori-v658` → `kunsori-v662`

**다음 세션 후보** (학생 사용 적은 시간대 진행):
1. 홈 뱃지 + 시험 목록 캐시 통합 (옵션 B) — 추가 reads ~89% ↓
2. 학원장앱 진도체크 헤드체크 확장 — 다중 사용자 stale 해결
3. 옛 schema 정리 Phase 1 — dead code 3건 (~30분)
4. 학생앱 reads 실측 (다음 7일 통계 비교)
5. Phase 5 출시 준비 (변동 없음)

---

## 2026-06-21 ~ 22: 모달 드래그 + MCQ 시간만료 표시 + 클린업 프롬프트 동기

SW v663 ↑ (~4 commit).

### 1) AI 정리 모달 드래그 이동 (`fdd1e49`, v663)
학원장 요청 — AI OCR 의 AI 정리 모달이 화면 중앙 고정이라 아래 페이지 목록·내용 가림.
- `_enableModalDrag(headerEl)` 헬퍼 — mousedown→mousemove translate, 화면 한계(50px), 헤더 안 button/input/textarea 클릭은 드래그 X
- `showModal(html, {draggable: true})` 옵션, `data-drag-handle` 속성 우선
- `closeModal`: `modalBox.transform = ''` 리셋
- AI 정리 모달 2개 적용 (`_cleanupShowCompareModal` + `_cleanupRenderBatchResult`)
- 직전 commit fdd1e49 에서 SW bump 누락 → `7b45b24` 보완 (v663 표시 정상화)

### 2) 단어시험 MCQ 시간만료 정답·오답 표시 모순 (`ab2ea90`, v664)
학원장 보고 — 김수호 학생 ④ 주정부 (정답) 누른 줄 알았는데 × 오답. 진단:
- 데이터 정상 (charCode 동일). 화면 모순의 원인 — 시간만료 시 `vqNext({allowEmpty:true})` → `ans.input=''` 박힘
- `_vqRenderMcqFeedback` 의 옵션별 `isCorrect`(opt === correctText) 는 ④에 ✓ 표시
- `_vqShowFeedbackBanner('' === correctText)` 는 × 오답 → 두 비교 분리돼 모순 화면
- fix: `userInputEmpty` 분기 추가
  · 정답 옵션: 시간만료 시 호박색 + "(정답)" / 정상 응시 시 초록색 + " ✓"
  · banner: 시간만료 시 "⏰ 시간 만료 — 정답: XXX" 명시

### 3) 클린업 '기본 정리' 프롬프트 코드 default 동기 (`5ae1ac4`, v665)
학원장이 super 앱에서 글로벌 `appConfig/cleanupPresets` 의 '기본 정리' 프롬프트
6번 규칙 추가: "출력은 반드시 영어 원문 그대로 유지하세요. 한국어 번역 절대 금지."
- 배경: AI OCR 정리 시 영어 본문이 한국어로 번역돼 나오는 경우 발생
- `public/admin/js/app.js` `_CLEANUP_DEFAULT_PRESETS` 의 '기본 정리' prompt 동기 (emergency fallback)
- CLAUDE.md 작업 규칙 (양방향 자동 동기 없음 — 사용자 명시 요청 시만)

### 작업 규칙 추가 (2026-06-21~22)
- **화면 id 만으로 화면 상태 구분 불가 시 명시 마커** — innerHTML 교체 패턴
  (`_vqRenderResult` 가 vocabQuiz 화면 안 결과 UI 박음) 에선 curId 가 '시험 중' 과
  '결과 보기 중' 모두 같음. `dataset.stage = 'result'` 같은 명시 마커로 구분.
- **모달 드래그 옵션 plain — false positive 회피** — 모든 showModal 에 자동 활성화는
  호출처가 많아 회귀 검증 비용 큼. `{draggable: true}` 옵션 + `data-drag-handle`
  속성 약속 + 점진 적용이 안전.

### 진행률 / 파일 크기 / SW 캐시 (2026-06-21~22)
- AI 정리 모달 드래그: ~100% (필요 시 다른 모달도 옵션 추가)
- MCQ 시간만료 표시 정합성: ~100%
- 클린업 프롬프트 동기: ~100%
- SW 캐시: `kunsori-v662` → `kunsori-v665`

---

## 2026-06-26: 재원생 목록 결제 컬럼

학생관리 재원생 테이블에서 자동결제 등록 여부 한눈에 확인 (`789dea3`, v666):
- `_app.html` studentTableBody thead: 등록일과 수강료 사이에 '결제' th (sortable + title 툴팁)
- colspan 12 → 13 갱신
- 재원생 row 에 결제 td 추가
  · `tuitionPlan.active === true`: 초록 ✓ (title="자동 청구 ON")
  · 그 외: 빨간 ✗ (title="OFF — 학생 수정에서 [매월 자동 청구서 생성] 체크")
- 휴원/퇴원 테이블은 그대로 (자동 OFF 가 정상)

진단 — default 학원 자동청구 OFF: 김가윤(4시반, 230,000원 — 검토 필요), TEST반 김기헌·문성미 (TEST 계정).

---

## 2026-06-27 ~ 28: 녹음숙제 검증 시스템 광범위 강화 (학원장 보고 연쇄 대응)

SW v666 → v682 (~21 commit). 학원장 보고 → 진단 → 다층 안전망 구축. 사후 진단·우선 확인 도구 + AI 강화 + 학생 안내 + 학원장 액션 일체화.

### 발견된 핵심 문제 — AI 채점 부정확

학원장 보고 케이스들 — 명백한 무음/부분 읽기에 AI 가 정상 점수 부여:
- **서준호 6/24**: voiceActivity 0%, 89초 → **78점** (옛 시스템)
- **조민서 6/23**: voiceActivity 0%, 89초 → **85점**
- **성소율 6/24**: duration 20초/표준 128초 (16%) → **75점**
- **문성미 6/9**: voiceActivity 0%, 31초 → 35점 + categoryScores 70/65/70/30 (모순)

광범위 진단 — 최근 14일 67건 시험에서 100% transcript 빈값 + 일부 음성<10% 인데 75~85점 다수.

### 구축한 다층 안전망

#### 1) 학원장 사후 진단·식별 도구 (1be2396·a2ab2d3·28bc56f·1376a0a·aef1aa3·10eb167·a8b4e3b·59e1b95)

학생 카드 색상 강조 — 우선 확인 대상 즉시 식별:
- 50점 이하 / 측정값 명백 abnormal → 진한 빨강 (#fca5a5)
- 임계: 음성<10% / 명료도<30% / 단조>70% / 짧음(minDur*0.5) / **시간 비율<50%** (본문 단어수 기반)
- 라벨 표시: `⚠ 음성<10%, 시간14%` 같이 사유 명시

학원장 상세 모달 — 6개 지표 색상 분기 + hover 툴팁:
- 시간 비율 / 말소리 / 속도 WPM / 명료도 / 단조로움 / 완독률
- 각 지표 ≥정상 초록 / 주의 호박 / 이상 빨강
- 라벨까지 span 으로 묶어서 hover 시 의미 설명
- **본문 보기** 펼침 (details/summary) — 학원장이 audio 들으면서 본문 비교

학생 카드 강조 6개 임계 통일:
| 지표 | 임계 |
|------|------|
| 점수 | ≤ 50 |
| 음성 활동 | < 10% |
| 명료도 | < 30% |
| 단조로움 | > 70% |
| 짧음 (학원장 minDur) | < minDur*0.5 |
| **시간 비율 (본문 표준)** | < 50% |

#### 2) AI 평가 강화 (7d96ff7·6aad1f8·ec153cb·037657b·b6b67db)

**0-39 점수 가이드 명확화** — 학원장이 Gemini 에 직접 문의해서 받은 개선안:
- "Silent (침묵), noise only (단순 소음), irrelevant babbling (무의미한 웅얼거림),
  or entirely different content (원문과 완전히 다른 내용). 이 경우 즉시 0점 처리"
- "score 0-10 사이 시 categoryScores 4개 모두 0 강제"
- "본문 단어 추측해서 weakPronunciation 생성 금지"

**본문 부분 읽기 차단** — 클라 + AI 동시:
- 클라: 본문 단어수 ≥ 30 시 `expectedDuration = words/150 * 60`. duration < expectedDuration * 0.5 면 모달
- AI: 프롬프트에 본문 단어수·예상 시간·녹음 길이 전달 + "30% 미만 → score 30 이하 강제"

**완독률 측정 (옵션 B)**:
- 프롬프트에 `transcribedWords` 응답 요청 (audio 에서 들린 영단어 배열)
- 서버에서 본문 단어 매칭 → `completionRate = matched/bookUnique * 100`
- responseSchema.required 에 추가 강제
- 학원장 모달: "완독률 25% (80/320 단어)" 표시 + 색상 분기
- 재평가 흐름에도 동일 정보 박힘

**무음 케이스 서버 강제 0점** (b6b67db) — AI 모순 응답 차단:
- transcribedWords.length === 0 OR voiceActivity < 5% OR completionRate < 5%
  → score 0 + categoryScores 4개 모두 0 + feedback 빈 배열 강제
- AI 가 무음에 가상 점수 응답해도 서버가 정직 처리

#### 3) 학생 안내·자가 인지 (000f1fe·3d62ca7·ae00a65)

**저장 전 sanity check 모달** — 임계 해당 시 학생에게 안내:
- 음성<10% / 명료도<30% / 단조>70% / 짧음 / 시간 비율<50%
- "마이크 확인 / 본문 일부만 읽음" 같은 친화 문구
- [확인] 그대로 제출 / [취소] 다시 녹음

**실시간 게인 막대** (3d62ca7) — Web Audio API AnalyserNode:
- 녹음 중 마이크 입력 강도 시각 표시 (가로 막대)
- 색상: <5% 빨강 / <15% 호박 / <40% 초록 / ≥40% 진초록
- 80ms transition (부드러운 갱신)
- 5초 무음 자동 모달 (3d62ca7) → 학원장 피드백 후 폐기 (ae00a65, "잠시 쉴 때도 떠서 거추장")
- 텍스트 고정: "막대가 움직이지 않으면 녹음이 안 되고 있어요"

**디바이스 정보 박음** (3d62ca7) — 학원장 진단용:
- userAgent / platform / os / browser (KAKAOTALK 인앱·삼성 브라우저 식별)
- 학원장 상세 모달 인라인 표시 "📱 Android · Chrome"
- 같은 폰 모델 학생들 패턴 즉시 파악

#### 4) 학원장 액션 (3387a41)

**[🚫 재시험] 버튼** — 학원장 상세 모달 풋터 (재평가 옆):
- 기존 녹음 + AI 평가 + 성적 기록 모두 삭제 → 학생 재응시 가능
- userCompleted/{uid} doc 삭제 / scores doc 삭제 / Storage audio 삭제
- academies.lastTestUpdate 갱신 (학생앱 캐시 30초 안 자동 fresh)
- excludedUids 와 다름 — 응시 가능 상태로 복원

### 작업 규칙 추가 (2026-06-27~28)

신규:
- **AI 모순 응답은 서버 후처리로 정직 정정** — Gemini 가 0-39 가이드 무시하고
  무음에 score 35 + categoryScores 70/65/70/30 응답하는 사례. 프롬프트 강화만으론
  한계. 서버가 transcribedWords/voiceActivity/completionRate 검증 후 강제 0점.
- **다층 안전망 구축 패턴** — 단일 차단보다 4단계 (사후 식별 / AI 강화 / 학생 안내 /
  학원장 액션) 가 효과. 한 단계 실패해도 다음 단계가 잡음.
- **임계 통일 — 학원장 결정 후 3곳 동시 적용** — 시간 비율 50% 결정 시 카드 강조 +
  모달 색상 + 학생 저장 전 모달 모두 동일 임계. 일관성 ↑ + 혼선 ↓.
- **실시간 게인 UX — 자동 알림 모달은 학생 흐름 방해** — 5초 무음 자동 모달은
  학생이 본문 보다가 쉬는 케이스에도 떠서 거추장. 시각 피드백(막대 색)만 +
  고정 안내 문구가 자연스러움.
- **완독률 측정 — transcribedWords required 필수** — responseSchema 의 required
  에 안 박으면 AI 가 생략 가능 필드로 인식 → 빈값 응답. 강제 응답 받으려면
  required 필수.
- **재평가 vs 재시험 구분** — 재평가: 같은 녹음 파일 AI 재호출 (한도 +1).
  재시험: 기존 데이터 삭제 후 학생 재응시 (한도 차감 X). 학원장이 새 시험
  출제하던 흐름 대체.
- **디바이스 정보 박기 — 학원장 진단 필수** — Quantum 4 같은 폰 모델별 마이크
  문제 패턴 즉시 파악. userAgent 박고 학원장 모달에 한 줄 표시 (`📱 Android · Chrome`).

### 진행률 / 파일 크기 / SW 캐시 (2026-06-27~28)

- 녹음숙제 검증 시스템 다층 안전망: ~100% (사후 식별 + AI 강화 + 학생 안내 + 학원장 액션)
- 학원장 진단 도구: ~100% (카드 강조 6임계 + 모달 6지표 색상·툴팁 + 본문 보기 + 디바이스 정보)
- AI 평가 정확도: 크게 향상 (0-39 강화 + transcribedWords required + 무음 서버 강제 0점)
- 학생 사전 차단: ~100% (게인 막대 + 저장 전 sanity check 5임계)
- 학원장 액션: ~100% (재평가 + 재시험 2 옵션)

파일:
- `api/check-recording.js`: 0-39 강화 + transcribedWords required + 본문 정보 +
  무음 케이스 서버 강제 0점
- `api/adminAction.js`: 재평가 흐름에 wordCount/expectedDuration/voiceActivity carry +
  completionRate/bookWordCount/heardWordCount 박음
- `public/admin/js/app.js`: 학생 카드 강조 6임계 + 모달 6지표 색상·툴팁 +
  본문 보기 펼침 + [🚫 재시험] 버튼 + 디바이스 인라인
- `public/js/app.js`: 게인 막대 (Web Audio API) + 저장 전 sanity check 5임계 +
  본문 단어수 자동 임계 + 디바이스 정보 박음 + voiceActivity 서버 전송
- SW 캐시: `kunsori-v666` → `kunsori-v682`

**다음 세션 후보** (학생 사용 적은 시간대 진행):
1. 옛 응시 일괄 재평가 — 학원장 결정 (수십 건, 학원 한도 부담)
2. AI 평가 추세 관찰 — 새 안전망 효과 측정 (며칠 후 통계)
3. 다른 마이크 문제 학생 진단 — 디바이스 정보 박힘 데이터 활용
4. 멀티학원장 / 옛 schema / 시험 목록 캐시 통합 (변동 없음)
5. Phase 5 출시 준비 (변동 없음)

---

## 2026-06-30: 녹음 AI 평가 모델·알고리즘 종합 정비 (3 모델 비교 → 1순위 변경 + 형광펜 DP LCS)

SW v682 → v691 (~13 commit). 학원장 보고 5건 진단 → 3 모델 비교 테스트 → 1순위 모델 변경 +
transcribedWords split + sequence 매칭 + 형광펜 시각화 + 임계 정책 변경 종합.

### 1) 3 모델 비교 진단

scripts/diag/compare-recording-models.js 외 케이스별 비교 스크립트 5종 — 정상응시(권서현 6/25)
/ 무음의심(조민서 6/24) / 부분읽기(성소율 6/24) / 홍지성 6/25·6/29 / 이은섭·성지율·임하리·서준호·
차윤민·권도현·이지호·용주영 등.

핵심 발견 (모델별 패턴 확정):
- **2.5-flash-lite (1순위)** — hallucination 다발:
  · 조민서 무음 audio → score 95, 344 단어 본문 추측 응답
  · 이은섭 부분 읽기 → 162 단어 (본문 전체) 응답, 학생 실제 42초만 발화
  · 성지율 정상 → 227 단어 (본문 전체) 응답, 도달 100% 가짜
- **3.1-flash-lite (2순위)** — 가장 우수: 무음 정직 0점, 정상 응시 정확한 score, 응답 시간 빠름 (3.2s)
- **2.5-flash (3순위)** — thinking model (응답 형식 문제):
  · responseMimeType:application/json + thinking tokens 가 maxOutputTokens 96% 소비 → 응답 잘림
  · maxOutputTokens 늘려도 thinking 비례 증가 (3000→16384 도 잘림)
  · 운영 (3000 limit) 에선 사실상 dead code — 폴백 3순위 도달 0건

### 2) 1순위 모델 변경 (commit 644d17f)

api/check-recording.js MODELS swap — 2.5-flash-lite → 3.1-flash-lite → 2.5-flash
**→ 3.1-flash-lite → 2.5-flash-lite → 2.5-flash**.

근거: 3.1-lite 무음 정직 응답 (가짜 점수 차단) / 3.1-lite 정상 응시 정확도 ↑ (홍지성 81% →
89%) / 3.1-lite 응답 시간 빠름 (7s → 3.2s) / 3.1-lite 503 영향 ↓ (사용량 적음).

비용: 학원당 월 +$3 추정 (default 학원 ~1500 응시). 503 시 fallback = 2.5-lite 호출 = **상호
보완 폴백 체인**.

### 3) 503 비용 처리 정리 (사용자 통찰)

작업규칙 — Gemini 503/429 응답은 **요금 0** (구글 표준): 1순위 503 받은 호출만 무료, 폴백된
모델은 그 모델 가격으로 청구. 폴백 정책 ≠ 가격 할인.

### 4) transcribedWords split + 프롬프트 강화 (commit 644d17f)

서준호 6/29 응시 진단 — Vercel 로그에서 transcribedWords 가 **문장 element** 로 응답:
["you've been looking at puppies for months", "said d w", ...]. 운영 코드가 문장 element
그대로 박음 → bookUnique.has 매칭 안 됨 → 완독률 0%.

수정: flatMap(s => match(/[a-z']+/g) || []) — 문장도 단어 단위 자동 추출. slice 200 → 500.
프롬프트 강화 — "MUST be flat array of INDIVIDUAL words. NEVER bundle into sentences." 예시 포함.

영향: 옛 응시 5건 중 3건 (서준호 본문1·2, 임하리) 완독률 부정확 → [🔁 재평가] 시 정상화.

### 5) sequence 매칭 도입 — lastReadPosition + avoidanceJumps (commit 644d17f)

서버 _calcLastReadPosition 신설: 학생 단어 마다 본문 위치 +25 윈도우 안에서 매칭 → 매칭된
마지막 위치 = lastReadPosition, 큰 점프 (>15) = avoidanceJumps.

학원장 모달 인라인: "시간 비율 105% · 완독률 85% · 도달 100% (끝까지)"

검증 (학원장 진단과 일치): 이은섭 반만 → 41% / 성지율 14/17줄 → 82% / 임하리 정상 → 100% +
6 점프 (자연 D.W. 건너뜀) / 권도현 본문 끝까지 + 발음 부정확 → 100% + 1 점프.

### 6) 느린 속도 페널티 제거 (commit 5b6efb0)

서지율 6/29 CH5 — 본문 87% 완독 + 다 읽었는데 score 25 (매우 낮음). 원인: AI categoryScores
pace 30 으로 종합 score 강한 페널티. 학생 duration 227s / 표준 95s = **시간 비율 239%**
(천천히 또박또박 읽음, 학습 단계 정상).

수정 — buildEvalPrompt 3곳: pace 카테고리 "느린 속도는 페널티 X" 명시 / 코멘트 예시 "천천히
또박또박 잘 읽었어요" 추가 / Overall Score 가이드 신규.

### 7) 학원장 카드 색상 정책 (commit cbabb20)

학원장 정책 — **AI 점수보다 객관 측정(완독·도달) 우선**.

3단계 색상:
- 도달 100% OR 완독률 90%+ → 🔵 정상 (시간·점수 무관, hasNormalIndicator)
- 시간 < 90% OR 점수 ≤50 OR 측정값 이상 → 🔴 강한 빨강
- 시간 90~110% → 🟠 약한 빨강 (도달·완독 정상이면 자동 해제)
- 시간 > 110% → 🔵 정상 (느린 학생 정상)

⚠ hallucination 위험 인지: 무음 + AI 본문 추측 → 완독률 100% 가짜 = 정상 표시 가능. 1순위
3.1-lite + split 으로 크게 감소했으나 0 아님.

### 8) 형광펜 시각화 5단계 진화

학원장 통찰 — 본문에 학생 읽은 단어 형광펜으로 시각화. 알고리즘 5번 진화:

#### 8a) 위치 기반 sequence 매칭 (Greedy + cutoff) — 5b6efb0
LCS-like sequence 매칭으로 학생 도달 위치 추적. 매칭된 위치 노란색 / 도달 너머 옅은 회색 이탤릭.

#### 8b) set intersection (완독률 일관) — 76b21be
용주영 케이스: 완독률 86% 인데 LCS 11% 만 매칭. 원인 LCS sequence 한 방향 진행 → 학생 누락
시 다발 미스. set intersection 으로 변경. 단점: 흔한 단어 (the) 모든 위치 false positive.

#### 8c) 위치 기반 매핑 (LCS 매칭 위치 Set) — 027e161
학원장 통찰 — set intersection false positive. 가운데 3줄 건너뜀: 도달 100% + 완독 89% +
가운데 the 노란색 (가짜). LCS 결과의 매칭된 토큰 인덱스만 Set 에 기록 → 그 위치만 노란색.
도달 너머 (maxMatched 초과) 옅은 회색 이탤릭.

#### 8d) 구두점·apostrophe 정규화 — 92d852a·09ca87b
도달 못한 부분 구두점 (. , " ! ?) 도 옅은 회색 이탤릭 / apostrophe 정규화 norm = s =>
s.replace(/'/g, '') — it's ↔ its 매칭.

#### 8e) DP LCS 정식 알고리즘 (commit 118f9ee) ★ 최종
권도현 케이스 — 완독률 75% 인데 Greedy LCS 42%, 사용자 "엇비슷하게 칠해져야지" 요구. 원인:
Greedy LCS 한 방향 진행 → 학생 발음 부정확 시 bookPos 못 따라가 다발 미스. 윈도우 확대
(25→40) 도 흔한 단어 false jump 로 악화 (권도현 42% → 27%).

해결 — 정식 DP LCS (동적계획법): 학생 sequence × 본문 sequence DP 테이블 (Int16Array 메모리
최적) / 윈도우 제약 없이 최장 공통 부분 수열 정확 계산 / 흔한 단어도 최적 위치 자동 매칭 (false
jump 없음) / 시간 복잡도 O(N×M) 브라우저 즉시.

시뮬레이션 (CH5 응시 6명) — 사용자 의도 완벽 만족:
- 이지호: 완독률 93% / Greedy 46% → DP **97%** (+117)
- 서지율: 완독률 82% / Greedy 47% → DP **84%** (+84)
- 권도현: 완독률 75% / Greedy 42% → DP **74%** (+73)
- 성지율: 완독률 89% / Greedy 73% → DP 78% (+10)
- 홍주영: 완독률 92% / Greedy 99% → DP 99% (변동 없음)
- 고다율: 완독률 99% / Greedy 100% → DP 100% (변동 없음)

→ **DP LCS = 완독률과 거의 일치**, 정상 응시 영향 없음.

### 9) 시간 비율 임계 정책 변경

학원장 카드 강조 (수차례 조정): 50% → 80% → 90% (단계 색상 도입). 90~110% 약한 빨강 +
도달 100%·완독 90%+ 시 자동 해제.

### 10) thinkingBudget: 0 + adminAction carry

api/check-recording.js generationConfig 에 thinkingConfig: { thinkingBudget: 0 } 추가 — 폴백
3순위 2.5-flash 가 thinking 으로 잘리지 않도록 보험. 1·2순위 lite 모델은 영향 0.

api/adminAction.js 재평가 흐름에 lastReadPosition·avoidanceJumps·transcribedWords carry — 운영
응답과 재평가 응답 동일 데이터 박힘.

### 작업 규칙 추가 (2026-06-30)

신규:
- **AI 응답 변동성 = 운영상 문제** — 같은 audio + 같은 모델 라도 sampling 으로 다른 응답
  (홍지성 응시 시점 18% / 재테스트 72%). 학원장이 보는 점수가 응시 시점 운에 따라 변동. 근본
  해결책 = 1순위 모델 변경 + 응답 검증 + 폴백 보강.
- **Greedy LCS sequence 매칭의 한 방향 진행 한계** — 학생 단어 누락 시 bookPos 못 따라가 그
  뒤 매칭 다발 미스. 윈도우 확대 (25→40) 도 흔한 단어 false jump 로 악화 (권도현 42%→27%).
  → 정식 DP LCS (동적계획법) 필수. 시간 복잡도 O(N×M) 작아 브라우저 즉시.
- **AI transcribedWords 응답 형식 변동** — 단어 단위 vs 문장 element 응답 변동성 (temperature
  0.9 영향). flatMap split 으로 어떤 응답 형식이든 단어 단위 자동 추출. 운영 코드 방어 필수.
- **흔한 단어 (the, is) 매칭의 위치 정보 손실** — set intersection 형광펜은 본문 모든 위치 매칭
  (false positive). 가운데 건너뜀 케이스 식별 불가. 위치 기반 매핑 (LCS 매칭 위치 Set) 으로
  해결.
- **AI 점수 vs 객관 측정 우선순위** — 학원장 카드 정책: 도달 100% 또는 완독률 90%+ → 정상. AI
  score 극단이어도 객관 측정 우선. AI 응답 변동성 영향 최소화.
- **느린 속도 페널티 = AI 평가 흔한 부작용** — 학생이 천천히 또박또박 읽는 게 학습 단계 정상인데
  AI categoryScores pace 가 깎임 + 종합 score 추가 페널티. 프롬프트에 "느린 속도는 페널티 X"
  명시 필수.
- **2.5-flash thinking model 운영 한계** — responseMimeType:application/json + thinking
  tokens 가 maxOutputTokens 96% 소비 → 응답 잘림. thinkingBudget:0 으로 thinking 끄거나
  responseMimeType 제거 (plain text 파싱) 필요. 다만 폴백 3순위 도달 거의 0 라 운영 영향 작음.
- **DP LCS 시뮬레이션 우선** — Greedy → DP 같은 알고리즘 변경 시 운영 학생 데이터로 시뮬레이션
  먼저. 권도현·이지호 등 케이스 검증 후 적용. 시뮬레이션 결과 보고 사용자 결정 받음.

### 파일 크기 / SW 캐시 (2026-06-30)

- api/check-recording.js: 1순위 모델 swap + transcribedWords flatMap split + 프롬프트 단어 단위
  강제 + sequence 매칭 + 느린 속도 페널티 제거 + thinkingBudget 0 + 응답에 transcribedWords 추가
- api/adminAction.js: 재평가 시 lastReadPosition·avoidanceJumps·transcribedWords carry
- public/admin/js/app.js: 학원장 모달 도달 N% 인라인 + 형광펜 DP LCS + apostrophe 정규화 + 3색
  시각화 + 시간 비율 3단계 색상 + hasNormalIndicator
- public/js/app.js: recordings 에 transcribedWords + sequence 결과 박음
- SW 캐시: kunsori-v682 → kunsori-v691

### 진행률 (2026-06-30)

- 녹음 AI 평가 신뢰도: **~100%** (모델 변경 + split + sequence + 형광펜 DP LCS + 카드 정책)
- 학원장 진단 도구: **~100%** (도달·점프·완독률·형광·카드 색상 3단계)
- 느린 속도 페널티 제거: ~100% (서지율 같은 케이스 정상화)
- 응답 변동성 보완: ~80% (modelUsed·elapsedMs 박힘 미완)

**완료 (이 세션, 2026-06-30)**:
- ✅ 3 모델 비교 진단 (정상·무음·부분읽기·hallucination 패턴 확정)
- ✅ 1순위 모델 변경 (2.5-flash-lite → 3.1-flash-lite, hallucination 차단)
- ✅ transcribedWords flatMap split (문장 응답 대응)
- ✅ 프롬프트 강화 (단어 단위 응답 강제 + 예시)
- ✅ sequence 매칭 도입 (lastReadPosition·avoidanceJumps)
- ✅ 학원장 모달 도달 위치 인라인 표시
- ✅ 느린 속도 페널티 제거 (서지율 케이스)
- ✅ 학원장 카드 색상 3단계 정책 (도달·완독 우선)
- ✅ 시간 비율 임계 단계 변경 (50→90, 90~110 약한 빨강)
- ✅ 형광펜 5단계 진화 — Greedy → set → 위치 기반 → DP LCS
- ✅ 도달 못한 부분 구두점도 옅은 회색 이탤릭
- ✅ apostrophe 정규화 (it's ↔ its)
- ✅ DP LCS 정식 알고리즘 (권도현 42% → 74%, 이지호 46% → 97%)
- ✅ thinkingBudget 0 (2.5-flash 폴백 보험)
- ✅ adminAction.js 재평가 carry

**다음 세션 후보**:
1. modelUsed·elapsedMs 응답 박힘 (응답 변동성 추적, 향후 신규 응시)
2. 옛 응시 [🔁 재평가] 권장 학생 명단 (서준호·서지율·이지호·임하리 등 split·DP LCS 적용)
3. AI 평가 추세 관찰 (새 안전망 며칠 후 통계)
4. Phase 5 출시 준비 (변동 없음)

---

## 녹음숙제 평가 — 용어 정리 (2026-06-30 확정)

| 라벨 (학원장 화면) | 내부 필드 | 의미 | 알고리즘 |
|----------|--------|------|--------|
| **매칭** | `completionRate` | AI 가 들은 단어 중 본문 단어와 **일치 비율** (단어 인식 정확도) | unique set intersection |
| **완독** | `lastReadPosition` | 학생이 본문 어디까지 **도달**해서 읽었는지 (%) | sequence LCS-like (윈도우 25) |
| 회피 점프 | `avoidanceJumps` | 본문에서 큰 점프 (>15 단어) 횟수 — 가운데 건너뜀 회피 식별 | 위 sequence 매칭 부산물 |
| 시간 비율 | `duration / expectedSec` | 학생 녹음 시간 / 본문 표준 시간 (150 WPM) | 클라 측정 |
| 형광펜 (모달 본문 보기) | DP LCS 매칭 위치 Set | 학생이 정확히 발음한 본문 위치 시각화 | DP LCS (Greedy 아님) |
| 강제 0점 | `forcedZero` | 무음 또는 transcribedWords 빈값 시 score 0 강제 | 서버 검증 |

### 용어 사용 규칙 (혼동 방지)
- **"완독률"** 단어는 **사용 안 함** (옛 라벨, 사용자 혼동) — `completionRate` 는 **매칭** 으로 표시
- **"도달"** 단어는 **사용 안 함** (옛 라벨) — `lastReadPosition` 은 **완독** 으로 표시
- 사용자 지시·답변 시 **매칭 / 완독** 사용 (내부 필드명 `completionRate`/`lastReadPosition` 는 코드 호환성 위해 유지)
- 학원장 카드 cautionReasons 라벨: `'완독'` (도달 < 90 시)

### 학원장 카드 색상 정책 (2026-06-30 최종)

| 완독 (lastReadPosition) | 시간 비율 | 색상 |
|------|------|------|
| ≥ 90% | - | 🔵 정상 |
| < 90% | < 90% | 🔴 강한 빨강 (시간 라벨) |
| < 90% | 90%+ | 🟠 약한 빨강 (완독 라벨) |
| - | - | 🔴 강한 빨강 (음성/명료도/단조/짧음/점수≤50 임계 시) |

### hasNormalIndicator (강한 빨강·약한 빨강 모두 해제)
- 완독 ≥ 90% OR 매칭 ≥ 90% → 정상 표시

### 모달 헤더 인라인 표시 예시
```
130초 · 말소리 71% · 속도 148 WPM · 명료도 70% · 단조 30% · 매칭 87% · 완독 100% (끝까지) · 📱 Android Chrome
```

---
> Source: [harry3085/ReadAloud](https://github.com/harry3085/ReadAloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
