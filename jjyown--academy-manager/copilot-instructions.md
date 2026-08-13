## academy-manager

> > 글로벌 룰(`~/.claude/CLAUDE.md`)에 더해 이 프로젝트 한정 규칙. **이 파일이 항상 우선.**

# Academy Manager — 매뉴얼

> 글로벌 룰(`~/.claude/CLAUDE.md`)에 더해 이 프로젝트 한정 규칙. **이 파일이 항상 우선.**

## 1. 정체성 — 뭘 만드는 프로젝트인지
학원 운영 통합 시스템. 출석/일정/수납/학생/숙제/채점·해설 자동화를 한 코드베이스에서 처리.

- **클라이언트**: 정적 사이트 (HTML/CSS/Vanilla JS) — 운영자/선생님·학부모(`parent-portal/`)·학생 숙제 제출(`homework/`)·자동채점(`grading/`)
- **백엔드**: FastAPI ([grading-server/](grading-server/)) — Gemini Vision + Google Drive + Supabase 연동, 시험지 해설 자동 제작
- **DB/인프라**: Supabase (Auth + Postgres + Edge Functions + Storage)
- **배포**: 프론트는 Vercel(`highroad-math`), 채점 서버는 Docker(로컬/클라우드 동일 이미지)

## 2. 도구 — 어떤 스택·외부 서비스
- Node `20.x` (정적 사이트, 빌드 단계 없음)
- Python `>=3.11` (`grading-server/requirements.txt` 참고: FastAPI 0.115, supabase 2.7, google-generativeai 0.8, pdfplumber, PyMuPDF)
- Supabase 프로젝트 ref: `jzcrpdeomjmytfekcgqu` (운영)
- Google Drive API (숙제 제출/해설 인덱스 저장)
- MCP: Supabase (read-only) — 위험 명령(`apply_migration` 등)은 글로벌 deny

## 3. 검증 방법 — **가장 중요**. 변경 후 반드시 해당 항목 실행

### 프론트엔드 (정적 사이트)
- 로컬 띄우기: `python -m http.server 8000` 후 브라우저에서 `http://localhost:8000`
- 변경한 화면을 실제 브라우저로 클릭해 확인. 콘솔 에러 0개 확인.
- Vanilla JS이므로 타입체크 없음 — `node -c <file>.js`로 구문 체크 가능

### 백엔드 (FastAPI / `grading-server/`)
- 의존성: `cd grading-server && pip install -r requirements.txt`
- 로컬 실행: `cd grading-server && uvicorn main:app --reload --port 8000`
- 변경 모듈 import 검증: `cd grading-server && python -c "import main"` (최소한)
- 헬스 체크: `curl http://localhost:8000/` 또는 라우터별 엔드포인트
- Docker 전체 검증: `docker compose up --build` (루트에서)

### Supabase 마이그레이션
- **MCP `apply_migration` 절대 사용 금지** (deny 처리됨). SQL Editor 직접 실행 패턴.
- 신규 파일은 `migrations/NNNN_*_YYYYMMDD.sql`, BEGIN/COMMIT으로 감싸기, 하단에 검증 SELECT 포함.
- 적용 전 SQL Editor 상단에서 `select current_database(), inet_server_addr()`로 운영 프로젝트(`jzcrpdeomjmytfekcgqu`) 연결 확인.
- 작성 후 사용자에게 "SQL Editor에 붙여넣고 Run" 안내, 결과 확인 후 다음 단계.

