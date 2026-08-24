## community-trend-crawler

> 커뮤니티 사이트 인기글을 수집해 Supabase에 저장하는 크롤러.

# community-trend-crawler

커뮤니티 사이트 인기글을 수집해 Supabase에 저장하는 크롤러.
GitHub Actions로 주기적 실행.

## 참조 워크플로우

이 프로젝트는 StartHub 서비스의 커뮤니티/키워드/뉴스 크롤러로 운영된다.
작업 승인, 검증, diff review-only, allowed files stage, commit/push 기준은 StartHub의 워크플로우 문서를 따른다.

- [../StartHub/docs/workflow/ai-workflow.md](../StartHub/docs/workflow/ai-workflow.md)
- [../StartHub/docs/workflow/ai-workflow-git.md](../StartHub/docs/workflow/ai-workflow-git.md)

단, 운영 DB 마이그레이션, 운영 데이터 삭제/복구, secret/env 변경, 외부 서비스 설정 변경은 별도 사용자 확인 후 진행한다.

## 트리거 방식 (2026-05-05~, 2026-07 단일화)

정기 실행원은 **cron-job.org → GitHub `workflow_dispatch` API** 하나로 **단일화**한다.
GitHub Actions 무료 정시 cron 큐 지연(최대 3시간+) 회피가 도입 배경이다.

**2026-07 변경**: crawl.yml 의 GitHub 자체 `on.schedule`(`17`/`47`)을 **제거**했다.
cron-job.org 와 동일 :17/:47·동일 모드로 **이중 트리거**되어 news_top 을 연속 저장
(last-write-wins)·movement 왜곡·큐 적체를 유발했기 때문이다(실운영 재현). 이제
정기 실행은 cron-job.org 의 workflow_dispatch 만이 권위 실행원이다.

- **권위 정기 실행원 = cron-job.org**: 매시 KST HTTP POST → workflow_dispatch
  - **호출 endpoint**: `POST https://api.github.com/repos/kskedu/community-trend-crawler/actions/workflows/crawl.yml/dispatches`
  - **Headers**: `Authorization: Bearer <PAT>` + `Accept: application/vnd.github.v3+json` + `Content-Type: application/json`
  - **PAT**: classic, scopes: `repo` (private 레포)
  - **Body 계약(중요)**: **`:17` 호출 → `{"ref":"main","inputs":{"mode":"full"}}` / `:47` 호출 → `{"ref":"main","inputs":{"mode":"news_top_only"}}`**.
    - crawl.yml 은 `inputs.mode` 로만 모드를 판정한다(schedule 제거로 `github.event.schedule` 분기 소멸). mode 를 생략(`{"ref":"main"}`)하면 crawl.yml 이 안전하게 `full` 로 정규화하지만, 그러면 :47 의 news_top_only 갱신 cadence 가 사라지므로 **두 job 이 각각 올바른 mode 를 전달해야 한다.**
    - ⚠️ cron-job.org 실제 설정은 이 저장소로 확인할 수 없다 → 아래 "외부 확인 게이트" 참조.
- **보조 실행원 = healthcheck.yml (stale 복구 전용)**: 매시 :23, `community_posts` stale(≥120m)/down 시에만 crawl.yml 을 workflow_dispatch. **dispatch 전 crawl.yml 의 queued/in_progress run 을 확인**해 이미 실행 중이면 생략한다(중복 완화, check-then-act·비원자적). run 조회 실패 시 **fail-closed**(dispatch 생략). 텔레그램 알림 유지.
- **수동 실행 = 비상·검증 전용**: crawl.yml workflow_dispatch(UI 또는 `gh workflow run`, mode 선택, default full).

