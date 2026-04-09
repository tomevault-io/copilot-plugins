## vuldocker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`vulDocker`는 LLM+RAG를 활용해 취약 환경을 자동 합성/보강하고, Docker에서 실행·검증·리뷰·패키징까지 파이프라인을 수행하는 테스트베드입니다. PoC/취약 환경은 **격리된 로컬 Docker**에서만 사용됩니다. 상세 설계는 `docs/handbook.md` 참고.

## Setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

**API 키 설정 (선택)** — 미설정 시 결정적 스텁으로 동작:
- `config/api_keys.ini` → `[openai] api_key = sk-...`
- 또는 환경변수: `OPENAI_API_KEY` / `VUL_LLM_API_KEY`

## Commands

### 파이프라인 단계별 실행 (SID 필요)

```bash
# 1. PLAN (SID 생성)
python orchestrator/plan.py --input inputs/mvp_sqli.yml

# 2. E2E 루프 실행 (GENERATE→EXECUTE→VERIFY→REVIEW까지 자동)
python orchestrator/run_pipeline.py --sid <SID> --mode deterministic

# 단계별 실행:
python agents/researcher/main.py --sid <SID> --mode deterministic
python agents/generator/main.py --sid <SID> --mode deterministic
python executor/runtime/docker_local.py --sid <SID> --build --run
python evals/poc_verifier/main.py --sid <SID>
python agents/reviewer/main.py --sid <SID> --mode deterministic
python orchestrator/pack.py --sid <SID> [--allow-intentional-vuln]
```

### 테스트

```bash
# 전체 단위 테스트
pytest

# 단일 테스트 파일
pytest tests/test_plan_sid_isolation.py

# 단일 테스트 함수
pytest tests/test_plan_sid_isolation.py::test_function_name -v

# E2E 테스트 (Docker 필요)
VULD_RUN_E2E=1 pytest -m e2e

# CI 스모크
bash ops/ci/smoke_regression.sh
bash ops/ci/run_case.sh
```

## Architecture

### 파이프라인 흐름

```
inputs/*.yml → PLAN → [RESEARCH] → GENERATE → EXECUTE → VERIFY → REVIEW → PACK
```

각 단계는 `SID`(Scenario ID) 기준으로 아티팩트를 분리 저장:
- `metadata/<SID>/` — plan.json, generator_runs.json, reviewer_reports.json, loop_state.json 등
- `workspaces/<SID>/app/` — 생성된 앱 코드, Dockerfile, PoC
- `artifacts/<SID>/build/` — build.log, sbom.spdx.json
- `artifacts/<SID>/run/` — run.log, summary.json, index.json
- `artifacts/<SID>/reports/` — evals.json

### SID 결정론적 해시

`SID = sha256(model_version | prompt_hash | seed | retriever_commit | corpus_snapshot | pattern_id | deps_digest | base_image_digest | ...)[:12]` — `common/sid.py:compute_sid`

### 핵심 모듈

| 모듈 | 역할 |
|------|------|
| `orchestrator/plan.py` | 요구 YAML 정규화 → SID 계산 → plan.json 작성 |
| `orchestrator/run_pipeline.py` | LoopController 기반 E2E 루프 (Reflexion 메모리 포함) |
| `orchestrator/pack.py` | 산출물 수집/스냅샷/manifest.json 생성 |
| `orchestrator/loop_controller.py` | 반복 제어 및 실패 맥락 주입 |
| `common/sid.py` | `compute_sid()` — SID 필드 해시 |
| `common/paths.py` | metadata/workspaces/artifacts 경로 규칙 |
| `common/run_matrix.py` | 단일/다중 취약 번들 처리 |
| `common/llm/provider.py` | litellm 백엔드/스텁 자동 전환 (`LLMClient`) |
| `common/config/api_keys.py` | `config/api_keys.ini` API 키 로더 |
| `common/contracts.py` | Generator 계약 빌드/로드/검증 |
| `common/variability/manager.py` | Variation Key 정규화/디코딩 프로파일 |
| `agents/generator/service.py` | 템플릿/합성(hybrid) 모드 선택, 가드레일 적용 |
| `agents/generator/synthesis.py` | 매니페스트 기반 합성 엔진 (결정적 폴백 포함) |
| `agents/reviewer/service.py` | 로그/정적 이슈 분석, LLM 피드백, 루프 상태 기록 |
| `executor/runtime/docker_local.py` | Docker build/run, readiness, SBOM(Syft), 보안 옵션 |
| `evals/poc_verifier/main.py` | 플러그인 레지스트리 기반 검증 집계 |

### Generator 모드

- `deterministic` — 스텁 기반, LLM 키 없이도 동작
- `diverse` — temperature/top-p/k 변이
- `hybrid` — 합성(synthesis) 우선 → 실패 시 템플릿 폴백 (기본값)

Generator는 `workspaces/templates/**/template.json` 탐색 후 매칭 여부에 따라 경로를 결정.

### Eval 룰

검증 기준은 `docs/evals/rules/<CWE>.yaml`(정적) 및 `metadata/<SID>/runtime_rules/*.yaml`(Researcher가 생성, 런타임)에 정의. `success_signature` + `flag_token` 모두 run.log에서 검출되어야 PoC 성공 판정.

### LLM 클라이언트

`common/llm/provider.py:LLMClient`는 litellm 미설치 또는 키 미설정 시 자동으로 결정적 스텁으로 폴백. 테스트에서는 `tests/` 내 픽스처 파일을 사용.

### 입력 파일 형식

`inputs/*.yml` — CWE, 스택, 변이키, RAG 스냅샷, 정책을 명시. 예시: `inputs/mvp_sqli.yml`, `inputs/multi_vuln_example.yml`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sw1029)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sw1029)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
