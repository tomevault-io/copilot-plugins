## supabase-hermit

> 이 프로젝트는 은둔마을 앱/웹이 공유하는 **Supabase DB 스키마의 단일 정본(Single Source of Truth)**.

# CLAUDE.md — 은둔마을 Supabase 중앙 프로젝트

이 프로젝트는 은둔마을 앱/웹이 공유하는 **Supabase DB 스키마의 단일 정본(Single Source of Truth)**.

## 구조

```
supabase-hermit/
├── supabase/
│   ├── config.toml                 # Supabase 프로젝트 설정
│   └── migrations/                 # 모든 마이그레이션 원본 (37개)
│       ├── 20260301000001_schema.sql              # 베이스라인: 테이블/함수/뷰/트리거/인덱스
│       ├── 20260301000002_rls.sql                 # 베이스라인: RLS 정책
│       ├── 20260301000003_infra.sql               # 베이스라인: 권한(grants) + Storage
│       ├── 20260302000000_fix_rls_update_policies.sql  # UPDATE 정책 수정
│       ├── 20260303000001_core_redesign.sql       # 리액션 RPC, 소프트삭제, FK CASCADE, 제약조건
│       ├── 20260303000002_fix_group_members_recursion.sql  # (레거시) RLS 자기참조 재귀 수정
│       ├── 20260303000003_post_update_reanalysis.sql  # 글 수정 시 감정분석 자동 재실행
│       ├── 20260304000001_fix_reactions_data.sql      # 리액션 데이터 정합성 수정
│       ├── 20260306000001_remove_author_column.sql    # author 컬럼 제거
│       ├── 20260307000001_recommendation_improvements.sql  # 추천 개선 (트렌딩, 감정 폴백, 시간 감쇠)
│       ├── 20260308000001_ux_redesign.sql             # UX 리디자인: initial_emotions, user_preferences, 감정 RPC들
│       ├── 20260309000001_security_performance_fixes.sql  # 보안/성능: boards RLS, 인덱스, RPC 최적화, 제약조건
│       ├── 20260310000001_comprehensive_improvements.sql  # 공개 게시판, 뷰 최적화, 검색 RPC, 동시성/관리자 삭제
│       ├── 20260311000001_fix_rpc_missing_columns.sql     # get_posts_by_emotion RPC 컬럼 보강
│       ├── 20260311000002_fix_search_posts_columns.sql    # search_posts RPC 컬럼 보강
│       ├── 20260311000003_analysis_status_retry.sql       # 감정분석 상태 추적 + 재시도 (status/retry_count/error_reason)
│       ├── 20260312000001_search_v2.sql                  # 검색 v2: 풀텍스트 + 관련도 + 하이라이트 + 감정 필터
│       ├── 20260313000001_admin_groups_rls_fix.sql       # (레거시) groups UPDATE/DELETE RLS + invite_code CHECK
│       ├── 20260314000001_drop_search_posts_v1.sql       # search_posts v1 제거 (deprecated)
│       ├── 20260315000001_search_v2_ilike_escape.sql    # search_posts_v2 ILIKE 와일드카드 이스케이프
│       ├── 20260315000002_fix_search_v2_column_order.sql # search_posts_v2 CTE 컬럼 순서 수정
│       ├── 20260316000001_cleanup_stuck_analyses.sql     # 감정분석 stuck 상태 자동 정리 함수
│       ├── 20260317000001_post_analysis_rls.sql         # post_analysis SELECT: 인증된 사용자만
│       ├── 20260317000002_reactions_rls_cleanup.sql      # reactions/user_reactions 직접 쓰기 정책 제거
│       ├── 20260318000001_boards_constraints.sql        # boards 이름/설명 길이 CHECK 제약조건
│       ├── 20260319000001_remove_group_board_system.sql # 그룹/게시판 시스템 완전 제거
│       ├── 20260320000001_advisor_performance_security.sql # RLS initplan 최적화 + search_path + extension 이동
│       ├── 20260321000001_admin_cleanup_test_data.sql    # 관리자 전용 테스트 데이터 정리 RPC
│       ├── 20260322000001_fix_analysis_cooldown.sql      # 감정분석 쿨다운 버그 수정 (analyzed_at NULL 허용)
│       ├── 20260323000001_daily_post.sql                # 오늘의 하루: posts 확장, 트리거 조건, 뷰 재생성, RPC 3개
│       ├── 20260324000001_daily_insights.sql            # 나의 패턴: 활동-감정 인사이트 RPC
│       ├── 20260325000001_anonymous_alias.sql           # v2: 고정 익명 별칭 + 자동 부여 트리거
│       ├── 20260325000002_comment_replies.sql           # v2: 댓글 답글 (parent_id, 1단계)
│       ├── 20260325000003_notifications.sql             # v2: In-App 알림 + 자동 생성 트리거
│       ├── 20260325000004_user_blocks.sql               # v2: 사용자 차단
│       ├── 20260326000001_v2_improvements.sql           # v2 점검: 타임존 수정, 별칭 레이스컨디션, 알림/차단 개선
│       ├── 20260327000001_v3_refinement.sql            # v3 정비: post_type CHECK, 알림 인덱스, 별칭 partial unique, analyzed_at 기본값, 답글 검증 트리거
│       ├── 20260328000001_mypage_rpc_optimization.sql  # v4: RPC KST 타임존 + INVOKER + 범위 비교 최적화
│       ├── 20260329000001_daily_evolution.sql          # daily Evolution: generated column, 별칭, RPC 신규 3개, insights KST, custom_activities
│       ├── 20260329000002_fix_today_daily_reactions.sql # get_today_daily: heart만 + comment_count
│       ├── 20260329000003_side_effects_fix.sql         # soft_delete 알림 정리, daily 재작성 방어
│       ├── 20260329000004_pg_cron_setup.sql            # pg_cron: stuck 분석 5분 자동 정리
│       ├── 20260329000005_streak_rewards.sql           # 스트릭 보상: get_my_streak RPC + 마일스톤
│       ├── 20260330000001_legacy_cleanup.sql           # 레거시 정리: user_blocks 인덱스 + RPC COMMENT 문서화 (33개)
│       ├── 20260331000001_daily_v2.sql                # Daily v2: 히스토리 조회 + 월간 감정 리포트
│       ├── 20260401000001_block_user_defensive.sql    # block_user 방어적 처리: 별칭 미존재 시 NOOP + 자기 차단 방지
│       ├── 20260402000001_poetry_board.sql            # 시 게시판 추가 (board_id=13, always_anon)
│       ├── 20260402000002_remove_image_feature.sql    # 이미지 기능 제거: image_url DROP, Storage 정책 제거, 뷰 재생성
│       ├── 20260403000001_remove_poetry_board.sql     # 시 게시판 제거: 게시글 자유게시판 이관 + board 삭제
│       ├── 20260404000001_lint_fixes.sql              # lint 수정: search_posts_v2 image_url 제거, admin_cleanup author_id
│       └── 20260404000002_lint_fixes_2.sql            # lint 수정: get_my_streak STABLE→VOLATILE
├── shared/
│   ├── constants.ts                # 공유 상수 (ALLOWED_EMOTIONS, EMOTION_EMOJI, SEARCH_SORT_OPTIONS, SEARCH_CONFIG, ANALYSIS_STATUS/CONFIG, VALIDATION, MOTION, ACTIVITY_PRESETS, DAILY_CONFIG, DAILY_INSIGHTS_CONFIG 등)
│   ├── types.ts                    # 공유 비즈니스 타입 (Post, Comment, Notification, UserBlock, ActivitySummary 등)
│   └── utils.ts                    # 공유 순수 함수 (validatePostInput, validateCommentInput, validateDailyPostInput)
├── types/
│   └── database.gen.ts             # 자동 생성 DB 타입 (gen-types.sh 산출물)
├── scripts/
│   ├── db.sh                       # Supabase CLI 래퍼 (push/pull/diff/lint/status/gen-types)
│   ├── gen-types.sh                # DB 스키마 -> TypeScript 타입 생성
│   ├── sync-to-projects.sh        # 앱/웹 레포로 migrations + config + types 동기화
│   └── verify.sh                   # 3개 레포 간 파일 정합성 검증
├── docs/
│   ├── SCHEMA.md                   # DB 스키마 상세 문서
│   ├── SCRIPTS.md                  # 스크립트 사용법 상세
│   ├── CLIENT-ARCHITECTURE.md      # 앱/웹 클라이언트 연동 아키텍처
│   ├── audit/                      # 코드베이스 분석 스냅샷 (대화 시작 시 참조)
│   │   ├── central-audit.md        # 중앙 레포 현황 (마이그레이션, RPC, 타입, 스크립트)
│   │   ├── web-audit.md            # 웹 레포 현황 (구조, API, 훅, 컴포넌트)
│   │   ├── app-audit.md            # 앱 레포 현황 (구조, API, 훅, 컴포넌트)
│   │   └── tech-debt.md            # 3개 레포 기술부채 + 개선 백로그
│   ├── plan/                       # 진행 예정/진행 중 설계 문서 (활성 문서만)
│   ├── archive/                    # 완료된 설계 문서 아카이브 (complete+backup+memo 통합)
│   └── reference/                  # 외부 참조 (Supabase Advisor, 보류 중 설계)
├── package.json                    # npm scripts 편의용 (의존성 없음)
├── .env                            # SUPABASE_ACCESS_TOKEN (git 제외)
└── CLAUDE.md
```

