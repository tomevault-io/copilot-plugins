## campusfit

> 흩어진 장학금·공모전·지자체혜택·대외활동·인턴십 정보 중, 온보딩에서 받은 조건(국립/사립, 지역)에

# 캠퍼스핏 (CampusFit)

흩어진 장학금·공모전·지자체혜택·대외활동·인턴십 정보 중, 온보딩에서 받은 조건(국립/사립, 지역)에
맞는 것만 걸러 보여주는 서비스. 팀 단위 공고는 인앱 팀원모집 게시판(댓글 지원)으로 실제 지원까지 이어지게 한다.
자세한 배경은 [캠퍼스핏-기획서.pptx](../../캠퍼스핏-기획서.pptx) 참고.

핵심 기능 3가지 (기획서 우선순위 그대로):
1. 조건 기반 정보 필터링 — 서비스의 심장
2. 카테고리별 목록 + 마감 임박
3. 팀원모집 게시판 (댓글로 지원)

## 절대 규칙

- `design/design-tokens.css`에 없는 색상·radius 값을 임의로 추가하지 않는다.
- 기획서 MVP 범위 밖 기능(카테고리 내 검색, 스크랩, 커뮤니티, 개인 알림 등)을 먼저 만들지 않는다.
- 디렉토리 구조·기술 스택처럼 큰 구조를 바꾸기 전엔 먼저 제안하고 사용자 승인을 받는다.
- git push·PR 생성은 사용자가 명시적으로 요청할 때만 한다.

## 금지 사항

- 요청하지 않은 기능을 먼저 추가하지 않는다.
- 팀원모집이 아닌 일반 공고에 인앱 상세 화면을 새로 만들지 않는다 — 원래 설계는 공식 페이지로
  나가는 것이다 (design-system.md 원칙 7).
- PR의 "내가 설명할 수 있는 부분 / 아직 이해 못 한 부분 / 새로 알게 된 것" 회고 3섹션을 AI가
  대신 써주지 않는다 — 질문만 하고, 사용자 본인의 말로 채운다.
- 모르는 내용을 아는 척하지 않는다.

## 디렉토리 구조

```
campusfit/
  design/                 디자인 시스템 (소스 오브 트루스)
    prototype.html         클릭형 HTML/CSS 프로토타입
    design-tokens.css      색상·타이포 토큰
    design-system.md       컴포넌트 패턴 · 톤 원칙
  client/                 React + Vite 프론트엔드
    src/
      pages/                화면 단위 컴포넌트
      components/           재사용 UI 조각 (필요해지면 생성)
      styles/
        tokens.css            design/design-tokens.css를 복사한 것 — 원본 바뀌면 여기도 동기화
        components.css        design-system.md의 컴포넌트 패턴 (.frame, .cat-card, .pill, .row, .post-card 등)
      api/
        client.js             서버 API 호출 wrapper
  server/                 Express 백엔드
    src/
      index.js              앱 진입점
      routes/                /api/listings, /api/board
      data/seed.js           더미 시드 데이터 (DB 붙기 전까지)
  .claude/skills/
    campusfit-design/       디자인 검수용 개인 skill
    pr/                     과제 PR 생성 절차 skill
```

## 왜 이 구성인가

- **React + Vite**: 미션 기본 스택. 화면 전환이 많은 SPA라 라우팅(react-router-dom)을 같이 둠.
- **Express**: 별도 백엔드 프레임워크 학습 곡선 없이 REST API만 빠르게 얹기 위함. RAG/LLM 계층이 없는
  서비스라 Express만으로 충분 — 별도 LLM 계층은 필요 없다 (온보딩 조건이 알바 매니저 프로젝트처럼
  대화형 인수인계가 아니라 단순 선택형이기 때문).
- **디자인 토큰을 CSS 변수로**: Tailwind나 CSS-in-JS 대신 순수 CSS 변수를 쓰는 이유는, 프로토타입을
  그대로 만든 방식(HTML/CSS)과 결과물 사이에 변환 손실이 없게 하기 위함. `design/design-tokens.css`가
  원본이고 `client/src/styles/tokens.css`는 그걸 복사한 사본이다.
- **더미 시드 데이터**: DB를 아직 정하지 않았기 때문에 `server/src/data/seed.js`의 배열로 임시 대체.
  라우트(`routes/listings.js`, `routes/board.js`)는 이미 실제 API 모양으로 짜여 있어서, DB가 정해지면
  데이터 소스만 바꾸면 된다.

## 작업 절차 스킬

- 화면 디자인·검수: `campusfit-design` — `.claude/skills/campusfit-design/SKILL.md`
- 과제 PR 생성: `pr` — `.claude/skills/pr/SKILL.md`

## 코딩 컨벤션

- 컴포넌트 파일명: PascalCase (`Main.jsx`), 그 외 파일: camelCase
- 새 화면을 만들기 전에 `.claude/skills/campusfit-design`의 체크리스트로 디자인 의도와 맞는지 확인
- 새 CSS 클래스를 추가하기 전에 `client/src/styles/components.css`에 재사용 가능한 게 있는지 먼저 확인
- API 응답 형식: 목록은 배열을 바로 반환 (래핑 객체 없음), 에러는 `{ error: string }`

## 커밋 메시지 규칙

`<type>: <설명>` 형식, 설명은 한국어.

| type | 용도 |
|---|---|
| `feat` | 새 기능 |
| `fix` | 버그 수정 |
| `design` | 화면/디자인 시스템 변경 (기능 변경 없음) |
| `refactor` | 동작은 그대로, 구조만 변경 |
| `chore` | 설정, 의존성, 잡일 |
| `docs` | 문서만 변경 |

예: `feat: 팀원모집 게시판 댓글 API 추가`, `design: 마감임박 pill 색상 조정`

## 아직 결정 안 된 것 (2주차 전에 정할 것)

- **DB**: 지금은 `seed.js` 더미 배열. SQLite로 갈지 다른 걸 쓸지 미정.
- **로그인/계정**: 기획서엔 온보딩(조건 선택)만 있고 회원가입 언급이 없다. 댓글 지원에 "누가" 썼는지
  구분하려면 최소한의 세션/닉네임 정도는 필요한지 정해야 한다.
- **배포**: 로컬 개발만 세팅된 상태, 배포 플랫폼 미정.
- **팀원모집 상태 변화**: 모집 완료된 글을 어떻게 표시할지 (design-system.md에도 동일하게 남겨둠).
- **공고 자동 수집**: 지금은 사람이 안 올리고, 한국장학재단·청년정책(공공데이터포털 API 있는 곳)부터
  자동 연동하고 공모전처럼 API 없는 곳은 크롤링/수동으로 시작하는 걸 검토 중 (2026-07-09). 원문
  상세페이지로 바로 연결하려면 소스 데이터에 링크 필드가 있어야 하고, DB 스키마(출처/원문링크/
  최종수집일시)도 이 결정에 달려있음.

---
> Source: [sohyunll/campusfit](https://github.com/sohyunll/campusfit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
