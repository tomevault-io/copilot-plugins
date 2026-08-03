## synapse-enforcer-leet

> LEET 언어이해 고난도 지문·문항 세트 컴파일러. **생성 = 로컬 Gemma4-31B-it(사고 모드)**,

# CLAUDE.md — 작업 가이드

## 프로젝트 한 줄
LEET 언어이해 고난도 지문·문항 세트 컴파일러. **생성 = 로컬 Gemma4-31B-it(사고 모드)**,
**판정 = 사람**(스틸맨 시트 O/X). 설계 근거는 [docs/METHODOLOGY.md](docs/METHODOLOGY.md), 성능 좌표는 [docs/BENCHMARK.md](docs/BENCHMARK.md).

## 설계 불변 원칙 (수정 전 확인)
- **검증 분리**: 생성 모델에게 진위의 최종 판정을 맡기지 않는다(판정 상한 실측 — METHODOLOGY §2).
  스틸맨은 열거 전용, 봉쇄 판정은 사람(또는 배치용 대형 모델 표결).
- **게이트는 결정론 우선**: 코드로 잡을 수 있는 것(길이·극성·복사·슬롯·스페이싱)은 콜로 잡지 않는다.
- **외부 의존성 0**: 표준 라이브러리만 사용한다. 새 pip 의존성 추가 금지.
- 진위 판정·문항 수리 방향·룰 변경은 저장소 소유자(사람)의 확인을 거친다.

## 실행
```bash
bash serving/serve_gemma4_31b_it_vllm.sh                  # vLLM 서빙 (localhost:8000, 사고 켬)
python3 native/native_set.py                              # 세트 생성 → native/runs/
python3 native/set_to_hwpx.py native/runs/native_set_v0.json   # HWPX 조판
python3 native/serve_native.py --port 8977                # 웹 러너
python3 application/engine/generate.py <knowledge.json> --think-native --stop-after passage --out <dir>  # 지문
cd application && python3 app.py --port 8899              # 클래식 웹 UI
```
환경변수: `LEET_PASSAGE_DIR`(지문 폴더) · `LEET_LADDER_TAG`(산출 태그) · `LEET_OUT`(출력 폴더) ·
`LEET_BASE_URL`/`LEET_MODEL`(서버·모델 강제) · `LEET_REQ_KW`(요청 파라미터 JSON 병합 — 예: `{"reasoning_effort":"high","temperature":1}`).

## 구조 급소
- `native/nq_modules.py` — 문항 문법 정본(L1 불변 계약 / L2-COMMON / 유형 코어 3종 / L3 실물).
  **수정 시 사다리 검증 필수**: 쌩 대조 → 통합 1콜 → 직접 풀이 채점(유일성·홉·현저성·복사) 후 커밋.
- `native/native_set.py` — 게이트 체인(축 배분→레인→린트→탐지→수리→스틸맨). 프롬프트 수정과
  게이트 수정을 한 커밋에 섞지 말 것(회귀 원인 분리).
- `application/engine/generate.py` — 클래식 파이프라인 + 지문 네이티브 경로. 5,500줄 단일 파일 —
  광역 리팩터 금지, 국소 수정 원칙.
- 산출물(`native/runs/`, `out/`, `*.log`)은 .gitignore — 커밋하지 않는다(예시 승격 시 examples/로 복사).

## 백로그 (v1)
1. **표적 수리 콜에 세트 컨텍스트 주입** — 수리가 자기 문항만 보고 타 문항과 재료 중복을 만든 실측 있음 (최우선)
2. 정답 결함(반전형 등) 시 해당 레인 재생성 1회 루프
3. 린트 정밀화: 부정 술어 문맥의 극성 오탐 면제("~하기 어렵다"), 30~40자 짧은 슬롯을 길이대역 검사에 인정
4. 세트 탐지의 복합 문항 표기("Q1, Q3") 파싱 · 수리 우선순위/라운드
5. custom 발문 실증(구절 의미형 — 이해 코어 + 국소 앵커 파라미터) · 네이티브 경로의 웹 UI(app.py) 통합

## 커밋 규칙
작동 단위로 커밋하고 push로 공개 동기화한다. 산출물 파일·개인 경로·비밀은 커밋 금지.

---
> Source: [Glockevonpavlov/synapse-enforcer-leet](https://github.com/Glockevonpavlov/synapse-enforcer-leet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
