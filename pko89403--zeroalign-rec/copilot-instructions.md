## zeroalign-rec

> 이 파일은 LLM이 이 프로젝트의 지식 저장소를 체계적으로 관리하기 위한 스키마 레이어다.

# AGENTS.md

이 파일은 LLM이 이 프로젝트의 지식 저장소를 체계적으로 관리하기 위한 스키마 레이어다.
사용자와 LLM이 시간이 지남에 따라 함께 진화시킨다.

---

## 프로젝트 개요

SID(Semantic ID) 기반 training-free 추천 시스템을 로컬 환경에서 실험하기 위한 Python 코드베이스다.

> 기술 스택·주요 모듈·빌드/검증 명령·Repo-local Codex Commands 등 도메인 구체 정보는 [`.agents/policies/local-adaptation.md`](.agents/policies/local-adaptation.md)를 참조한다.

---

## 개발 워크플로 스킬 레이어

이 저장소에는 범용 엔지니어링 워크플로 스킬 레이어도 함께 존재한다.

- 활성 스킬 위치: `.agents/skills/`
- Claude runtime 설정: `.claude/settings.json`
- Codex hook 설정: `.codex/hooks.json`
- 체크리스트 및 정책: `.agents/playbooks/`, `.agents/policies/`
- hook 스크립트: `scripts/hooks/`
- optional phase executor: `scripts/execute.py`

운영 원칙:

1. `raw/`, `graphify-out/`, ADR/설계 노트 작업은 항상 `docs-manager`와 이 `AGENTS.md` 스키마가 우선한다.
2. 코드 구현, 테스트, 리뷰, 릴리스 흐름은 imported agent skills를 사용할 수 있다.
3. imported skill의 일반 예시가 이 저장소 구조와 충돌하면 `.agents/policies/local-adaptation.md` 규칙을 우선한다.
4. Claude Code safety hooks는 `.claude/settings.json`과 `scripts/hooks/` 스크립트로 활성화된다.
5. 사용자가 자연어로 PR 생성을 요청해도 `.github/pull_request_template.md`를 반드시 기준으로 사용한다.
6. `gh pr create --body` 또는 `--body-file`로 템플릿을 우회하지 않는다. `gh`를 쓸 때는 `--template .github/pull_request_template.md` 또는 동등한 템플릿 기반 경로를 사용한다.
7. 사용자에게 의사결정 옵션을 제시할 때는 공수가 크더라도 근본적으로 '제대로된 방향'을 권장안으로 먼저 제안한다. 공수 절충안이나 현상 유지 옵션은 권장안 뒤에 함께 병기한다.

> 언어 규칙은 [`.agents/policies/local-adaptation.md`](.agents/policies/local-adaptation.md)의 "Language Conventions"를, graphify orchestration 세부는 [`.agents/skills/graphify-manager/SKILL.md`](.agents/skills/graphify-manager/SKILL.md)를 참조한다.

---

## 3-레이어 아키텍처

이 프로젝트의 지식 저장소는 3개 레이어로 구성된다.

```
.
├── AGENTS.md                          ← 스키마 레이어 (이 파일)
├── graphify-out/                      ← primary machine-readable knowledge graph
└── raw/                               ← human-owned source corpus
    ├── design/
    │   ├── adr/                       ADR / 의사결정 기록
    │   ├── specs/                     설계 스펙
    │   ├── diagrams/                  다이어그램 source
    │   ├── screenshots/               시각 reference
    │   └── notes/                     설계 노트 / 개요
    └── external/
        ├── papers/                    논문/외부 문서
        ├── datasets/                  데이터셋 원문 문서
        ├── models/                    모델 원문 문서
        └── experiments/               실험 자료
```

### 레이어 1: 원문 소스 (`raw/`)

**진실의 원천(Source of Truth)**이다.

- `raw/design/**`는 설계/ADR/노트/다이어그램 source다
- `raw/external/**`는 논문/데이터셋/모델/실험 원문 source다
- `.agents/playbooks/`, `.agents/policies/`는 하네스 체크리스트/정책이며 Graphify source가 아니다

### 레이어 2: Primary Graph (`graphify-out/`)

현재 assistant가 먼저 읽어야 하는 machine-readable knowledge layer다.

- 아키텍처/코드베이스 질문에 답하기 전에 `graphify-out/GRAPH_REPORT.md`를 먼저 읽어 god node와 community 구조를 파악한다.
- `GRAPH_REPORT.md`만으로 부족하면 `graphify-out/graph.json`을 다음으로 보고, `raw/`는 source-level 확인에만 사용한다.

> 구체 실행 경로, 커밋 산출물, full refresh 입력 제약은 [`.agents/skills/graphify-manager/SKILL.md`](.agents/skills/graphify-manager/SKILL.md) 참조.

### 레이어 3: 스키마 (이 파일)

LLM에게 현재 knowledge model과 Graphify-first 운영 규칙을 알려주는 설정 문서다.
사용자와 LLM이 시간이 지남에 따라 함께 진화시킨다.

---
> Source: [pko89403/ZeroAlign-Rec](https://github.com/pko89403/ZeroAlign-Rec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
