## llm-brain

> Claude는 이 시스템의 **컴파일러**다. `raw/` 소스를 읽어 `wiki/`를 생성·갱신하고, 사용자 질문에 wiki 기반으로 답한다.

# llm-brain — Claude Code 운영 지침

Claude는 이 시스템의 **컴파일러**다. `raw/` 소스를 읽어 `wiki/`를 생성·갱신하고, 사용자 질문에 wiki 기반으로 답한다.

## 가드레일 (절대 위반 금지)

1. `raw/` 출처 없이 `wiki/` 신규 생성·사실 수정 금지
2. query 응답 중 `wiki/` 편집 금지
3. 학습 데이터만으로 `wiki/` 작성 금지 — 반드시 `raw/` 근거 필요
4. `raw/` 파일 수정 금지 — 읽기 전용

### 프라이버시 경계 (Agent Memory OS)

- `episodes/` — 운영 맥락(`user_goal`·`inputs`·`outputs` verbatim)을 담는 append-only 원장. **ingest·express·query·curate 4 경로가 자동 기록**(fail-soft). **gitignored** (스키마 예시 1개만 `examples/`에 커밋) — one-way door 누출 방지.
- `index.md` — wiki 목차(gitignored `wiki/`의 파생물). `business/` 민감 제목 노출 방지로 **gitignored·git 추적 제외**(결정 2026-06-28; 과거 public 커밋 history scrub은 별도 사람 판단).
- `procedures/` — git-tracked이되 **OKF export 제외**(`schema/okf_export.yaml`의 `exclude_paths`).
- `wiki/memory_health_report.md` — okf `META_FILES`에 등재돼 공개 OKF 번들에서 봉인(미포함).

## 명령어

> 플러그인(`commands/`)으로 제공된다. 설치 시 커맨드는 `/llm-brain:ingest`처럼 네임스페이스가 붙는다 — 아래 `/ingest`·`/okf` 등은 `/llm-brain:` 접두로 읽는다. 자연어("ingest 해줘", "okf 해줘")로도 호출된다.

### ingest
```
"ingest 해줘"
"/ingest https://url [--resonance high|medium|low]"
"/ingest ~/path/to/file.pdf [--resonance high]"
"/ingest '텍스트 내용' [--resonance medium]"
```
`scripts/ingest.py` 실행 → `schema/ingest.md` 규칙 적용 → `wiki/` 생성·갱신 → `index.md` 업데이트
> 에피소드 자동기록: raw 저장 직후 `episodes/YYYY-MM.jsonl`에 1줄 append (status `pending_wiki_compilation`, fail-soft — 실패해도 ingest 경로 불간섭).

### curate
```
"curate --distill"     # distill_level 점진 압축
"curate --lifecycle"   # TTL 초과 페이지 archive 후보
"curate --all"         # 전체 실행 (audit + distill + lifecycle)
```
`scripts/curate.py` 실행 → `schema/curate.md` 규칙 적용

### export-graph (wikilink 그래프 export)
```
uv run python scripts/export_graph.py
```
`[[wikilink]]` 파싱 → `wiki/graph.json` 생성 (D3 force-graph 형식).
mini-graph는 `wiki_app` `/api/page/{slug}/graph` 엔드포인트로 조회.

### okf (wiki → OKF v0.1 호환 번들 export)
```
"okf 해줘"                     # /llm-brain:okf — 먼저 dry-run 검토 후 okf/ 번들 생성
"/llm-brain:okf --dry-run"      # export 대상·제외·통계만 (파일 미작성, 보안 검토용)
"/llm-brain:okf --strip-internal"  # 외부 공유본 (x-llmbrain-* 제거)
```
`okf` 커맨드(`commands/okf.md`)가 `scripts/okf_export.py`를 실행해 `wiki/`를
OKF v0.1(Google Open Knowledge Format) 호환 번들 `okf/`로 투영한다 (동료·외부 에이전트·habix
제품이 번역 없이 소비). frontmatter는 OKF 예약 6필드로 매핑, 내부 필드는 `x-llmbrain-*`로 보존,
`[[wikilink]]`는 `/`-절대경로 마크다운 링크로 변환. 변환 규칙: `schema/okf.md`.