## DB 스키마 요약

### 테이블 (10개)
| 테이블 | 설명 |
|---|---|
| `boards` | 게시판 (id=1,2,12; 익명모드 설정) |
| `posts` | 게시글 (소프트삭제, 자동 감정분석, initial_emotions, post_type: post/daily, activities, **created_date_kst** generated) |
| `comments` | 댓글 (소프트삭제, **parent_id로 1단계 답글**) |
| `reactions` | 리액션 집계 (post_id + reaction_type 별 count) |
| `user_reactions` | 사용자별 리액션 기록 |
| `post_analysis` | AI 감정 분석 결과 (emotions 배열, status/retry_count/error_reason) |
| `user_preferences` | 사용자 설정 (감정 선호, 테마, 온보딩, **display_alias**, custom_activities, longest_streak, last_milestone) |
| `app_admin` | 앱 관리자 |
| `notifications` | **In-App 알림** (reaction/comment/reply, actor_alias, read) |
| `user_blocks` | **사용자 차단** (blocker_id + blocked_alias) |

### 뷰 (1개)
- `posts_with_like_count` — 게시글 + 좋아요수 + 댓글수 + 감정 + post_type + activities (security_invoker)

### RPC 함수 (35개)
| 함수 | 설명 |
|---|---|
| `toggle_reaction(post_id, type)` | 리액션 토글 (SECURITY DEFINER + advisory lock) |
| `get_post_reactions(post_id)` | 게시글 리액션 + 사용자 상태 조회 |
| `soft_delete_post(post_id)` | 게시글 소프트삭제 (본인 + 관리자) |
| `soft_delete_comment(comment_id)` | 댓글 소프트삭제 (본인 + 관리자) |
| `get_emotion_trend(days)` | 감정 트렌드 집계 (상위 5개, pct 포함) |
| `get_recommended_posts_by_emotion(post_id, limit)` | 감정 기반 추천 (폴백 + 시간 감쇠) |
| `get_trending_posts(hours, limit)` | 트렌딩 게시글 (참여도/시간 가중) |
| `get_posts_by_emotion(emotion, limit, offset)` | 특정 감정 게시글 필터 |
| `get_similar_feeling_count(post_id, days)` | 비슷한 마음 사용자 수 |
| `get_user_emotion_calendar(user_id, start, end)` | 사용자 감정 캘린더 히트맵 |
| `get_emotion_timeline(days)` | 감정 분포 타임라인 |
| `get_my_activity_summary()` | 내 활동 요약 (글/댓글/반응/스트릭) |
| `search_posts_v2(query, emotion, sort, limit, offset)` | 게시글 검색 v2 (풀텍스트 + 관련도 + 하이라이트) |
| `cleanup_stuck_analyses()` | 5분+ stuck 감정분석 자동 failed 전환 |
| `admin_cleanup_posts(user_id?, before?, after?)` | 관리자 전용: 테스트 글 hard delete (CASCADE) |
| `admin_cleanup_comments(user_id?, before?, after?)` | 관리자 전용: 테스트 댓글 hard delete |
| `create_daily_post(emotions, activities?, content?)` | 오늘의 하루 생성 (posts + post_analysis 원자적, 하루 1회 KST) |
| `update_daily_post(post_id, emotions, activities?, content?)` | 오늘의 하루 수정 (posts + post_analysis 동기화) |
| `get_today_daily()` | 오늘(KST) 내 daily 게시글 조회 (홈 진입점용) |
| `get_daily_activity_insights(days?)` | 나의 패턴: 활동-감정 상관관계 (7일+ daily 필요) |
| `generate_display_alias()` | 고정 익명 별칭 생성 (30×30 풀 + 충돌 시 숫자 접미사) |
| `get_my_alias()` | 내 별칭 조회 |
| `get_notifications(limit, offset)` | 알림 목록 조회 (최신순) |
| `get_unread_notification_count()` | 미읽음 알림 수 |
| `mark_notifications_read(ids)` | 선택 알림 읽음 처리 |
| `mark_all_notifications_read()` | 전체 알림 읽음 처리 |
| `block_user(alias)` | 특정 별칭 차단 (미존재 시 NOOP, 자기 차단 방지) |
| `unblock_user(alias)` | 차단 해제 |
| `get_blocked_aliases()` | 차단 별칭 목록 조회 |
| `get_yesterday_daily_reactions()` | 어제 daily 반응 (KST 서버 계산) |
| `get_same_mood_dailies(post_id, emotions)` | 같은 감정 daily 3개 (오늘 KST) |
| `get_weekly_emotion_summary(week_offset)` | 주간 감정 요약 (Top 5 감정, Top 활동) |
| `get_my_streak()` | daily 스트릭 (연속/총/최장 + 마일스톤 + streak freeze) |
| `get_my_daily_history(limit, offset)` | 내 daily 히스토리 (역순, 페이지네이션) |
| `get_monthly_emotion_report(year, month)` | 월간 감정/활동 리포트 (Top 5, 반응 수) |

