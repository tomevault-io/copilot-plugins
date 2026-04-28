## zeroalign-rec

> 이 파일은 LLM이 이 프로젝트의 지식 저장소를 체계적으로 관리하기 위한 스키마 레이어다.

# AGENTS.md

이 파일은 LLM이 이 프로젝트의 지식 저장소를 체계적으로 관리하기 위한 스키마 레이어다.
사용자와 LLM이 시간이 지남에 따라 함께 진화시킨다.

---

## 프로젝트 개요

SID(Semantic ID) 기반 training-free 추천 시스템을 로컬 환경에서 실험하기 위한 Python 코드베이스다.

- **로컬 추론**: Apple Silicon MLX (`mlx-lm`, `mlx-embeddings`)
- **생성 모델**: `mlx-community/Qwen3.5-9B-OptiQ-4bit`
- **임베딩 모델**: `mlx-community/Qwen3-Embedding-4B-4bit-DWQ`
- **패키지 관리**: `uv`
- **CLI**: `typer` + `rich`

### 주요 모듈

| 모듈 | 경로 | 역할 |
|------|------|------|
| config | `src/sid_reco/config.py` | 환경 변수 + `.env` 기반 설정, 경로 해석 |
| llm | `src/sid_reco/llm.py` | `MLXTextGenerator` — 로컬 생성 추론 |
| embedding | `src/sid_reco/embedding.py` | `MLXEmbeddingEncoder` — 로컬 임베딩 추론 |
| mlx_runtime | `src/sid_reco/mlx_runtime.py` | MLX 런타임 probe 및 지원 환경 진단 |
| datasets | `src/sid_reco/datasets/` | 데이터셋 로더 및 전처리 (`foodcom.py`) |
| taxonomy | `src/sid_reco/taxonomy/` | neighbor context 임베딩/FAISS 검색, taxonomy dictionary 생성, item-level taxonomy structuring (`item_projection.py`) |
| sid | `src/sid_reco/sid/` | Phase 1 마지막 단계용 structured item serialization, MLX embedding, CPU residual K-means compiler, FAISS indexing, recommendation statistics 생성 (`serialization.py`, `embed_backend.py`, `compiler.py`, `indexing.py`, `stats.py`) |
| recommendation | `src/sid_reco/recommendation/` | Phase 2 recommendation pipeline: interest sketch, semantic retrieval, bootstrap rerank, confidence aggregation, grounding |
| cli | `src/sid_reco/cli.py` | `doctor`, `smoke-mlx`, `smoke-llm`, `smoke-embed`, `prepare-foodcom`, `build-neighbor-context`, `build-taxonomy-dictionary`, `structure-taxonomy-item`, `structure-taxonomy-batch`, `compile-sid-index`, `recommend` |

### 빌드 및 검증 명령

```bash
uv sync --all-groups          # 의존성 설치
uv run pytest                 # 테스트
uv run ruff check .           # 린트
uv run mypy src               # 타입체크
uv run sid-reco doctor        # 환경 상태 확인
uv run sid-reco smoke-mlx     # MLX 런타임 진단
uv run sid-reco recommend --help
uv run sid-reco build-neighbor-context --help
uv run sid-reco build-taxonomy-dictionary --help
uv run sid-reco structure-taxonomy-item --help
uv run sid-reco structure-taxonomy-batch --help
uv run sid-reco compile-sid-index --help
```

### Repo-local Codex Commands

이 저장소의 repo-local Codex 확장점은 `.agents/skills/`다.
공식 Codex App built-in slash command와는 별개로, enabled skill은 slash 목록에 나타날 수 있다.

- `/docs-manager` 또는 `/doc-manager` — 위키/ADR/인덱스 작업과 `README.md`, `AGENTS.md`, `.github/copilot-instructions.md`, `.harness/reference/local-adaptation.md` 동기화를 포함한 문서 반영 루틴
- `/spec` — 구현 전에 `SPEC.md`를 정리하는 spec-driven workflow
- `/plan` — `tasks/plan.md`, `tasks/todo.md` 기반 작업 분해
- `/build` — incremental-implementation + TDD 기반 구현 흐름
- `/test` — TDD / Prove-It 기반 검증 흐름
- built-in `/review` 또는 `$code-review-and-quality` — 5축 코드 리뷰
- `/code-simplify` — 동작 보존 단순화 검토
- `/ship` — 배포/릴리스 전 체크리스트

