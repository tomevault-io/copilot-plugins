## cshj-management-app

> 모임 멤버들의 출석·미션 실적을 관리하고 랭킹을 시각화하는 웹앱.

# 실행자들 실행실적 관리 앱

## 프로젝트 개요

모임 멤버들의 출석·미션 실적을 관리하고 랭킹을 시각화하는 웹앱.

## 기술 스택

- **순수 HTML + CSS + Vanilla JS (ES6+)** — 프레임워크 없음
- **단일 파일**: `index.html` 하나로 완결
- **외부 라이브러리**: Chart.js CDN 1개만 허용
- **데이터 저장**: `localStorage` 전용 (서버·DB 없음)
- **빌드 없음**: 브라우저에서 직접 열면 즉시 동작

## 파일 구조

```
index.html        # 앱 전체 (HTML + CSS + JS 포함)
CLAUDE.md         # 이 파일
.claude/
  launch.json     # 개발 서버 설정
  settings.local.json
```

## localStorage 키 구조

| 키 | 타입 | 설명 |
|----|------|------|
| `members` | Array | 회원 목록 |
| `sessions` | Array | 회차(모임) 목록 |
| `attendance` | Object | 출석 기록 (`memberId__sessionId` → `"O"/"X"`) |
| `missions` | Array | 미션 목록 |
| `missionScores` | Object | 미션 점수 (`memberId__missionId` → 숫자) |
| `settings` | Object | 레벨 기준, 출석 점수, 편집 잠금 비밀번호 |

## settings 구조

```js
{
  levels: [
    { name: '레전드', minScore: 500 },
    { name: '다이아',  minScore: 350 },
    { name: '골드',    minScore: 200 },
    { name: '실버',    minScore: 100 },
    { name: '브론즈',  minScore: 0   }
  ],
  attendScore: 10,
  lockPw: null   // 4자리 숫자 문자열 또는 null (잠금 미설정)
}
```

## 점수 계산 공식

```
총점 = 출석 점수 + 미션 점수
출석 점수 = O 횟수 × settings.attendScore (기본 10점)
미션 점수 = 해당 회원의 missionScores 합산
레벨 = settings.levels 를 minScore 내림차순 순회, 총점 >= minScore 첫 번째 적용
```

## 화면 구성 (5개 탭)

- **랭킹**: 총점 내림차순, 레벨 배지, 순위 변동 표시
- **회원**: 회원 CRUD, 상세 페이지(출석 이력·미션 점수 막대)
- **출석**: 회차 CRUD, O/X 토글 편집, 참석자/불참자 분리, 회차명·날짜 편집 가능
- **미션**: 카테고리 필터(AI·투자·마인드셋·기타), 미션×회원 교차표, 점수 입력
- **설정**: 레벨 기준 편집, 출석 점수 설정, 편집 잠금(4자리 PIN), JSON 내보내기/가져오기, 전체 초기화

## 미션 교차표 레이아웃

- **행 = 미션**, **열 = 회원** (모바일 세로 스크롤 최적화)
- 회원 이름은 열 헤더에 세로 글씨(`writing-mode: vertical-rl`)로 표시
- 개인별 합계는 이름 바로 아래 헤더 두 번째 행에 표시
- 점수 입력: 5점 단위 증감, 최대점수 초과 불가
- 셀 색상: 만점=초록, 부분점수=주황, 미입력=회색

## 편집 잠금 (PIN Lock)

- `settings.lockPw`에 4자리 숫자 저장
- 잠금 활성화 시 헤더 우측 🔒 아이콘 표시
- 잠금 대상: 회원 추가/편집/삭제, 출석 편집, 회차 추가/편집, 미션 점수 입력, 미션 추가
- `sessionUnlocked` 변수로 세션 내 잠금 해제 상태 관리
- 설정 탭에서 비밀번호 설정·변경·삭제 가능

## 개발 규칙

- 프레임워크·빌드 도구 추가 금지
- `crypto.randomUUID()`로 ID 생성
- 미션 점수 입력 시 최대점수 초과 불가 검증 필수
- 전체 초기화는 2단계 확인 다이얼로그 필수
- 최대 너비 480px, 모바일 우선 레이아웃

## 디자인 테마 (다크 모드)

```css
--bg-page:   #0E1018
--bg-card:   #181C28
--bg-muted:  #1F2438
--border:    rgba(255,255,255,0.08)
--text-main: #EAEAF6
--text-sub:  #8A8AAE
--text-hint: #505070
--accent:    #5B8DF0
--success:   #3EC87A
--danger:    #F05A6A
```

- 헤더: 보라-네이비 그라디언트 + 로고 이미지 내장(base64)
- 탭바: `#12162A` 딥 다크, 글자 12px semi-bold, 활성=흰색
- 카드·버튼: 드롭 섀도우 + 그라디언트
- 모달: backdrop-filter blur 오버레이

## 아바타 색상 (index % 5, 다크 테마)

| index | bg | text |
|-------|----|------|
| 0 | rgba(120,100,240,0.2) | #A090FF |
| 1 | rgba(60,180,130,0.2) | #60D0A0 |
| 2 | rgba(220,100,80,0.2) | #F08070 |
| 3 | rgba(80,160,240,0.2) | #70B0F0 |
| 4 | rgba(220,160,60,0.2) | #E0A050 |

## 레벨 배지 색상 (다크 테마)

| 레벨 | bg | text |
|------|----|------|
| 레전드 | rgba(255,200,80,0.15) | #FFD060 |
| 다이아 | rgba(100,180,255,0.15) | #70C0FF |
| 골드 | rgba(255,165,60,0.15) | #FFA050 |
| 실버 | rgba(180,180,200,0.12) | #B0B0CC |
| 브론즈 | rgba(200,120,80,0.15) | #D08060 |

## 미션 카테고리

`AI` · `투자` · `마인드셋` · `기타` (기본값: AI, 최대점수 기본값: 20점)

## MCP 설정

- **Playwright MCP**: 자동화 테스트용 (`npx @playwright/mcp@latest`)
- **GitHub MCP**: 저장소 관리용 (`npx @modelcontextprotocol/server-github`)

---
> Source: [kjj0708-ai/cshj-management-app](https://github.com/kjj0708-ai/cshj-management-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
