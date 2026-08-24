## dashboard-urban

> Codex가 이 저장소에서 항상 먼저 참고하는 작업 지시다. 이 파일에는 작업 판단에 필요한 핵심 규칙만 둔다. 상세 Apps Script API 계약, 시트 구조, 요청/응답 예시는 `docs/apps-script-api.md`에서 확인한다.

# Codex Project Guide

Codex가 이 저장소에서 항상 먼저 참고하는 작업 지시다. 이 파일에는 작업 판단에 필요한 핵심 규칙만 둔다. 상세 Apps Script API 계약, 시트 구조, 요청/응답 예시는 `docs/apps-script-api.md`에서 확인한다.

## Project Map

- `index.html`: 메인 정적 대시보드. 로컬 폴더에서 직접 열어 쓰는 1차 운영 파일이며, GitHub Pages도 같은 파일을 루트에서 서빙한다.
- `btskin.html`, `belrmon.html`, `gyunghee.html`, `eyecare.html`, `igochi.html`, `echi.html`, `seoulup.html`, `jejuexpress.html`: 거래처별 월말보고서 HTML.
- `scripts/naver-review-monitor.mjs`: 네이버플레이스 방문자 리뷰 모니터링 자동화.
- `scripts/fetch-report-context.mjs`: 월말보고서 작성 전 Apps Script 데이터를 조회해 `.report-context/YYYY-MM/*.json` 스냅샷을 만드는 조회 전용 스크립트.
- `scripts/ops-check.mjs`: Codex 운영 점검 스크립트. 이전 도구명, 오래된 파일명, JSON 오류, 보고서 링크 파일 존재 여부, UTC 날짜 슬라이싱, 리뷰 모니터 문법 오류를 확인한다.
- `.github/workflows/naver-review-monitor.yml`: 리뷰 모니터 정기 실행 워크플로.
- `apps-script/*.gs`: Google Apps Script에 수동 반영할 패치/헬퍼 코드. 배포된 라이브 코드 자체가 아니라 적용용 스니펫이다.
- `docs/apps-script-api.md`: Apps Script API, Google Sheets 구조, 대시보드 운영 규칙의 상세 기준 문서.
- `docs/briefs/`: 거래처별 상세 브리프 위키. 대시보드 `ClientBriefs`보다 긴 맥락, 원본 자료 링크, 작성 기준을 보관한다.
- `docs/compliance/`: 의료광고 검수 원본 PDF, 운영 가이드, JSON 체크리스트, 서브에이전트 검수 프롬프트.
- `review-monitor.config.example.json`: 로컬 리뷰 모니터 설정 예시. 실제 설정 파일 `review-monitor.config.json`은 커밋하지 않는다.

## Operating Principles

- 이 프로젝트는 로컬 폴더의 `index.html` 직접 사용과 GitHub Pages 배포를 동시에 지원하는 정적 HTML 대시보드다. 사용자가 명시하지 않으면 번들러, 로컬 서버, 프레임워크 마이그레이션, 빌드 산출물 구조를 추가하지 않는다.
- 로컬과 GitHub Pages는 같은 루트 파일을 사용해야 한다. 로컬 전용 경로, PC 절대경로, GitHub 전용 절대 URL을 대시보드 내부 링크로 넣지 않는다.
- 기존 단일 HTML 구조를 존중한다. 대규모 재작성보다 필요한 영역만 좁게 수정한다.
- 한국어 문구, 거래처명, 의료/마케팅 리포트 문맥은 의미가 바뀌지 않도록 보존한다.
- 파일은 UTF-8 기준으로 다룬다. 한글이 깨져 보이면 먼저 출력 인코딩 문제인지 확인한다.
- API URL, Google Sheets URL, clientId처럼 문서화된 운영 식별자는 임의로 교체하지 않는다.
- Apps Script API, 시트 컬럼, action 이름, 응답 구조를 바꾸면 같은 작업에서 `docs/apps-script-api.md`도 갱신한다.
- 로컬 전용 설정, 토큰, `.env`, `review-monitor.config.json`, `credentials.local.js`, 로그 파일, `node_modules/`는 저장소에 추가하지 않는다.