---

## 개발 워크플로 스킬 레이어

이 저장소에는 범용 엔지니어링 워크플로 스킬 레이어도 함께 존재한다.

- 활성 스킬 위치: `.agents/skills/`
- 하네스 support 자산 루트: `.harness/`
- 체크리스트 참조: `references/`
- upstream 문서 스냅샷: `.harness/reference/`
- 훅 스냅샷: `.harness/hooks/`
- archived command prompt 초안: `.harness/reference/command-drafts/`
- archived persona markdown: `.harness/reference/agent-personas/`

운영 원칙:

1. `docs/sources/`, `docs/wiki/`, ADR/교차 참조/인덱스 작업은 항상 `docs-manager`와 이 `AGENTS.md` 스키마가 우선한다.
2. 코드 구현, 테스트, 리뷰, 릴리스 흐름은 imported agent skills를 사용할 수 있다.
3. imported skill의 일반 예시가 이 저장소 구조와 충돌하면 `.harness/reference/local-adaptation.md` 규칙을 우선한다.
4. 위키 문서는 계속 한국어로 유지한다.

---

## 3-레이어 아키텍처

이 프로젝트의 지식 저장소는 3개 레이어로 구성된다.

```
.
├── AGENTS.md                          ← 스키마 레이어 (이 파일)
└── docs/
    ├── sources/                       ← 원문 소스 레이어 (불변)
    │   ├── papers/                    SID·추천 관련 논문
    │   ├── datasets/                  데이터셋 원본 문서·스키마
    │   ├── models/                    모델 공식 문서·벤치마크
    │   └── experiments/               실험 로그·결과 데이터
    └── wiki/                          ← 위키 레이어 (LLM 소유)
        ├── INDEX.md                   전체 페이지 목록
        ├── summaries/                 소스별 요약
        ├── entities/                  구체적 대상 (모델, 데이터셋 등)
        ├── concepts/                  추상 개념 (SID, Training-Free 등)
        ├── comparisons/               비교·분석
        ├── overviews/                 주제별 합성 개요
        ├── decisions/                 ADR (Architecture Decision Record)
        └── logs/                      인제스트 건별 로그
```

### 레이어 1: 원문 소스 (`docs/sources/`)

**진실의 원천(Source of Truth)**이다.

- 변경 불가(immutable) — LLM은 읽기만 하고 수정하지 않는다
- 사용자가 큐레이션한 소스 문서를 저장한다: 논문, 데이터셋 문서, 모델 자료, 실험 로그
- 하위 카테고리:
  - `papers/` — 학술 논문 PDF 또는 텍스트 요약
  - `datasets/` — 데이터셋 스키마, README, 출처 문서
  - `models/` — 모델 카드, 벤치마크 발췌, 공식 문서
  - `experiments/` — 실험 설정, 결과 데이터, 로그 (기록 후 불변)

### 레이어 2: 위키 (`docs/wiki/`)

**LLM이 완전히 소유**하는 마크다운 파일 디렉터리다.

- LLM이 페이지 생성, 업데이트, 교차 참조 유지를 담당한다
- 사용자는 읽기만 한다
- 모든 페이지에 YAML 프론트매터가 필수다

### 레이어 3: 스키마 (이 파일)

LLM에게 위키 구조, 컨벤션, 워크플로를 알려주는 설정 문서다.
사용자와 LLM이 시간이 지남에 따라 함께 진화시킨다.

---

## 위키 페이지 타입

### 1. 요약 (Summary)

소스 1개를 읽고 핵심 내용을 정리한 페이지다.

**프론트매터:**

```yaml
---
title: "<소스 제목 또는 핵심 주제>"
date: YYYY-MM-DD
type: summary
sources:
  - docs/sources/<category>/<filename>
---
```

**본문 구조:**

```markdown
# <제목>

## 출처 정보
- 저자, 발행일, 출처 유형 등

## 핵심 주장/발견
- 소스의 주요 논점, 결과, 발견 사항

## 방법론
- 사용된 접근법, 실험 설계 (해당되는 경우)

## 한계
- 저자가 밝힌 한계 또는 분석 상 한계

## Related
- 관련 위키 페이지 링크
```

### 2. 엔티티 (Entity)

모델, 데이터셋, 라이브러리, 도구 등 구체적인 대상을 설명하는 페이지다.