## 4. DO
- 변경 후 위 검증 명령어 **반드시 실행**. 못 돌리면 못 돌렸다고 명시.
- 새 일 시작 시 `/memory`로 관련 기억 확인 후 진행.
- 시크릿은 `.env.local` / Supabase Secrets / Vercel 환경변수에. 코드/커밋/로그에 절대 X.
- 마이그레이션은 트랜잭션 + 적용 후 검증 SELECT 포함.
- 커밋 메시지는 한국어 + Conventional Commits (`feat(grading): ...`, `fix(rls): ...`).
- **자동 토의 + 자율 협업** — 다음 신호 감지 시 도메인별 토의 파일에 시간순 회의록 누적하며 자동 토의 진행: 새 기능 추가, DB 스키마/마이그레이션, 인증·RLS 변경, LLM/OCR 호출 추가 또는 fallback, 외부 API 신규 호출, 자동화 toggle, 영향 파일 3개 이상. **도메인별 분리 (2026-05-18 결정)** — `docs/DISCUSSIONS.md`(인덱스) + `docs/DISCUSSIONS_homework.md`(숙제관리) + `docs/DISCUSSIONS_haeseol.md`(해설제작지 매니저 시점) + `docs/DISCUSSIONS_persona.md`(페르소나·시스템 메타). 신규 토의 진입 시 도메인 판별 → 해당 파일 상단 누적, 다른 도메인 누적 금지. **총 20개 페르소나** — 이 프로젝트에서 18개 호출 가능 (글로벌 13 + 매니저 전용 5). 글로벌 13 = chief-reviewer / cost-monitor / curriculum-designer / education-expert / problem-author / product-manager / school-math-teacher / security-reviewer / solution-writer / student-tester / textbook-designer / ui-ux-designer / **ai-ml-engineer (2026-05-19 신설, LLM·OCR·프롬프트 전담)**. 매니저 전용 5 = `academy-developer`(메타 라우터) + `academy-frontend-developer` + `academy-backend-developer` + `academy-supabase-developer` + `academy-reviewer`. 해설 전용 2 = `haeseol-developer`/`haeseol-reviewer` (해설 작업창에서만). 자기들끼리 토의, 사용자는 의뢰인 입장. **각 페르소나는 다른 의견 무조건 수용 금지** — 본인 도메인 관점에서 독립 판단, 결론은 "할만하다/조건부/불가" 명시, 불가 시 대안 제시 필수. 자세한 흐름은 [docs/CLAUDE_USAGE_GUIDE.md §10-3](docs/CLAUDE_USAGE_GUIDE.md).
- **작업 시작 전 페르소나 매핑 점검 의무 (2026-05-19 신설)** — 의뢰인 신규 요청 진입 시 즉시 Edit/Write/Bash 금지. 먼저 ① 요청 도메인 판별 (frontend / backend / supabase / haeseol / LLM·AI / 콘텐츠검토 / 보안 / 비용 / 페르소나·메타 / 기타) ② 호출 후보 페르소나 + skip 페르소나 표 1회 보고 (호출 / skip 사유 1줄 / 미해당) ③ 자율 토의 트리거 여부 판별. **생략 가능 조건**: 단순 질문·읽기 작업·1~2줄 fix·이미 도메인 잠긴 세션(`feedback_chat_session_domain_declaration`)의 후속 작업·의뢰인이 페르소나를 직접 지정한 경우. 영역↔페르소나 매핑 표는 memory `feedback_persona_mapping_before_work` 참고. 비전공자 의뢰인이 "어떤 관점에서 검토받는지" 한 번에 확인 가능하게.
- **자율 학습 누적** — 페르소나가 작업/토의 중 발견한 룰(꼭 해야 함 / 안 됨 / 패턴)을 해당 페르소나 마크다운 `## 학습 노트` 섹션에 자동 누적. 의뢰인 명령("이 룰 [페르소나]에 학습시켜줘")으로도 추가. 자세한 흐름은 [docs/CLAUDE_USAGE_GUIDE.md §10-4](docs/CLAUDE_USAGE_GUIDE.md).
- **현 상태 안주 금지 — 대안 적극 제시** — 토의 시 페르소나들은 현재 스택(Supabase / Vercel / Railway / Google Drive / Gemini)에 갇히지 말 것. 더 나은 도구·서비스·아키텍처(AWS, Cloudflare, OpenAI, Claude API 등)가 본 도메인에 명확히 유리한 경우 **과감히 제시**. 형식: "현재 [도구 A] vs 대안 [도구 B] / B가 나은 점 / 마이그레이션 비용 / 권장 여부". 단 비전공자 1인 운영 부담을 항상 고려 — 새 도구는 "지금 당장은 미루자" 결론도 가능.
- **git 히스토리 참고 필수** — 모든 페르소나는 작업·토의 시 `git log` / `git diff` / `git blame` / 최근 commit message를 자체 조회. 코드 현재 상태만 보고 추측 금지 — **항상 변경 맥락 확인**. 특히 검토자(academy-reviewer / haeseol-reviewer / security-reviewer / cost-monitor)는 최근 30일 commit 패턴 점검 후 의견 작성. 회귀 사고 이력(2026-05-09 RLS / 비용 spike)은 git log로 재현 가능.
- **마무리까지 체크 (현업 라이프사이클)** — 작업은 `git commit`/`git push`로 끝 X. **다음까지 완료해야 종결**: ① 검증 명령 실행 결과 명시 ② 배포 후 즉시 모니터링 (Vercel deploy log / Railway log / Supabase advisor) ③ 학습 노트 추가 (페르소나 마크다운 `## 학습 노트`) ④ 해당 도메인 토의 파일(`docs/DISCUSSIONS_<domain>.md`) 후속 변경 섹션 채우기 + 토의 상태 [종결]로 변경 + `docs/DISCUSSIONS.md` 인덱스 색인 갱신. ⑤ 회귀 발견 시 즉시 토의 재개 또는 롤백.
- **LLM/OCR 호출·자동화 toggle 도입 시 사전 차단 3종 + catch handler 표준** — Gemini/외부 LLM·OCR 호출 추가, fallback 변경, 자동화 toggle 도입 시 다음 모두 충족: ① 비활성 toggle (기본 OFF) ② **일일 비용 hard cap** (GCP API Quota override 또는 Supabase `cost_counter` 진입 가드 — 사후 알림만으론 부족, 2026-05-09 5분 spike 사례) ③ in-flight 가드(409 패턴) + 실패 모니터링. catch handler verbosity+sanitize(memory `feedback_catch_handler_verbosity_sanitize`) 동시 적용. 신규 LLM PR 체크리스트 전체는 [docs/CLAUDE_USAGE_GUIDE.md §10-5](docs/CLAUDE_USAGE_GUIDE.md).