## Data And API Rules

- Apps Script POST 요청은 반드시 `redirect: 'follow'`를 포함한다.
- Apps Script POST 요청에는 `Content-Type` 헤더를 추가하지 않는다. CORS preflight 문제를 피하기 위한 운영 규칙이다.
- 글 보관함 저장은 기존 `addDraft`, `updateDraft`, `deleteDraft` POST 구조를 유지한다.
- 글 보관함 조회는 가능하면 `GET ?action=draftsList`로 본문 없는 목록을 받고, 사용자가 펼치거나 복사할 때만 `GET ?action=draftDetail&draftId=...`로 본문을 조회한다. 대시보드 전체 동기화는 `GET ?action=summary&draftMode=light`를 사용할 수 있다.
- 날짜와 마감일 계산은 KST 로컬 날짜 기준으로 처리한다. `new Date().toISOString().slice(0, 10)` 방식으로 오늘 날짜를 만들지 않는다.
- 공식 clientId는 `btskin`, `belrmon`, `eyecare`, `seoulup`, `echi`, `igochi`, `jejuexpress`, `kyunghee`, `hwabuk`, `jocheon`만 사용한다. `kyunghee`는 기존 Drafts 호환을 위해 365경희부부한의원 피부클리닉에 유지하고, `hwabuk`과 `jocheon`은 지점별 글 보관함/Naver 라우팅에만 사용한다.
- 잘못된 별칭 `bellemont`, `eyeclinic`, `igo`는 사용하지 않는다. 각각 `belrmon`, `eyecare`, `igochi`로 정정한다.
- 발행 기록은 `postLogs`를 기준으로 하며, `publishedAt`은 `YYYY-MM-DD`, `month`는 `YYYY-MM`, `channel`은 문서의 정규화 규칙에 맞춰 처리한다.
- API/Sheets에서 확인 가능한 정량 수치와 사용자가 제공한 외부 콘솔 수치를 구분한다. 확인되지 않은 수치는 만들지 않고 “자료 확인 필요”로 남긴다.
- 의료/시술 콘텐츠 초안 작성 전에는 `clientBrief&clientId=...`, `docs/briefs/clients/{clientId}.md`, `docs/compliance/`를 함께 확인한다.

## Dashboard Rules

- `index.html`은 CDN 기반 브라우저 실행 파일이다. Node 빌드 검사가 대시보드 JSX까지 검증해주지 않는다는 점을 감안한다.
- 화면 문구를 바꿀 때는 대시보드 사용자에게 보이는 운영 표현으로 작성한다. 내부 구현 설명이나 사용법 문장을 불필요하게 늘리지 않는다.
- 대시보드 `ClientBriefs`는 빠른 운영 입력용이다. 긴 거래처 맥락, 원본 자료, 작성 패턴은 `docs/briefs/`에 보관한다.
- 거래처 카드, 손익, 비용, 글 보관함, 캘린더는 Apps Script `summary` 응답과 연결되어 있다. 관련 데이터 구조를 바꾸면 `docs/apps-script-api.md`를 확인하고 함께 수정한다.
- 서브/순위 업무의 `진행중/완료` 상태는 Sheets `MonthlyJobs.note`를 기준으로 동기화한다. `localStorage`는 화면 즉시 반영과 API 응답 전 임시 항목 보존용으로만 사용한다.
- 월간 초기화는 자동 실행하지 않는다. 월말보고서 작성을 끝낸 뒤 대시보드 `다음 달 초기화` 버튼으로만 실행하며, 버튼은 Sheets 처리 성공을 확인한 후 로컬 상태를 초기화해야 한다.
- `reportFile` 값은 `btskin.html` 같은 루트의 상대 HTML 파일명으로 둔다. `https://...`, `file://...`, `/absolute/path` 형식은 로컬/Pages 동기화를 깨뜨리므로 사용하지 않는다.
- UI 변경이 있으면 로컬 `index.html` 직접 열기 기준을 먼저 고려하고, 외부 접속 검증이 필요할 때 GitHub Pages 배포 화면을 확인한다.