**제외/민감 설정 (보안):**
- 경로 제외(`business/**`·`canvas/**`·`episodes/**`·`procedures/**`)는 커밋되는 `schema/okf_export.yaml`의 `exclude_paths`.
- 🔴 **민감 키워드(`sensitive_patterns`)·민감 페이지(`exclude_slugs`)는 gitignored
  `schema/okf_export.local.yaml`에만 둔다** — 커밋되는 yaml에 실명·내부명을 넣으면 그 자체가 누출.

> ⚠ **drift 주의**: `okf/`는 export 시점 스냅샷이다. `wiki/` 갱신 후 재export 안 하면 stale.
> 🔴 **public 커밋 전 보안 게이트 (one-way door)**: `okf/`는 Git 커밋·push되면 history 영구.
> 커밋 전 `--dry-run`으로 ① `business/` 제외 ② `sensitive_hits=0` ③ `excluded` 카운트=기대값을
> 사람이 확인. fresh clone/CI엔 `okf_export.local.yaml`이 없어 게이트가 비활성(stderr 🔴 경고) —
> 그 상태로 커밋 금지.

### query
```
"[질문]에 대해 알려줘"
```
`index.md` 검색 → 관련 `wiki/` 페이지 로드 → wiki 기반 답변
wiki에 없으면: "raw 데이터가 필요합니다" 응답
접근한 페이지 `access_count` 갱신

### express
```
"express blog '[주제]'"
"express lecture '[주제]' --slides N"
"express summary --week"
"express report '[주제]'"
```
`scripts/express.py` 실행 → `express/{type}/YYYY-MM-DD-{slug}.md` 저장
blog: `raw/blog/`에도 복사 (ingest 피드백 루프)
> 에피소드 자동기록: 초안 저장 직후 `episodes/YYYY-MM.jsonl`에 append (status `draft_ready`, fail-soft).

### wiki-web (HTML 검색 페이지)
```
uv run python -m wiki_app
# → http://localhost:8000
```
로컬 HTML 검색·페이지뷰 인터페이스. CLI `/query`의 시각화 버전.

- **검색 알고리즘**: 제목+desc+tags+page_title 점수 매칭 (B). 결과 < 3개 시 본문 grep 자동 확장 (C). 한국어/영문 모두 작동.
- **AI 답변 토글**: `claude -p` CLI 라이브 연결 (SSE 스트리밍 `/api/ai-answer/stream` 포함). CLI 부재 시 `status: unavailable` fallback.
- **백엔드**: `wiki_app/` (FastAPI · uv) — 6 endpoints (`/api/index`, `/api/search`, `/api/page/{slug}`, `/api/page/{slug}/graph`, `/api/ai-answer`, `/api/ai-answer/stream`)
- **프론트엔드**: `wiki_app/static/` (vanilla JS + Pretendard)
- **테스트**: `tests/test_wiki_app_*.py` (5 modules, 73 tests)
- **운영 가드레일**: 페이지뷰 시 wiki frontmatter `access_count` 자동 +1 (CLI query와 동등)
- **에피소드 자동기록**: AI 답변 1건마다 `episodes/YYYY-MM.jsonl`에 append (task_type `ai_answer`, fail-soft — `finally`에서 최종 status 기록, 응답 경로 절대 불간섭)
- **설계 문서**: `docs/superpowers/specs/2026-05-22-wiki-search-html-mvp-design.md`

### brain_context (작업기억 팩 — 턴 직전 컨텍스트 조립)
```
uv run python scripts/brain_context.py --task "..." --topic "..." --type query|express|curate|custom [--max-pages N] [--json]
```
한 작업을 시작하기 전에 흩어진 메모리를 **결정적 순서**의 한 팩으로 모은다 (임베딩 없는 file-first 조립).
6 섹션: ① 목표 ② 관련 semantic 페이지(`index.md` 키워드 점수 + graph degree 동점 정렬) ③ 최근 관련 episode(`episodes/`) ④ 후보 procedure(`procedures/`) ⑤ 제약(CLAUDE.md 가드레일 정적 주입) ⑥ 출처 경로(raw/ provenance).
`--type`은 episode `task_type` 필터를 파생(`query`→`ai_answer`, `curate`→`curate`, `express`/`custom`→topic만). `--json`은 무손실 구조 출력(기본: 마크다운).

