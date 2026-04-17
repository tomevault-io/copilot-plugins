## content-collector

> YouTube · Threads · Instagram 콘텐츠를 자동/수동으로 수집하고, AI로 분석해서 Notion에 날짜별로 정리하는 시스템.

# Content Collector — Claude 작업 가이드

## 프로젝트 한 줄 요약
YouTube · Threads · Instagram 콘텐츠를 자동/수동으로 수집하고, AI로 분석해서 Notion에 날짜별로 정리하는 시스템.

---

## 핵심 아키텍처

```
URL 입력 (스케줄 or 봇)
  → platformDetect.js   (어떤 플랫폼인지 판별)
  → scrapers/*.js       (플랫폼별 원본 데이터 수집)
  → analyzer/ai.js      (Claude Haiku로 요약 + 태그)
  → notion/client.js    (날짜 페이지 → 플랫폼 섹션 → 블록)
  → notify.js           (텔레그램/디스코드 완료 알림)
```

## 두 가지 수집 모드

| 모드 | 파일 | 동작 |
|---|---|---|
| Mode A (스케줄) | `scheduler/cron.js` | KST 09:00 / 14:00 / 19:00 / 23:00 자동 실행 |
| Mode B (봇) | `bots/telegram.js`, `bots/discord.js` | 링크 전송 시 즉시 처리 |

두 모드 모두 `utils/processUrl.js`의 `processUrl()` 함수로 합류함.

---

## 코드 작성 원칙

### 파일 수정 전 반드시 확인
- `utils/processUrl.js` — 수집 흐름의 진입점. 변경 시 양쪽 모드에 모두 영향.
- `notion/client.js` — Notion 블록 구조. 수정 시 기존 저장 형식과 호환 유지.
- `utils/platformDetect.js` — URL 파싱 로직. 새 플랫폼 추가 시 여기 먼저 등록.

### 환경변수 패턴
- 필수: `NOTION_TOKEN`, `NOTION_ROOT_DB_ID`, `ANTHROPIC_API_KEY`
- 선택: 텔레그램/디스코드/YouTube API — 없어도 기본 동작 유지
- 새 환경변수 추가 시 반드시 `.env.example`에도 추가

### 에러 처리 규칙
- 스크래퍼 오류 → console.warn + 빈 값 반환 (전체 파이프라인 중단 금지)
- AI 분석 오류 → fallback 값으로 계속 진행 (Notion 저장 우선)
- Notion 저장 오류 → 재시도 없이 즉시 throw (사용자에게 알림)

### 비동기 패턴
- 스케줄 수집: 계정별 순차 처리 (API rate limit 방지)
- 단건 URL 처리: scrape → analyze → save 순차 await
- 알림 발송: `Promise.allSettled` (한쪽 실패해도 다른 쪽 전송)

---

## 플랫폼별 제약 사항

| 플랫폼 | 공식 API | 자동 계정 수집 | 댓글 필터 방법 |
|---|---|---|---|
| YouTube | ✅ Data API v3 | ✅ (API 키 필요) | channelId 일치 확인 |
| Threads | ❌ 미제공 | ⚠️ HTML 파싱만 | 현재 미구현 |
| Instagram | ⚠️ 제한적 | ⚠️ HTML 파싱만 | 현재 미구현 |

---

## Notion 저장 구조

```
Root DB (날짜 목록)
  └── 2025-03-25 (화)          ← 날짜 페이지 (자동 생성)
        ├── Topics: AI마케팅·숏폼  ← AI가 자동 누적
        ├── ## YouTube
        │     └── [callout] 영상제목
        │           ├── 요약
        │           ├── 핵심포인트
        │           └── 태그 | 수집방법
        ├── ## Threads
        └── ## Instagram
```

날짜 페이지는 `datePageCache` Map으로 캐시됨 — 같은 날 중복 생성 방지.

---

## 자주 쓰는 테스트 명령

```bash
# 단건 URL 수동 테스트
curl -X POST http://localhost:3000/trigger \
  -H "Content-Type: application/json" \
  -d '{"url":"<테스트 URL>"}'

# 헬스체크
curl http://localhost:3000/health

# 스케줄 즉시 실행 (테스트용 — cron.js에서 직접 호출)
node -e "require('./src/utils/processUrl').runCollectionCycle({ youtube:[], threads:[], instagram:[] })"
```

---

## 새 스크래퍼 추가 방법

1. `src/scrapers/새플랫폼.js` 생성
2. `utils/platformDetect.js`에 URL 패턴 추가
3. `utils/processUrl.js`의 switch 문에 케이스 추가
4. `.env.example`에 필요한 환경변수 추가
5. `README.md` 업데이트

---

## skills/ 폴더 안내

| 파일 | 용도 |
|---|---|
| `skills/add-scraper.md` | 새 플랫폼 스크래퍼 추가 절차 |
| `skills/add-notion-field.md` | Notion 저장 필드 추가/수정 절차 |
| `skills/debug-collection.md` | 수집 오류 디버깅 체크리스트 |
| `skills/deploy-update.md` | Railway/Render 배포 업데이트 절차 |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lodaebone6980) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