**프론트매터:**

```yaml
---
title: "<대상 이름>"
date: YYYY-MM-DD
type: entity
tags: [<tag1>, <tag2>]
sources:
  - docs/sources/<category>/<filename>
---
```

**본문 구조:**

```markdown
# <대상 이름>

## 개요
- 무엇인지, 왜 중요한지

## 현재 상태
- 이 프로젝트에서 어떻게 사용되고 있는지

## 사용법/설정
- 구체적인 설정, 명령, 코드 예시

## Related
- 관련 위키 페이지 링크
```

### 3. 개념 (Concept)

SID, Training-Free Recommendation 등 추상적인 개념을 설명하는 페이지다.

**프론트매터:**

```yaml
---
title: "<개념 이름>"
date: YYYY-MM-DD
type: concept
tags: [<tag1>, <tag2>]
sources:
  - docs/sources/<category>/<filename>
---
```

**본문 구조:**

```markdown
# <개념 이름>

## 정의
- 한 문단으로 정의

## 배경/동기
- 왜 이 개념이 필요한지

## 핵심 메커니즘
- 어떻게 작동하는지

## 이 프로젝트에서의 역할
- 이 프로젝트에서 어떻게 활용되는지

## Related
- 관련 위키 페이지 링크
```

### 4. 비교 (Comparison)

두 개 이상의 대상이나 접근법을 비교·분석한 페이지다.

**프론트매터:**

```yaml
---
title: "<A> vs <B> (vs <C>)"
date: YYYY-MM-DD
type: comparison
subjects: [<A>, <B>]
sources:
  - docs/sources/<category>/<filename>
---
```

**본문 구조:**

```markdown
# <A> vs <B>

## 비교 기준
- 어떤 축으로 비교하는지

## 비교 표

| 기준 | A | B |
|------|---|---|
| ... | ... | ... |

## 분석
- 표에서 드러나지 않는 맥락, 트레이드오프

## 결론/권장
- 이 프로젝트 맥락에서의 권장 사항

## Related
- 관련 위키 페이지 링크
```

### 5. 개요/합성 (Overview)

여러 소스를 종합하여 주제 전체를 조망하는 합성 페이지다.

**프론트매터:**

```yaml
---
title: "<주제> 개요"
date: YYYY-MM-DD
type: overview
sources:
  - docs/sources/<category>/<filename1>
  - docs/sources/<category>/<filename2>
---
```

**본문 구조:**

```markdown
# <주제> 개요

## 범위
- 이 개요가 다루는 범위

## 주제 요약
- 소스들에서 추출한 핵심 내용 종합

## 핵심 인사이트
- 개별 소스에서는 보이지 않지만 종합하면 드러나는 인사이트

## 미해결 질문
- 아직 답이 없거나 추가 소스가 필요한 질문

## Related
- 관련 위키 페이지 링크
```

### 6. 결정 기록 (ADR)

Architecture Decision Record. 프로젝트에서 내린 기술적 의사결정을 기록한다.

**프론트매터:**

```yaml
---
title: "<결정 제목>"
date: YYYY-MM-DD
type: adr
status: Proposed | Accepted | Deprecated | Superseded
supersedes: decisions/<이전 ADR 파일명>  # 선택
sources: []
---
```

**본문 구조:**

```markdown
# ADR-NNN: <결정 제목>

## Context
- 결정이 필요한 배경, 제약조건

## Decision
- 무엇을 결정했는지

## Consequences
### 긍정적
- ...
### 부정적/제약
- ...
### 다음 단계
- ...

## Related
- 관련 위키 페이지 링크
```

---

## 위키 컨벤션

### 파일명

- kebab-case를 사용한다
- 엔티티: 대상명 기반 (`qwen3-embedding-4b.md`, `food-com-dataset.md`)
- ADR: `adr-NNN-<제목>.md` (`adr-001-dev-environment.md`)
- 요약: `<소스 이름 또는 핵심 키워드>.md`
- 비교: `<A>-vs-<B>.md`
- 개요: `<주제>.md`
- 인제스트 로그: `ingest-YYYY-MM-DD-<slug>.md`

### 교차 참조

- 상대 경로 마크다운 링크를 사용한다: `[dev-environment](../entities/dev-environment.md)`
- 모든 페이지의 `## Related` 섹션에 관련 페이지 링크를 포함한다
- 양방향 링크를 유지한다 — A가 B를 참조하면, B도 A를 참조해야 한다