### Edge Functions (앱 레포에서 관리)
| 함수 | JWT 검증 | 설명 |
|---|---|---|
| `analyze-post` | X | DB Trigger (게시글 INSERT/UPDATE시 자동 호출, **post_type='post'만**, 60초 쿨다운, 서버 재시도 2회) |
| `analyze-post-on-demand` | O | 수동 감정 분석 요청 (fallback + 재시도, 쿨다운 우회, 서버 재시도 2회) |

> **Phase E1 (2026-03-13, 배포 완료)**: 한국어 전문 프롬프트 + Gemini Structured Output 적용. emotions 외에 risk_level/risk_indicators/context_notes를 로그 기록 (DB 저장은 Phase E2 예정). 설계: `docs/plan/DESIGN-emotion-upgrade.md`

## 워크플로

### 1. 새 마이그레이션 작성
이 디렉터리에서만 작성. 앱/웹 레포에서 직접 만들지 않음.

```bash
# 새 SQL 파일 생성 (타임스탬프 형식: YYYYMMDDHHMMSS)
vi supabase/migrations/20260304000001_description.sql

# dry-run 확인
bash scripts/db.sh push --dry-run

# 적용 (자동으로 gen-types -> sync -> verify 실행됨)
bash scripts/db.sh push
```

> `push` 성공 시 자동으로 타입 재생성(`gen-types.sh`) → 앱/웹 동기화(`sync-to-projects.sh`) → 정합성 검증(`verify.sh`)이 실행된다.
> gen-types 실패 시 sync를 건너뛰고 에러를 표시한다.
> 수동 sync가 필요한 경우: `bash scripts/sync-to-projects.sh`