### 장애 시 복구 절차
1. GitHub Actions 상태(githubstatus.com Actions)가 partial_outage/major_outage 면 **두 workflow 를 Disable**(`gh workflow disable crawl.yml healthcheck.yml`)해 큐 적체·유입을 멈춘다.
2. 정체된 queued/in_progress run 을 확인(`gh run list --status queued/in_progress`)하고 필요 시 취소.
3. Actions 가 Operational 로 복귀하고 queued/in_progress 0 인지 확인.
4. cron-job.org 설정(아래 게이트)을 확인한 뒤 crawl.yml 만 Enable → 다음 cron-job.org dispatch 1건 자연 관찰 → 정상 확인 후 healthcheck.yml Enable.

### 외부 확인 게이트 (merge·Enable 전 필수)
cron-job.org 설정은 코드로 검증 불가. 재활성화 전 **사용자가 cron-job.org 콘솔에서** 확인:
- `:17` job 이 `inputs.mode=full` 을 전달하는지
- `:47` job 이 `inputs.mode=news_top_only` 를 전달하는지
- 대상 `ref=main` 인지
- 동일 시각 중복 등록된 cron job 이 없는지
- 인증 PAT 가 유효한지

### 잔여 P2 후속 과제 (이번 단일화로 미해소)
1. **healthcheck 중복 확인의 비원자성**: check-then-act 라 두 호출 동시 관측 시 이중 dispatch 가능(정상 케이스 완화일 뿐, 0건 보장 아님).
2. **full ↔ news_top_only shared news_top 경합 — 앱단 freshness guard 로 완화 완료 (2026-07-20, `e7240e3`)**: 아래 "news_top freshness guard" 절 참조. **완전 원자화는 미해소**(TOCTOU 잔존, 항목 3 참조).
3. **DB optimistic guard / 실행 세대 보호 — 근본(원자) 해결은 미착수**: 현재는 앱단에서 "실행 시작 시점에 읽은 previous" 와 비교하는 **best-effort** 뿐이라, read~write 사이 다른 run 이 더 최신을 write 하는 순수 TOCTOU 는 못 막는다. 근본 해결은 upsert 를 **DB RPC 조건부 write**(`generated_at` compare-and-set)로 바꾸는 것 — service_role/migration 동반하는 별도 PR. concurrency(모드별 group, `cancel-in-progress:false`)는 **healthcheck 복구 dispatch 의 pending 취소(유실) 위험** 때문에 채택하지 않기로 결정함(2026-07-20 계획 리뷰, GitHub Actions 표준 concurrency 는 새 pending 이 기존 pending 을 취소).