### 소스 참조

- 프론트매터 `sources` 필드에 `docs/sources/` 기준 상대 경로를 명시한다
- 요약 페이지에서 `sources`는 필수다 (최소 1개)
- 엔티티, 개념 등은 소스 없이도 생성 가능하다 (프로젝트 내부 결정 기반)

### LLM 작성 규칙

1. **사실 기반**: 소스에 있는 내용만 작성한다. 추측하지 않는다.
2. **소스 추적**: 주장이나 수치에는 출처 소스를 명시한다.
3. **교차 참조 적극 활용**: 관련 페이지가 있으면 반드시 링크한다.
4. **인덱스 동기화**: 페이지 생성/삭제 시 `INDEX.md` + 해당 카테고리 `README.md`를 동시에 업데이트한다.
5. **프론트매터 필수**: 모든 위키 페이지에 YAML 프론트매터를 포함한다.
6. **한국어 사용**: 문서는 한국어로 작성한다.

---

## Operation 1 — 인제스트 (Ingest)

새 소스를 원문 컬렉션에 추가하고 위키에 반영하는 워크플로다.

### 트리거

사용자가 소스 파일을 `docs/sources/{category}/`에 배치한 후, LLM에게 인제스트를 요청한다.

### 워크플로

```
1. 소스 읽기
   └─ 사용자가 지정한 소스 파일을 읽고 핵심 내용을 파악한다

2. 요약 페이지 생성
   └─ docs/wiki/summaries/<slug>.md 에 Summary 타입 페이지를 작성한다

3. 기존 페이지 영향 분석
   └─ INDEX.md를 참조하여 영향받는 기존 페이지를 식별한다
   └─ 분석 대상: entities, concepts, comparisons, overviews

4. 기존 페이지 업데이트
   └─ 관련 기존 페이지에 새 소스 내용을 반영한다
   └─ 프론트매터 sources 필드에 새 소스 경로를 추가한다

5. 신규 페이지 생성
   └─ 새로운 엔티티나 개념이 발견되면 해당 타입의 페이지를 생성한다

6. 교차 참조 갱신
   └─ 새 페이지 ↔ 기존 페이지 간 양방향 링크를 추가한다

7. 인덱스 업데이트
   └─ docs/wiki/INDEX.md 를 업데이트한다
   └─ 영향받은 카테고리의 README.md 를 업데이트한다

8. 인제스트 로그 작성
   └─ docs/wiki/logs/ingest-YYYY-MM-DD-<slug>.md 를 생성한다
```

### 인제스트 로그 형식

```markdown
---
title: "Ingest: <소스 이름>"
date: YYYY-MM-DD
type: ingest-log
source: docs/sources/<category>/<filename>
---

# Ingest Log: <소스 이름>

## 처리한 소스
- 경로: `docs/sources/<category>/<filename>`
- 유형: 논문/데이터셋 문서/모델 자료/실험 로그

## 생성된 페이지
- [<페이지 제목>](<위키 경로>)

## 수정된 페이지
- [<페이지 제목>](<위키 경로>) — 변경 내용 요약

## 주요 발견
- 소스에서 발견한 핵심 인사이트
```

### 처리 모드

- **단건 처리**: 소스를 하나씩 처리하며 각 단계에서 사용자와 논의
- **일괄 처리**: 여러 소스를 순차적으로 처리하되 감독을 줄임. 완료 후 전체 인제스트 로그 목록 보고

### 참고

- 단일 소스가 10~15개 위키 페이지에 영향을 줄 수 있다
- 기존 페이지를 업데이트할 때는 기존 내용을 보존하면서 새 정보를 추가한다
- 기존 내용과 새 소스가 상충하면 두 관점을 모두 기술하고 차이를 명시한다

---

## Operation 2 — 쿼리 (Query)

위키를 대상으로 질문하면 관련 페이지를 찾아 인용과 함께 답변을 합성하는 워크플로다.

### 트리거

사용자가 프로젝트 지식에 관한 질문을 제시한다.

### 워크플로

