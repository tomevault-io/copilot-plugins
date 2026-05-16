## tokensave

> > **단일 진실 원천(SSOT).** 이 파일은 tokensave 하네스 5인 팀의 모든 산출물 기준점이다. 5인 팀이 동시에 작업하므로, 의문이 생기면 이 문서를 우선 참조한다.

# tokensave — Claude Code/하네스 토큰 절감 스킬 (메타 하네스)

> **단일 진실 원천(SSOT).** 이 파일은 tokensave 하네스 5인 팀의 모든 산출물 기준점이다. 5인 팀이 동시에 작업하므로, 의문이 생기면 이 문서를 우선 참조한다.

## 목표

Claude Code + 하네스 운영에서 발생하는 **토큰 낭비 패턴을 외부 검색·실증·기존 하네스 감사 3축으로 분류**하고, 분류 결과를 (1) 작업 시작 전 사전 점검(Pre-flight), (2) 신규 스킬·에이전트 설계 가이드(Authoring), (3) 기존 하네스 전수 감사(Audit), (4) 런타임 자동 가드(Hook) 4가지 모드에서 활용 가능한 `tokensave` Claude Code 스킬로 패키징한다.

**메타 원칙 (제일 중요).** 우리가 이 스킬을 만드는 과정 자체가 가장 큰 토큰 낭비 사례가 되어선 안 된다. 본 하네스 자체에 본 스킬이 권장하는 패턴(불필요한 opus 회피, 결정적 작업은 코드, 병렬 활용, 컨텍스트 분리)을 적용한다.

## 4대 철칙

1. **검색-기반 분류, 카탈로그 의존 금지.** 외부 1차 출처(Anthropic 공식·X·블로그·GitHub·논문)에서 패턴을 수집해 분류학을 구성한다. `harness-diagnostic` 21개 갭 카탈로그는 *참고용*일 뿐이며, 그 안에서도 **HD-003(결정적 작업의 LLM 라우팅)·HD-010(report-first 컨벤션의 산출물 유실)·HD-011(writer agent의 per-call 한도 부재) 셋만 검증된 유효 패턴**으로 채택한다. 나머지 18개는 표준 reliability를 harness 패턴인 척 포장한 면이 있어 자동 import 금지 — 외부 검색에서 동일 패턴이 독립적으로 ≥ 2회 확증돼야 분류학에 등록한다.
2. **결정적 작업은 Python, LLM은 판단에만.** HD-003 패턴은 이 스킬의 1순위 적용 대상. 스크립트는 결정적 정적 분석, LLM은 패턴 해석·우선순위·재작성 권고에만 사용한다.
3. **모델 티어 분기.** 본 하네스의 5인 에이전트도 Haiku/Sonnet/Opus를 작업 성격에 맞게 배분한다. 리서치 스카우트 3명(sonnet) · 분류학자(opus 판단 필요) · 스킬 작성(opus) · QA(sonnet).
4. **활성화 톤은 단호하게.** SKILL.md의 트리거·필수 단계·금지 패턴은 사용자 참고 X 프롬프트처럼 mandatory·zero-tolerance 톤으로 작성. 단, 내용은 데이터·출처 기반.

## 5인 에이전트 팀

| # | 에이전트 | 모델 | 역할 | 산출물 |
|---|---------|------|------|--------|
| 1 | `token-research-scout-anthropic` | sonnet | Anthropic 공식(docs.anthropic.com/claude-code, anthropic 엔지니어링 블로그, Claude API docs, cookbook)에서 prompt caching·model selection·context management·agent SDK·subagent·hooks 영역 best practice 수집 | `_workspace/01_research/anthropic.md` + sources.json |
| 2 | `token-research-scout-community` | sonnet | X·Reddit·HN·블로그·GitHub repo(awesome-claude-code, claude-code-* 류)에서 실전 토큰 절감 후기·anti-pattern·트릭 수집. 가능한 한 정량 수치(절감률·비용 추정) 포함 | `_workspace/01_research/community.md` + sources.json |
| 3 | `token-research-scout-empirical` | sonnet | arXiv·Augment Code·Fowler·OpenAI harness 글·`harness-diagnostic` v0.2 카탈로그·`paper-maker` diagnostic·`ai-tell-taxonomy` 같은 기존 분류학 분석 + 기존 ~/.claude/agents 104개·skills 32개·CLAUDE.md 33개 하네스 정적 baseline 측정 | `_workspace/01_research/empirical.md` + `_workspace/02_audit/baseline.md` |
| 4 | `token-pattern-taxonomist` | opus | 3개 스카우트 결과를 통합해 **N대 카테고리 × M개 서브패턴(심각도 S1/S2/S3) 분류학** 작성. 각 패턴: 트리거 keyword·예시·검출 방법·완화 방법·외부 출처 ≥ 2건 | `_workspace/03_patterns/taxonomy.md` |
| 5 | `tokensave-skill-builder` | opus | 분류학을 받아 SKILL.md(강한 활성화 룰·4가지 모드)·scripts/audit.py·scripts/estimate_cost.py·**scripts/model_selector.py**(★ headline)·hooks 예제·**references/task_to_model_matrix.md**(★ headline)·기타 references/ 모두 작성 | `_workspace/05_skill/SKILL.md`, `scripts/*.py`, `references/*.md` |

**★ Headline 산출물 (사용자가 즉시 가치 느끼는 두 가지)**