### news_top freshness guard (2026-07-20, `e7240e3`)
`run_news_briefing()` 이 upsert 직전 새 payload 의 `generated_at` 을 직전에 읽은 `previous.generated_at` 과 비교해, **새 값이 previous 보다 명확히 과거일 때만** upsert 를 건너뛴다(오래된 실행이 최신 news_top 을 덮어쓰는 것 방지). 동일 시각·최신·결측·비문자열·파싱 실패는 전부 write 허용(fail-open) — 정상 신선 실행을 막지 않는다.
- helper: `main.py` `_parse_generated_at` / `_is_stale_news_top_write` (순수 함수, 테스트 가능).
- stale 시: news_top upsert 만 생략. `keyword_cache`·`community_posts` 경로는 영향 없음. 후보/decisions/rejection_counts 는 그대로 진단 DB 에 보존되고, `status=skipped` / `skip_reason=STALE_WRITE_SKIPPED`(`news/diagnostics.py` `SKIP_REASON_STALE_WRITE`) 로 기록돼 **발행 성공 집계에서 제외**된다.
- 근거는 `thresholds` JSONB 의 `stale_write_v1` namespace 에 저장(`previous_generated_at`/`new_generated_at`/`mode`/`run_id`/`comparison`/`result`, secret·기사 본문 없음).
- **선행조건**: StartHub `news_keyword_runs.skip_reason` CHECK 에 `STALE_WRITE_SKIPPED` 가 등록돼 있어야 한다(StartHub PR #73, migration `docs/migrations/supabase-news-diag-skip-reason-stale-*.sql`, 운영 적용·postcheck 통과 완료). 이 CHECK 없이 guard 가 이 값을 보내면 RPC INSERT 가 CHECK 위반으로 **진단 적재 전체가 조용히 실패**한다 — 그래서 migration 을 먼저 적용하고 crawler guard(`e7240e3`, PR #14) 를 나중에 merge하는 순서 게이트를 지켰다.
- **한계**: 완전한 원자적 경합 해결이 아니다("실제 경합 축소"가 아니라 "이미 관측된 최신값보다 오래된 실행의 후행 write 차단"). 위 항목 3 참조.

## 구조

```
community-trend-crawler/
├── main.py              # 진입점, 스크래퍼 + 키워드 크롤러 통합 실행
├── models.py            # Post 데이터 모델
├── config.py            # 공통 설정 (Chrome 12종 헤더, 타임아웃 등)
├── scrapers/            # 커뮤니티 게시글 크롤러
│   ├── base.py          # BaseScraper (fetch, fetch_bytes, fetch_og_image)
│   ├── clien.py · ruliweb.py · ppomppu.py · mlbpark.py
│   ├── bobaedream.py · inven.py · dcinside.py · humoruniv.py
│   ├── theqoo.py · slrclub.py · todayhumor.py · etoland.py
│   ├── cook82.py · instiz.py · ygosu.py · natepann.py
│   └── (fmkorea.py, ddanzi.py — 비활성)
├── keywords/            # 검색엔진 실시간 키워드 크롤러
│   ├── base.py          # BaseKeywordScraper (active 플래그로 optional/degraded 소스 skip)
│   ├── danawa.py        # 다나와 인기 키워드 Top 10
│   ├── daum.py          # 다음 실시간 트렌드 Top 10
│   ├── daangn.py        # 당근마켓 인기 검색어 Top 10 (gnb_popular_keyword 앵커 파싱)
│   ├── nate.py          # 네이트 실시간 이슈 키워드 Top 10 (jsonLiveKeywordDataV1.js EUC-KR JSON 파싱)
│   ├── msn.py           # MSN 최신 인기 검색어 Top 10 (api.msn.com trendingsearch 비공식 JSON 파싱)
│   └── namuwiki.py      # 비활성(active=False) — namu.news 서비스 종료, 대체 upstream 없음
├── processor/
│   ├── dedup.py         # URL 기반 중복 제거
│   ├── filter.py        # 광고/공지/노이즈 필터
│   └── scorer.py        # 점수 계산
└── db/
    └── supabase.py      # upsert_posts, upsert_keywords
```

## 스크래퍼 현황

| 사이트 | ID | 상태 | 비고 |
|---|---|---|---|
| 클리앙 | clien | ✅ | og:image |
| 루리웹 | ruliweb | ✅ | |
| 뽐뿌 | ppomppu | ✅ | hot.php 전체 인기글 |
| 엠팍 | mlbpark | ✅ | |
| 보배드림 | bobaedream | ✅ | |
| 인벤 | inven | ✅ | |
| 디씨인사이드 | dcinside | ✅ | |
| 웃긴대학 | humoruniv | ✅ | |
| 더쿠 | theqoo | ✅ | og:image |
| SLR클럽 | slrclub | ✅ | EUC-KR, 자체 이미지 |
| 오늘의유머 | todayhumor | ⚠️ | EUC-KR. 국내 IP 정상, GH Actions 등 해외 IP에서 403 가능 — 발생 시 source-level skipped, 전체 실패로 안 번짐 |
| 이토랜드 | etoland | ✅ | `/hit/list` (UTF-8). 리스트 썸네일 우선, 부족 시 og:image 폴백 |
| 82쿡 | 82cook | ✅ | best_article.php |
| 인스티즈 | instiz | ✅ | |
| 와고 | ygosu | ✅ | 베스트 daily |
| 네이트판 | natepann | ✅ | UTF-8, 대문 '톡커들의 선택' Top 40 (talkerChoiceArea0/1) |
| 에펨코리아 | fmkorea | ❌ | 봇 차단(430) |
| 딴지일보 | ddanzi | ❌ | 제거 |

## 키워드 스크래퍼 (keywords/)

검색엔진 실시간 키워드 수집 → Supabase `keyword_cache`. StartHub 프론트는
이 테이블을 직접 조회해 즉시 표시 (Vercel 함수 미경유).

| 소스 | ID | 대상 URL | 비고 |
|---|---|---|---|
| 다나와 | danawa | `/dsearch.php?query=best` | `hot_keyword` 섹션 파싱. Vercel은 403 차단되어 GH Actions(한국 친화 IP)로 이관 |
| 다음 | daum | `/search?w=tot&q=ㄴㄴ` | `list_trend` 내 `data-keyword` 추출 |
| 네이트 | nate | `/js/data/jsonLiveKeywordDataV1.js` | 네이트 메인 홈 `#olLiveIssueKeyword`는 5개씩 두 페이지로 로테이션 렌더링(정적 HTML엔 5개만 존재)돼 자체 JS(`nate_general_v20260202.js`)가 호출하는 이 비공식 JSON 엔드포인트를 직접 사용 — EUC-KR 인코딩, `[rank, title, flag, delta, 검색어]` 배열 10개. `item[4]`를 keyword로 사용 |
| MSN | msn | `api.msn.com/news/feed/segments/trendingsearch` | msn.com/ko-kr은 완전 SPA(React)라 정적 HTML엔 위젯이 없음 — 실제 브라우저 네트워크 캡처로 찾은 비공식 API. `Referer: https://www.msn.com/` 헤더만 있으면 세션/쿠키 불필요. 응답의 `data` 필드가 JSON 문자열로 이중 인코딩돼 있어 한 번 더 `json.loads` 필요. `IsAds` 광고 항목 제외 후 `Score` 내림차순 Top 10. 키워드 클릭은 Bing 검색(`bing.com/search?mkt=ko-kr&q=`)으로 연결 — MSN 자체 검색창도 Bing으로 위임하는 것 확인 |
| 당근마켓 | daangn | `/kr/buy-sell/` | 헤더 네비 `data-gtm="gnb_popular_keyword"` 앵커 텍스트 파싱. href의 `in={지역코드}`는 요청 IP 기준 지역값이라 저장 URL에서 제거하고 `search=` 파라미터로 재조립 |
| 나무위키 | namuwiki | (비활성) | `namu.news` 2026-06 서비스 종료(복구 시도 금지). 대안으로 `namu.wiki` 본사이트 우측 "실시간 검색어" 위젯을 검토했으나 raw HTML(SSR)에 미포함 — 클라이언트 JS 렌더링 전용이라 브라우저 크롤링 없이는 수집 불가. `keywords/namuwiki.py`의 `active=False`로 표시, `main.py` run()에서 skip |

## Supabase DB 스키마

### community_posts

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | PK |
| title | text | 게시글 제목 |
| content | text | 본문 (일부 사이트만) |
| image_url | text | 썸네일 이미지 URL |
| source_url | text | 원본 게시글 URL (upsert 기준) |
| source_site | text | 커뮤니티 ID (`clien`, `ruliweb`, `theqoo`, `ygosu` 등) |
| upvotes | integer | 추천 수 |
| comments | integer | 댓글 수 |
| views | integer | 조회 수 |
| score | double precision | 계산된 인기 점수 (processor/scorer.py) |
| img_hash | text | 이미지 해시 (중복 판별용) |
| created_at | timestamptz | DB insert 시각 |
| collected_at | timestamptz | 크롤링 수집 시각 |
| click_count | integer | 프론트 클릭 수 |
| fav_count | integer | 프론트 즐겨찾기 수 |

- **upsert 키**: `source_url`
- **`collected_at` 갱신**: upsert 시 `datetime.now(UTC)`를 매번 명시 주입.
  과거에 필드 누락으로 상위 고정 인기글(엠팍 등)이 실시간/단기 range에서
  누락되는 버그 있었음 — [db/supabase.py](db/supabase.py) `upsert_posts` 참조
- **프론트 조회**: [StartHub/js/community.js](../StartHub/js/community.js)에서 `source_site` 필터 + `score/comments/views` 정렬

### keyword_cache
검색엔진 실시간 키워드. `keywords/` 크롤러가 30분 주기로 upsert.
`namuwiki`는 2026-07 이후 비활성(source 미실행)이라 신규 upsert가 없음 — 기존 row는
TTL 없이 남아있는 stale 데이터이므로 `updated_at` 기준으로 최신 여부를 판단해야 함
(후속 이슈: TTL 미도입).

| 컬럼 | 타입 | 설명 |
|---|---|---|
| source | text (PK) | `danawa` / `daum` / `daangn` / `nate` / `msn` / (`namuwiki`, 비활성) |
| keywords | jsonb | `[{keyword, url}, ...]` |
| updated_at | timestamptz | 마지막 수집 시각 |

- **upsert 키**: `source`
- **프론트 조회**: [StartHub/js/app.js](../StartHub/js/app.js) `_fetchKeywordCache()`에서 Supabase 직접 조회

## 필터링 정책 (processor/filter.py)

### 관리 방식 (2026-05-05~)
- **DB 우선**: Supabase `trend_block_keywords` 테이블에서 `enabled=true` 항목 로드
- **Fallback**: DB 조회 실패 시 filter.py 하드코딩 목록 사용
- **어드민 관리**: [StartHub/admin/trends.html](../StartHub/admin/trends.html) > `필터 관리` 탭에서 CRUD
- **이력 기록**: 추가/삭제 시 `trend_block_keyword_logs` 테이블에 자동 기록
- DB 스키마: [StartHub/docs/supabase-trend-block-keywords-migration.sql](../StartHub/docs/supabase-trend-block-keywords-migration.sql)

### 제목 길이
- 5자 이하 제목 전부 제거 (코드 고정, DB 비관리)

### 차단 키워드 / 패턴
→ Supabase `trend_block_keywords` 테이블에서 관리 (어드민 UI)
- type `keyword`: 제목에 포함 시 차단
- type `pattern`: 정규식 매칭 차단

### 오탐 위험으로 미포함
- 수익, 재테크, 코인, 이벤트, 판매, 공구, 직구

## 트러블슈팅 이력

- **2026-05-08 etoland 0건 수집 이슈**
  - 증상: DB에 `etoland` 글이 0건 누적 — 단기/일간/주간 모두 비어 있음
  - 원인: etoland 사이트 개편으로 `/bbs/hit.php?wr_id=` URL 패턴이 사라지고 `/hit/list` + `/hit/{board}/view/{slug}-{id}` 구조로 변경
  - 해결: [scrapers/etoland.py](scrapers/etoland.py) 리스트 URL 변경 + 새 DOM 셀렉터(`a[href*="/hit/"][href*="/view/"]`, `span.truncate`, `span.comment-s`, `div.caption-m`)에 맞춰 파싱 재작성. 인코딩도 EUC-KR → UTF-8

## range별 데이터 흐름 (참고)

크롤러는 1시간 주기로 모든 사이트의 인기글 페이지만 수집한다. 사이트의 일/주간 카테고리를 별도로 호출하지 않음. 단기/12h/일간/주간은 프론트가 `collected_at` 기준으로 필터.
- 스크래퍼: 매시 인기글 페이지 1번 수집 → upsert(`source_url` PK, `collected_at` 매번 갱신)
- 프론트([StartHub/js/community.js](../StartHub/js/community.js)): `collected_at >= now - {3h,12h,24h,7d}` 필터 + `score desc, collected_at desc` 정렬

따라서 일/주간이 비어 보인다면:
1. 스크래퍼 자체가 실패해 0건 누적 (예: etoland 2026-05 이슈) — 로그/DB count 확인
2. 사이트 게시글 수가 적어 단순히 점수 정렬에서 다른 사이트에 밀림 — 정상

---
> Source: [kskedu/community-trend-crawler](https://github.com/kskedu/community-trend-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