## 5. DON'T
- **RLS `auth.uid()` → `(select auth.uid())` 래핑 금지** (advisor `auth_rls_initplan` 경고는 수용). 본 코드베이스에서 클라이언트가 `current_owner_id`를 잃고 `?owner_user_id=eq.null` 400 에러 대량 발생 회귀 사례 있음(2026-05-09 0028 마이그레이션 롤백).
- Supabase MCP `apply_migration` / `execute_sql`(DDL) / `reset_branch` / `delete_branch` 호출 금지. read-only SELECT만 사용.
- `.mcp.json` 토큰을 새 위치로 옮기지 말 것 — gitignore 처리 여부 먼저 확인 후 사용자에게 보고.
- `index.html` 같은 거대 단일 파일을 임의 분할 금지. 사용자 동의 후 진행.
- 새로운 빌드 도구(webpack, vite 등) 무단 도입 금지. "정적 사이트, 빌드 없음" 패턴 유지.

## 6. 분할 매뉴얼 (필요시 자동 로드)
- 프론트엔드 세부: [.claude/CLAUDE.frontend.md](.claude/CLAUDE.frontend.md)
- 백엔드 세부: [.claude/CLAUDE.backend.md](.claude/CLAUDE.backend.md)
- 학부모 포털: [docs/VERCEL_HIGHROAD_PARENT_PORTAL.md](docs/VERCEL_HIGHROAD_PARENT_PORTAL.md)

## 7. 트리거 단어 (이 단어가 나오면 정해진 동작)
- "**검증해줘**" → 위 §3 검증 절차 전체 실행
- "**마이그레이션 만들어줘**" → `migrations/NNNN_*_YYYYMMDD.sql` 템플릿(트랜잭션+검증 SELECT)으로 작성 후 SQL Editor 안내
- "**해설 제작**" / "**채점**" → `grading-server/` 모듈 우선 탐색
- "**숙제 제출 흐름**" → `homework/` + Edge Function `upload-homework` / `exchange-google-token`
- "**토의해주세요**" → **즉시 plan mode 진입** (Claude가 `EnterPlanMode` 도구 자동 호출) 후 도메인 판별 → 해당 도메인 파일(`docs/DISCUSSIONS_homework.md` / `_haeseol.md` / `_persona.md`) 상단에 시간순 회의록 형식으로 자율 토의 시작 + 인덱스 `docs/DISCUSSIONS.md` 색인 갱신. 사용자가 Shift+Tab 누를 필요 X. 토의 결론 후 `ExitPlanMode`로 사용자 승인 받고 일반 모드 복귀. "토의 좀", "회의해줘", "의견 들어봐" 등 변형 표현도 동일 처리.
- "**[페르소나명]에 학습시켜줘**" → 해당 페르소나 `.claude/agents/<name>.md` 의 `## 학습 노트` 섹션에 룰 추가
- "**여기는 [도메인]만**" → 새 채팅 첫 메시지로 감지 시 본 세션을 해당 도메인 전용으로 잠금 (memory `feedback_chat_session_domain_declaration`). 다른 도메인 작업·검토 요청 들어오면 1줄 확인 후 거절 또는 다른 채팅창 안내.

---
> Source: [jjyown/academy_manager](https://github.com/jjyown/academy_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