### 2. Remote 변경 사항 가져오기 (Dashboard에서 수정한 경우)
```bash
bash scripts/db.sh pull
```

### 3. 타입만 재생성
```bash
bash scripts/gen-types.sh
# 또는
bash scripts/db.sh gen-types
```

### 4. 상태 확인
```bash
bash scripts/db.sh status   # 로컬/remote 비교
bash scripts/db.sh lint      # RLS/스키마 린트
bash scripts/db.sh verify    # 레포 간 정합성 검증
```

### 5. npm scripts (편의용)
```bash
npm run push          # db push + gen-types + sync + verify
npm run push:dry      # dry-run
npm run pull          # remote -> local
npm run status        # migration 상태
npm run lint          # RLS/스키마 린트
npm run gen-types     # 타입만 재생성 (변경 감지)
npm run sync          # 동기화만 실행
npm run sync:dry      # 동기화 미리보기
npm run verify        # 레포 간 정합성 검증
```

## 동기화 대상

`sync-to-projects.sh`는 다음 6가지를 앱/웹 레포에 복사한다:

| 소스 (중앙) | 앱 대상 | 웹 대상 |
|---|---|---|
| `supabase/migrations/*.sql` | `supabase/migrations/*.sql` | `supabase/migrations/*.sql` |
| `supabase/config.toml` | `supabase/config.toml` | `supabase/config.toml` |
| `types/database.gen.ts` | `src/types/database.gen.ts` | `src/types/database.gen.ts` |
| `shared/constants.ts` | `src/shared/lib/constants.generated.ts` | `src/lib/constants.generated.ts` |
| `shared/types.ts` | `src/types/database.types.ts` | `src/types/database.types.ts` |
| `shared/utils.ts` | `src/shared/lib/utils.generated.ts` | `src/lib/utils.generated.ts` |