1. **`references/task_to_model_matrix.md`** — 작업 성격(예: 결정적 변환·정형 요약·자유 생성·창의 reasoning·코드 리뷰·팩트체크·번역·다국어·코딩·아키텍처 결정 등 N가지) × 추천 모델(Haiku/Sonnet/Opus) × 근거(가격·품질 벤치마크·실측) 매트릭스. 분류학자가 N개 작업 유형을 정의하고, 빌더가 Anthropic 공식 가격·품질 데이터(Scout A)와 커뮤니티 실측(Scout B)·홈 카탈로그 분포(Scout C)로 채운다.
2. **`scripts/model_selector.py`** — 작업 설명(자연어) + 입력 토큰 추정 + 품질 요구 수준을 받아 추천 모델·근거·예상 비용을 출력하는 CLI 도구. 결정 트리는 LLM 호출 0회(완전 결정적). 사용 예: `python model_selector.py --task "PDF에서 표 추출하고 CSV로 정규화" --tokens 12000 --quality medium`.

QA는 별도 에이전트 대신 오케스트레이터가 직접 — 활성화 룰 더블체크, 스크립트 실행 검증, 홈 카탈로그 baseline 감사 1회를 직접 돌린다. 분리된 QA 에이전트는 메타 원칙 위반(작은 검증 작업에 별도 에이전트 = 토큰 낭비).

## 파이프라인 (5단계)

```
Phase 1 (Research, 백그라운드 병렬)
  scout-anthropic ──┐
  scout-community ──┼──> _workspace/01_research/*.md
  scout-empirical ──┘     _workspace/02_audit/baseline.md
                          ↓
Phase 2 (Taxonomy, 단일)
  pattern-taxonomist ──> _workspace/03_patterns/taxonomy.md
                          ↓
Phase 3 (Skill Build, 단일)
  tokensave-skill-builder ──> SKILL.md + scripts/* + references/*
                          ↓
Phase 4 (QA, 오케스트레이터 직접)
  - SKILL.md 트리거 정확도 자체 점검
  - scripts 실행 검증
  - 홈 카탈로그 (~/.claude/agents 104개 + skills 32개 + CLAUDE.md 33개 하네스) 1회 전수 감사
  ──> _workspace/07_qa/audit_baseline.md
                          ↓
Phase 5 (Deploy)
  - ~/.claude/skills/tokensave/SKILL.md + scripts 복사 (사용자 승인 후)
```

병렬화는 Phase 1만. 이후는 모두 순차 — 다음 단계가 이전 결과에 의존하므로 병렬 의미 없음.

## 산출물 트리

```
<your-project-dir>/tokensave/
├── CLAUDE.md                       (this SSOT)
├── _reference/
│   └── harness-diagnostic/         (참고 자료 — 입력 1건일 뿐)
├── _workspace/
│   ├── 01_research/
│   │   ├── anthropic.md           (scout 1)
│   │   ├── community.md           (scout 2)
│   │   └── empirical.md           (scout 3)
│   ├── 02_audit/
│   │   └── baseline.md            (scout 3 부산물)
│   ├── 03_patterns/
│   │   └── taxonomy.md            (taxonomist)
│   ├── 05_skill/
│   │   └── SKILL.md               (builder, 사용자 승인 전 staging)
│   ├── 06_scripts/                (builder)
│   └── 07_qa/
│       └── audit_baseline.md      (오케스트레이터)
├── scripts/                        (최종 — _workspace/06에서 승격)
│   ├── audit.py
│   ├── estimate_cost.py
│   ├── model_selector.py
│   └── hook_check.py
├── references/                     (최종)
│   ├── model_tier_matrix.md
│   ├── python_vs_llm_tree.md
│   ├── prompt_caching_checklist.md
│   └── parallel_patterns.md
└── SKILL.md                        (최종 — _workspace/05에서 승격, 동시에 ~/.claude/skills/tokensave/ 에 복사 권고)
```

## 활성화 트리거 (사용자 측)

- "토큰 절감 스킬", "tokensave", "이 작업 모델 뭐가 적절해?", "이거 LLM 안 쓰고 파이썬으로 되나?", "내 하네스 감사해줘", "오퍼스 남용 점검", "프롬프트 캐싱 어떻게?", "에이전트 병렬 vs 순차?", "스킬 만들 때 토큰 절약 가이드"

후속 작업("패턴 추가", "감사 다시", "모델 재추천", "스킬 활성화 강화")도 동일 하네스로 처리.

## 외부 입력 자원

- `_reference/harness-diagnostic/` (사용자 본인 저장소 — paper-maker 21갭 카탈로그·CLI·NL prompt)
- Anthropic 공식: `docs.anthropic.com/en/docs/claude-code/*`, `docs.anthropic.com/en/docs/build-with-claude/prompt-caching`, `anthropic.com/engineering/*`
- 커뮤니티: X 검색 (`claude code token`, `claude code cost`, `sonnet vs opus`), Reddit r/ClaudeAI, HN
- 기존 ~/.claude/{agents,skills,CLAUDE.md} — 베이스라인 측정 대상

## 메타 원칙 셀프 체크 (이 하네스에 적용)

- [ ] 5인 팀 중 sonnet 3 + opus 2 — 불필요 opus 사용 0
- [ ] Phase 1은 백그라운드 병렬 — 3개 스카우트 동시 실행
- [ ] 결정적 작업(파일 카운트·grep·정적 분석)은 모두 Python — taxonomist·builder의 LLM 토큰을 판단에만 사용
- [ ] 별도 QA 에이전트 없음 — 오케스트레이터 직접 (검증은 deterministic)
- [ ] _reference/는 한 번 읽고 요약 메모만 남김 — 매번 재읽기 금지

## 현재 버전

v1 (2026-05-14, 초기 구성).

---
> Source: [epoko77-ai/tokensave](https://github.com/epoko77-ai/tokensave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