```
1. 질문 분석
   └─ 사용자의 질문에서 관련 키워드, 개념, 엔티티를 식별한다

2. 페이지 탐색
   └─ INDEX.md 를 참조하여 관련 위키 페이지를 찾는다
   └─ 관련 페이지들을 읽어 답변에 필요한 정보를 수집한다

3. 답변 합성
   └─ 관련 페이지들의 내용을 인용하며 답변을 구성한다
   └─ 인용 형식: [페이지 제목](위키 경로) 에 따르면 ...

4. 출력 형태 선택
   └─ 질문의 성격에 따라 적절한 형태를 선택한다:
      - 마크다운 텍스트 (기본)
      - 비교 표
      - Marp 슬라이드 덱
      - matplotlib 차트/그래프
      - 구조화된 목록
```

### 답변의 위키 저장

좋은 답변은 위키에 새 페이지로 저장할 수 있다. 탐색 자체가 지식 베이스에 쌓인다.

```
5. (선택) 위키 저장
   └─ 사용자가 답변 품질을 확인하고 저장을 승인한다
   └─ 적절한 카테고리(overview, comparison 등)에 페이지를 생성한다
   └─ INDEX.md + 카테고리 README.md 를 업데이트한다
   └─ 관련 기존 페이지에 교차 참조를 추가한다
```

---

## Operation 3 — 린트 (Lint)

주기적으로 위키 상태를 점검하여 일관성, 완전성, 최신성을 유지하는 워크플로다.

### 트리거

사용자가 LLM에게 위키 린트를 요청한다.

### 점검 체크리스트

| # | 항목 | 설명 | 심각도 |
|---|------|------|--------|
| 1 | 모순 검출 | 페이지 간 상충하는 주장 식별 | HIGH |
| 2 | 낡은 정보 | 최신 소스에 의해 대체된 주장 발견 | HIGH |
| 3 | 고아 페이지 | 인바운드 링크가 없는 페이지 식별 | MEDIUM |
| 4 | 누락 개념 | 여러 페이지에서 참조되지만 자체 페이지가 없는 개념 | MEDIUM |
| 5 | 누락 교차 참조 | 관련 페이지 간 빠진 링크 식별 | LOW |
| 6 | 데이터 공백 | 웹 검색이나 추가 소스로 채울 수 있는 빈 영역 제안 | LOW |

### 워크플로

```
1. 페이지 목록 로드
   └─ INDEX.md 에서 전체 위키 페이지 목록을 가져온다

2. 전수 스캔
   └─ 모든 위키 페이지를 읽는다

3. 체크리스트 점검
   └─ 위 6개 항목을 순서대로 점검한다

4. 린트 보고서 출력
   └─ 발견된 문제를 아래 형식으로 보고한다:

   | 심각도 | 항목 | 해당 페이지 | 설명 | 제안 조치 |
   |--------|------|-------------|------|-----------|
   | HIGH   | 모순 | A.md, B.md  | ...  | ...       |

5. (선택) 자동 수정
   └─ 사용자 승인 후 자동 수정 가능한 항목을 적용한다:
      - 누락 교차 참조 추가
      - INDEX.md 갱신
      - 카테고리 README.md 갱신
      - 고아 페이지에 인바운드 링크 추가
```

### 린트 실행 예시

```
사용자: 위키 린트 실행해줘

LLM:
## 린트 보고서 (2026-04-07)

전체 페이지: 2개
스캔 완료: 2/2

| 심각도 | 항목 | 해당 페이지 | 설명 | 제안 조치 |
|--------|------|-------------|------|-----------|
| MEDIUM | 누락 개념 | entities/dev-environment.md | "SID" 개념이 언급되지만 자체 페이지 없음 | concepts/sid.md 생성 |
| LOW | 데이터 공백 | decisions/adr-001-*.md | MLX 벤치마크 데이터 없음 | sources/models/에 벤치마크 추가 |

자동 수정 가능: 0건
수동 조치 필요: 2건
```

---

## 위키 인덱스 관리

### `docs/wiki/INDEX.md`

전체 위키 페이지 목록이다. LLM은 다음 시점에 이 파일을 업데이트해야 한다:

- 인제스트 오퍼레이션 완료 시
- 쿼리 결과를 위키에 저장할 때
- 린트 후 자동 수정을 적용할 때
- 페이지를 삭제하거나 이동할 때

### 카테고리별 `README.md`

각 카테고리 디렉터리의 `README.md`는 해당 카테고리 내 페이지 목록이다.
`INDEX.md`와 동시에 업데이트한다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pko89403) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