옵션: `--app` (앱만), `--web` (웹만), `--dry` (미리보기)

## 규칙

- **마이그레이션은 여기서만 생성** — 앱/웹 레포의 `supabase/migrations/`는 읽기 전용 복사본
- **push 후 sync 자동 실행** — `db.sh push` 성공 시 자동 처리
- **generated types는 수동 types와 공존** — `database.gen.ts`는 자동 생성, 비즈니스 타입은 별도 관리
- **Edge Functions는 앱 레포** — `supabase/functions/`는 앱에서 관리/배포
- **리액션은 RPC만 사용** — `reactions`/`user_reactions` 직접 쓰기 정책 제거됨. `toggle_reaction()` 사용
- **삭제는 소프트삭제** — `posts`, `comments`의 hard DELETE 정책 제거. `soft_delete_post()`, `soft_delete_comment()` 사용
- **멱등 마이그레이션** — `IF NOT EXISTS`, `CREATE OR REPLACE`, `DROP POLICY IF EXISTS` 패턴 적용
- **공유 상수/타입/유틸은 `shared/`에서만 수정** — `shared/constants.ts`, `shared/types.ts`, `shared/utils.ts`는 sync로 앱/웹에 배포. 앱/웹의 generated 파일 직접 수정 금지. `utils.ts`에는 외부 import 없는 순수 함수만 추가할 것
- **Expo SDK 업그레이드 시 `npx expo install --fix` 필수** — `npm install expo@~XX.0.0`만으로는 companion 패키지가 업데이트되지 않아 EAS Build 실패 위험. 반드시 `npx expo install --fix` → `npx expo install --check` → `npx expo-doctor` 순서 실행
- **앱 빌드 전 `bash scripts/pre-build-check.sh`** — SDK 호환성, TypeScript, expo-doctor, 중앙 sync, 테스트 5단계 검증. EAS Build 트리거 전 로컬에서 실행
- **감정 칩 표준 스타일** — 모든 인터랙티브 감정 칩은 `rounded-full px-3 py-1.5 text-xs` 통일. 활성: `EMOTION_COLOR_MAP[emotion].gradient[0]` bg + `font-semibold`, 비활성: 앱 `bg-stone-100 dark:bg-stone-800` / 웹 `bg-muted`. 새 감정 칩 UI 추가 시 이 표준을 따를 것
- **오늘의 하루 (daily post)** — `posts` 테이블의 `post_type='daily'` 경량 게시글. 같은 피드에 표시, 같은 리액션/댓글/Realtime/RLS. 감정분석 트리거는 `post_type='post'`만 발동. daily는 `create_daily_post()` RPC로 post_analysis 직접 생성 (AI skip). 활동 태그는 `ACTIVITY_PRESETS` 상수 기반. 설계: `docs/plan/baj/DESIGN.md`

## 배포 절차 (필수 게이트)

### 사전 점검 (배포 전 반드시 순서대로)
1. `bash scripts/db.sh lint` — DB 스키마 에러 확인
2. 웹: `npx tsc --noEmit` — 타입 체크 통과
3. 앱: pre-commit 훅 (tsc + eslint + prettier + jest) 통과
4. `bash scripts/verify.sh` — 3레포 정합성 확인
5. 앱 스토어 배포 시: `app.config.js`의 `NATIVE_VERSION` 올리기 → 커밋

### 배포 실행
- **DB**: `bash scripts/db.sh push` (자동으로 gen-types → sync → verify)
- **웹**: `git push` 후 반드시 `vercel --prod` 수동 배포 (자동 배포 Canceled 빈번)
- **앱 EAS Build**: 사용자 요청 시에만 실행
  - 먼저 `eas build:list --platform android --status in-progress` 확인
  - 진행 중인 빌드 있으면 `eas build:cancel <id>` 후 재실행
  - `eas build --platform android --profile production`
  - 빌드 완료 후 `eas submit --platform android --profile production --latest`
  - `serviceAccountKeyPath`: `./.key/apps-2182e-4cfe31bef56b.json` (로컬 파일)
- **Edge Functions**: `--no-verify-jwt` 필수

### 에러 발생 시 원칙
- **같은 명령 3번 반복 금지** — 2번 실패하면 멈추고 원인 분석 → 다른 방법 시도
- **재시도 전 기존 작업 취소** — 빌드/배포 중복 실행 금지
- **가용 자원 모두 활용** — WSL, 로컬 PC, 대시보드(Vercel/EAS/Supabase) 등