### memory_health (읽기전용 메타기억 진단)
```
uv run python scripts/memory_health.py --report
```
`wiki/memory_health_report.md` 생성. wiki(의미)·`episodes/`·`procedures/`를 **읽기만** 해 집계 리포트를 쓴다 — 어떤 wiki 페이지도 이동·삭제·생성하지 않는 **side-effect-free** 진단(curate의 distill/lifecycle/purge와 구분, 유일한 부작용은 리포트 파일 1개 쓰기).
리포트 섹션: memory_type별 페이지 수 · orphan semantic(inbound 0) · stale 절차(>180일 미검증) · 최근 에피소드(집계·메타만, verbatim 본문 비공개) · top 재사용 페이지 · 저신뢰 페이지 · archive 후보.
🔴 episode verbatim 본문은 리포트에 넣지 않으며, 리포트 파일은 okf `META_FILES`에 등재돼 공개 OKF 번들에서 봉인된다.

### doctor (설치 진단·수정)
```
uv run python scripts/doctor.py [--fix]
```
설치 상태 점검 — 필수 디렉토리·스크립트(메모리 OS 포함)·커맨드·설정(config/sources)·의존성·claude CLI. `--fix`는 누락 디렉토리 생성 + `sources.example.yaml`→`sources.yaml` 복사(**기존 파일 미덮어씀**, Rule 9). FAIL=설치 문제, WARN=선택/환경별. exit 1 if FAIL. 새 클론 직후 권장.

### wikiweb (웹 UI 로컬 실행)
```
uv run python -m wiki_app   # → http://localhost:8000
```
wiki-web HTML 검색·페이지뷰 UI를 로컬에서 띄운다(위 `wiki-web` 절과 동일 동작, 슬래시 커맨드 `/llm-brain:wikiweb`). 종료 `Ctrl+C`.

## procedures/ — 재사용 절차 (procedural 메모리)

`procedures/`의 `.md` 파일(각 frontmatter `memory_type: procedural`)은 "어떻게 하는가"를 담는 절차 메모리다. `scripts/procedures.py`가 slug 단위로 로드하고, `brain_context`가 후보 절차로 주입한다.

- git-tracked (공유 워크플로우) + **OKF-excluded** (`exclude_paths`의 `procedures/**` + okf가 `wiki/`만 스캔하는 구조적 이중망).
- 현재 4개 예시: `ingest.md` · `curate.md` · `express-blog.md` · `okf-export-safety.md`.

## 파일 명명

- wiki 페이지: `소문자-하이픈.md` (한국어 개념도 영문 slug)
- 프로젝트: `YYMMDD_project_name/` (언더스코어)
- wikilink: `[[페이지명]]` (확장자 없이)

## wiki frontmatter

```yaml
title: ...
type: concept|tool|person|project|business|lecture|insight
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [raw/파일경로]
distill_level: 0     # 0=원문 1=요약 2=핵심 3=한줄
access_count: 0
# --- Agent Memory OS (US-003) — 아래 6필드 전부 optional·null-safe (없어도 기존 페이지 유효) ---
memory_type: semantic    # (선택) semantic | episodic | procedural | meta | working
retention: durable       # (선택) durable | seasonal | ephemeral (decay 힌트)
confidence: 0.9          # (선택) 0..1 float
source_count: 6          # (선택) len(sources) 캐시
last_verified: YYYY-MM-DD # (선택) 최종 검증일
decay_policy: default    # (선택) 명명된 정책 키
# --- Team-Ready 훅 (v0.3.2 WS-6) — 아래 2필드도 optional·null-safe ---
owner: 이름              # (선택) 페이지 소유자/기여자 태그. 다중 기여자 병합은 P2 예약
scope: shared            # (선택) private | shared. private=okf 공개 번들 제외 / 미지정=shared(하위호환)
```

> `memory_health`가 위 optional 필드(`memory_type`·`confidence`·`last_verified` 등)로 집계·진단한다.
> `scope: private` 페이지는 `okf` export 시 공개 번들에서 항상 제외된다(`--strip-internal`은 `owner`/`scope`도 제거).
> 상세 스펙: `SPEC.md` / 사용 가이드: `README.md`

---
> Source: [kimsanguine/llm-brain](https://github.com/kimsanguine/llm-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