## Report Rules

- 거래처별 보고서 HTML은 기존 월별 탭을 누적하는 방식으로 관리한다.
- 새 월 보고서는 기존 최신 월 탭 위에 추가하고, 새 월을 기본 active 탭으로 둔다.
- 기존 월 보고서 내용은 삭제하지 않는다.
- 발행 내역은 API/Sheets `postLogs`를 기준으로 하되, 보고월의 `YYYY-MM` 필터를 적용한다.
- 월말보고서 작성 전 `npm.cmd run report:context -- --month YYYY-MM --client CLIENT_ID` 또는 `--all`을 실행해 보고서용 컨텍스트를 먼저 생성한다.
- `btskin`/`belrmon` 보고서는 상담내역 표준경로 확인을 위해 가능하면 `npm.cmd run report:consults:check -- --month YYYY-MM --client btskin,belrmon`도 먼저 실행한다.
- 생성된 `.report-context/` 파일은 업무/발행/상담/비용/계약/리뷰 데이터를 정규화한 작업용 스냅샷이며 커밋하지 않는다.
- 외부 콘솔 수치가 없으면 임의로 만들지 않는다. 필요한 항목은 “자료 확인 필요”로 남기거나 사용자에게 요청한다.

## Review Monitor Rules

- 네이버 리뷰 모니터링은 방문자 리뷰만 대상으로 한다. 블로그 리뷰는 저장하거나 비교하지 않는다.
- 현재 자동 모니터링 대상은 `btskin`, `belrmon`, `kyunghee`다.
- `check_failed` 상황에서는 `ReviewTargets.savedVisitorReviewCount`를 변경하지 않는다.
- 리뷰 수가 증가했으나 본문 확인이 막힌 경우에도 증가 사실은 `increased_but_blocked`로 기록하고 기준 리뷰 수는 현재 확인값으로 갱신한다.
- 텔레그램 설정이 없으면 스크립트는 `[codex-notice]` 로그로 요약을 남긴다.
- 실제 운영 데이터 갱신 전에는 가능하면 `review:monitor:dry-run`을 먼저 실행한다.

## Commands

PowerShell에서 `npm` 실행 정책 오류가 나면 `npm.cmd`를 사용한다.

```bash
npm run check
npm run check:syntax
npm run report:consults:check -- --month YYYY-MM --client btskin,belrmon
npm run report:context -- --month YYYY-MM --client btskin
npm run review:monitor:check
npm run review:monitor:dry-run
npm run review:monitor
```

- `npm run check`: 저장소 운영 점검 전체 실행.
- `npm run check:syntax`: 리뷰 모니터 스크립트 문법 검사.
- `npm run report:consults:check -- --month YYYY-MM --client btskin,belrmon`: 월말보고서 상담내역 API 표준경로 확인.
- `npm run report:context -- --month YYYY-MM --client btskin`: 월말보고서 작성용 거래처 데이터 스냅샷 생성.
- `npm run review:monitor:check`: `check:syntax`와 동일한 호환 명령.
- `npm run review:monitor:dry-run`: Apps Script 갱신 없이 모니터 로직 확인.
- `npm run review:monitor`: 실제 Apps Script 데이터를 갱신할 수 있는 운영 명령.

## Completion Checklist

- 변경 범위가 요청과 직접 관련된 파일에 제한되어 있는지 확인한다.
- 이전 AI 도구명 표현을 새로 추가하지 않는다.
- API 계약이나 시트 구조 변경이 있으면 `docs/apps-script-api.md`를 갱신한다.
- 리뷰 모니터 코드, 문서, 파일 구조, 보고서 파일명을 바꾸면 `npm.cmd run check` 또는 `npm run check`를 실행한다.
- 문서나 UI에서 파일 경로를 바꿨다면 `rg`로 이전 경로/파일명 잔여 참조를 확인한다.

---
> Source: [urbanplanad-source/dashboard-urban](https://github.com/urbanplanad-source/dashboard-urban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