## 에러 진단 프로토콜

에러 보고 시 **코드 분석 전에 반드시**:
1. 실제 에러 메시지/스크린샷 먼저 확보
2. 배포 상태 확인 (코드 문제가 아닌 배포 누락일 수 있음)
3. 확인된 원인만 타겟 수정 — 추측 기반 대규모 리팩토링 금지

## Claude 역할

- **웹/앱/중앙 3개 프로젝트의 개발 실무자이자 책임자** — 전권 위임
- **권한 범위** — 중앙 통제, 설계(아키텍처/DB 스키마/기술 선택/리팩토링), 구현, 디버깅 모두 자율
- **책임** — 위임된 모든 권한에 대한 결과 책임을 Claude가 짐. 버그, 장애, 설계 오류 발생 시 Claude가 원인 분석 → 수정 → 재발 방지까지 자율 수행
- 사용자 승인 불필요, 자율 판단 후 결과 보고만
- 정보가 필요할 때만 `needs.md`에 작성하여 사용자에게 요청
- 3개 프로젝트 모두 `.claude/settings.json`에 전체 도구 권한 허용 설정 완료

## 문서화 규칙

- **구현 전 설계 문서 작성** — `docs/plan/` 하위에 설계 문서 먼저 작성 후 구현
- **구현 완료 후 문서 갱신** — CLAUDE.md (스키마 요약, RPC 목록 등), docs/SCHEMA.md, MEMORY.md 반영
- **마이그레이션 추가 시** — CLAUDE.md의 마이그레이션 목록 + DB 스키마 요약 즉시 업데이트
- **shared/ 변경 시** — CLAUDE.md의 동기화 대상 + constants/types/utils 설명 업데이트
- **완료된 설계 문서** — `docs/archive/`로 이동
- **메모리 갱신** — 구현 완료된 작업은 MEMORY.md에 요약 기록, architecture.md에 구조 변경 반영
- **audit 갱신** — 작업 완료 후 변경된 레포의 `docs/audit/*.md` + `tech-debt.md` 즉시 업데이트. 새 대화 시작 시 audit 파일 4개를 먼저 읽고 전체 현황을 파악한 후 작업 시작

## needs.md 운용 규칙

- **`needs.md`** — Claude가 작업 중 사용자 정보/판단이 필요할 때 사용하는 소통 채널
- **작성 시점** — 작업 진행이 사용자 입력 없이 불가능할 때 `## 대기 중`에 항목 추가
- **해결 시점** — 사용자가 답변하면 Claude가 `## 해결됨`으로 이동 후 작업 재개
- **정리** — 해결된 항목은 주기적으로 삭제하여 파일을 깨끗하게 유지

## 연결 정보

- Project ref: `qwrjebpsjjdxhhhllqcw`
- 앱 레포: `/mnt/c/Users/Administrator/programming/apps/gns-hermit-comm`
- 웹 레포: `/home/gunny/apps/web-hermit-comm`

## 앱/웹 기술 스택

| 구분 | 앱 | 웹 |
|---|---|---|
| 프레임워크 | React Native (Expo SDK 55) | Next.js |
| 빌드 | EAS Build (GitHub App 자동 트리거) | Vercel |
| 모니터링 | @sentry/react-native ~7.11.0 | @sentry/nextjs |
| 스타일링 | NativeWind (Tailwind) | Tailwind CSS |
| 상태관리 | TanStack Query | TanStack Query |

## 상세 문서

- [DB 스키마 상세](docs/SCHEMA.md) — 테이블/뷰/함수/RLS/트리거/제약조건/Edge Functions 전체
- [스크립트 사용법](docs/SCRIPTS.md) — db.sh, gen-types.sh, sync-to-projects.sh, verify.sh 상세
- [클라이언트 아키텍처](docs/CLIENT-ARCHITECTURE.md) — 앱/웹 심리분석 흐름, 공유 코드 관리, Realtime 패턴
- [연동 서비스 및 비용](docs/SERVICES.md) — 전체 서비스 플랜/한도/토큰/비용 관리/대응 계획
- [감정분석 업그레이드 설계](docs/archive/DESIGN-emotion-upgrade.md) — E1-E3 단계별 업그레이드 (E1 완료, E2-E3 미착수)
- [구현 완료 아카이브](docs/archive/) — 주제별 이력 6개 (유지보수, 검색, 감정분석, 인프라, 마이페이지+daily, 가이드)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gunnysis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gunnysis)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
