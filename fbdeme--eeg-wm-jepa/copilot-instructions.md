## eeg-wm-jepa

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EEG-WM-JEPA is a research project building an EEG foundation model that combines:
- **LeWM (LeWorldModel)** — JEPA-based world model using SIGReg regularization (no EMA/stop-gradient)
- **LUNA-style Channel Mixer** — Learned-query cross-attention for topology-invariant EEG processing
- **Spatiotemporal Masking** — Simultaneous time + spatial masking for self-supervised pretraining

The goal is a universal EEG encoder that works across any electrode montage/equipment, pretrained via self-supervised learning, evaluated on classification benchmarks (TUAB, TUSL, BCI, TDBrain).

## Architecture (3-Stage Pipeline)

### Stage 1: EEG Standardization
- Resample → 200Hz
- Re-reference → Common Average Reference
- Bandpass filter → 0.5–75Hz
- Segment → 2-second windows

### Stage 2: Self-Supervised Pretraining
```
Raw EEG [B, C, T] (C is variable per dataset)
  → Patch Embedder (depthwise 1D Conv, channel-independent)
  → Channel Mixer (3D electrode coords + Fourier encoding + cross-attention → fixed-size [B, N, D])
  → Spatiotemporal Masking (60%, 500ms–1s blocks)
  → Transformer Encoder (RoPE positional embeddings)
  → Predictor (predicts masked region latent vectors)
  → Loss: Prediction Loss (MSE) + SIGReg + Query Specialization Loss
```

### Stage 3: Downstream Evaluation
- Linear probing (encoder frozen) and fine-tuning (encoder trainable)
- Benchmarks: TUAB (normal/abnormal), TUSL (sleep staging), BCI Competition (motor imagery), TDBrain

## Key Technical Decisions

- **SIGReg over RVQ**: Continuous latent space (SIGReg) chosen over discrete tokenization (RVQ) for training stability. These two approaches are mathematically incompatible.
- **Minimal preprocessing**: No ICA or heavy artifact removal. JEPA's latent prediction naturally filters noise.
- **2-second windows**: Matches Laya/LuMamba for fair baseline comparison.
- **Channel Mixer from LUNA/Laya**: Handles variable channel counts (8–256) by projecting to fixed-size representation via learned queries.

## Reference Codebases

- LeWM: https://github.com/lucas-maes/le-wm (SIGReg + Predictor architecture)
- Laya: LeJEPA-based EEG foundation model (Channel Mixer + Patch Embedder reference)
- LuMamba: LeJEPA + Mamba backbone for EEG
- DELTA: RVQ tokenizer for EEG (not used, but studied for comparison)

## Project Structure

```
EEG-WM-JEPA/
├── CLAUDE.md                  # 이 파일 (Claude Code 가이드)
├── main.py                    # 사전학습 entry point (학습 루프, AMP, wandb, ckpt)
├── pyproject.toml             # uv 의존성 정의
├── uv.lock
├── .claude/
│   └── skills/
│       └── hp-tune/           # HP 튜닝 자동화 skill (W&B Bayes sweep + Claude 능동 프루닝)
│           ├── SKILL.md       #   Claude 운영 프로토콜 (폴링/프루닝 룰)
│           ├── sweep.yaml     #   W&B sweep 설정 (lr, sigreg_projections, grad_accum)
│           └── README.md      #   사용자용 시작 가이드
├── configs/
│   └── default.yaml           # 모델/학습/데이터 하이퍼파라미터 (canonical)
├── src/
│   ├── model/
│   │   ├── eeg_jepa.py        # 전체 모델 (Encoder + Predictor + Loss)
│   │   ├── patch_embedder.py  # depthwise 1D Conv patcher
│   │   ├── channel_mixer.py   # LUNA-style 학습 쿼리 cross-attention (가변 채널 → 고정)
│   │   ├── masking.py         # Spatiotemporal block masking
│   │   └── lewm_modules.py    # LeWM Transformer 블록 (RoPE, SIGReg)
│   ├── preprocessing/
│   │   ├── standardize.py     # resample/CAR/bandpass/window
│   │   ├── reve_loader.py     # REVE memmap + 메타데이터
│   │   ├── dataset.py         # 더미/오프라인 dataset
│   │   ├── streaming_dataset.py  # HF Parquet 스트리밍 (현 사용)
│   │   └── mds_dataset.py     # MosaicML Streaming (보류, MDS 변환 포기 후 미사용)
│   ├── training/              # 학습 유틸 (현 비어있음, main.py에 통합되어 있음)
│   └── evaluation/
│       ├── downstream.py      # Linear probing + fine-tuning
│       └── bci_dataset.py     # BCI IV 2a 로더 (MOABB)
├── scripts/
│   ├── preprocess_reve.py            # REVE → parquet 메인
│   ├── preprocess_pipeline.py        # 단일 파일 전처리
│   ├── preprocess_parallel.py        # 멀티프로세스 전처리
│   ├── preprocess_aggressive.py      # OOM 회피용 보수적 변형
│   ├── resume_preprocess.sh          # 중단 재개
│   ├── monitor_preprocess.sh
│   ├── convert_parquet_to_mds*.py    # parquet → MDS 변환 (3종, 보류)
│   ├── eval_bci.py                   # BCI IV 2a 평가
│   └── prepare_downstream_hf.py     # Downstream 데이터 전처리 + HF 업로드 (BCI, APAVA)
├── tests/
│   └── test_forward_pass.py   # 더미 데이터 forward/backward 검증
├── paper/
│   └── main.md                # 논문 초안
└── docs/                       # 문서 (구조는 아래 Documentation 섹션 참조)
```

새 모듈/스크립트/디렉토리를 추가하거나 기존 파일을 이동·삭제하면 **반드시 위 구조도를 함께 갱신**할 것. 한 줄 설명을 함께 적어 새로 합류하는 사람이 한눈에 파악할 수 있게 유지한다.

## Documentation Convention

`docs/`는 두 층으로 구성된다:

### 1. 핵심 프로젝트 매니징 (`docs/` 루트)

항상 최신 상태로 유지해야 하는 5개 파일. 작업 후 관련 항목은 반드시 업데이트할 것.

- **`docs/todo.md`** — 할 일 관리. 완료된 항목 `[x]` 체크 + 날짜 기록. 진행 중인 항목은 상태 표시 (예: "— 진행 중 (54/322)").
- **`docs/history.md`** — 의사결정/기술적 발견의 연대기. Phase 번호로 추가하고 변경 로그 테이블에 한 줄 요약.
- **`docs/issues.md`** — 기술적 이슈 트래커. Issue 번호 부여, 상태 표기 (✅ 해결됨 / ⚠️ 모니터링 중 / ⚠️ 보류 중). **해결된 이슈도 절대 삭제하지 말 것** (의사결정 근거 보존). 이슈가 여러 문서에 걸치면 issues.md를 canonical source로 두고 history.md엔 한 줄 요약만 추가.
- **`docs/research_method.md`** — 연구 방법론/핵심 개념 (concepts). 아키텍처·학습 방식·이론적 배경의 reference 문서.
- **`docs/current_status.md`** — 실시간 상태 스냅샷. "지금 어디까지 됐고 다음에 뭘 해야 하는가"에 답할 수 있어야 함. 다음 상황에서 반드시 업데이트:
  - 주요 작업 완료 또는 중단 (전처리, 변환, 학습 등)
  - 서버 변경 (새 서버 확보, 기존 서버 종료)
  - 전략 변경
  - 장기간 자리를 비울 때

### 2. 보조 문서 (`docs/supplementary/`)

특정 주제에 대한 깊이 있는 설계 문서. 해당 주제가 변경될 때만 갱신.

- **`docs/supplementary/data_pipeline.md`** — 데이터 파이프라인 설계 (전처리/스트리밍/포맷)
- **`docs/supplementary/hyperparameter_design.md`** — 하이퍼파라미터 결정 근거 및 비교
- **`docs/supplementary/loss_analysis.md`** — pred_loss vs sigreg_loss 개념/실측 분석, schedule/데이터 효과, LeJEPA 이론과 우리 실험 cross-check (paper §3.8 reference)
- **`docs/supplementary/hp_tuning_history.md`** — HP 튜닝 전체 실험 기록 (Sweep v1~M4), 확정 config, 핵심 교훈 (collapse-recovery dynamics, schedule/optimizer/lambda 결론)
- **`docs/supplementary/hardware_specs.md`** — 하드웨어 스펙 정리 (HP 튜닝: RTX 4090 1×, Full pretraining: RTX 4090 8×), 서버 선정 근거, 논문 기재용 요약
- **`docs/supplementary/pretrain_config.md`** — Full pretraining 계획 (확정 HP, steps 결정 근거, 논문 비교, 실행 명령어, 모니터링 지표, downstream 계획)
- **`docs/supplementary/downstream_data.md`** — Downstream eval 데이터 가이드 (HF repo, 포맷, 채널, 레이블, 다운로드 명령어, 라이선스, Quick Start)

### 3. 코드와 문서의 동기화

- **`configs/default.yaml`**: 하이퍼파라미터 값 변경 시 config에 반영하고 `hyperparameter_design.md`에 근거 기록.
- 코드만 고치고 문서를 안 고치면 안 됨. 사용자가 별도로 요청하지 않아도 관련 docs는 함께 업데이트할 것.
- **`issues.md`는 문제가 발생하거나 해결될 때마다 반드시 업데이트**.
- 새 보조 문서를 만들면 `docs/supplementary/`에 두고, 이 CLAUDE.md의 Documentation Convention 섹션에 한 줄로 등록한다.
- 새 디렉토리/모듈/스크립트를 추가하거나 구조를 변경하면 위 **Project Structure** 트리도 함께 갱신한다.

### 4. Step = docs 업데이트 + commit + push

`docs/`를 갱신할 정도의 진척(주요 작업 완료/중단, 의사결정, 이슈 발생·해결, 전략 변경, 구조 변경 등)은 **하나의 "step"으로 간주**한다. 이 단위가 완료되면 다음을 한 번에 수행한다:

1. 관련 docs를 모두 업데이트 (todo / history / issues / current_status / supplementary 중 해당되는 것)
2. 코드 + 문서 변경을 함께 commit (메시지에 무엇이 왜 바뀌었는지 명시)
3. 즉시 `git push`

작은 임시 수정이나 탐색용 변경에는 적용하지 않지만, "docs를 건드릴 만한 일"이라면 그 자체가 step의 신호다. step을 끝내지 않고 다른 작업으로 넘어가지 말 것.

## Long-Running Job Execution Pattern

학습/sweep/전처리 등 **장시간 돌아가는 명령**을 실행할 때는 반드시 다음 3단계를 지킨다. Claude Code 세션이 끊기거나 토큰이 압축되어도 작업이 살아 있어야 하기 때문.

### 1. 실행 전 — 하드웨어 가용성 체크

먼저 다음을 확인 후 사용자에게 보고하거나 진행 결정:
- **GPU 점유 상태**: `nvidia-smi --query-gpu=memory.used,memory.free,utilization.gpu --format=csv` — 다른 프로세스 점유 중이면 절대 새로 띄우지 말 것
- **기존 학습/sweep 프로세스 중복 여부**: `pgrep -af 'python.*main\.py'`, `pgrep -af 'wandb agent'` — 같은 종류 작업이 이미 돌고 있으면 사용자에게 확인
- **RAM/disk 공간**: `/dev/shm` cache 사용량 (`du -sh /dev/shm/parquet_cache`), overlay 디스크 여유 (`df -h /workspace`) — cache가 비었거나 부족하면 사전 준비
- 위 중 하나라도 막히면 실행하지 말고 사용자에게 보고

### 2. 실행 — `setsid` + `nohup` + 파일 리다이렉트로 detach

세션 독립성을 위해 다음 패턴을 항상 사용:

```bash
mkdir -p runs/<job_name> && \
setsid nohup /venv/main/bin/python main.py <args> \
  </dev/null >runs/<job_name>/train.log 2>&1 &
disown
```

핵심 요소:
- `setsid` — 새 세션 생성, controlling tty 분리 (Claude Code shell이 죽어도 살아남음)
- `nohup` — SIGHUP 무시 (이중 안전장치)
- `</dev/null` — stdin 분리
- `>runs/<job>/train.log 2>&1` — stdout/stderr 파일로 (절대 터미널로 흘리지 말 것)
- `& disown` — 백그라운드 + jobs 테이블에서 제거
- W&B run name은 `--wandb-run-name`으로 명시 (나중에 식별 가능)

`run_in_background=true`만 쓰는 건 부족하다 — Bash 도구 백그라운드는 Claude 세션에 묶여 있어 토큰 압축 시 끊길 수 있음. 반드시 위 setsid 패턴과 병용.

### 3. 실행 후 — 살아있는지 확인

실행 직후 5~10초 sleep 후 다음을 확인하고 사용자에게 보고:
- **프로세스 살아있는지**: `pgrep -af '<job_name>'` — Python PID + setsid가 자식으로 분리됐는지
- **로그 첫 출력**: `tail -20 runs/<job_name>/train.log` — config 로드, 모델 build, wandb run id, 첫 step까지 떴는지
- **GPU에 올라갔는지**: `nvidia-smi` 메모리 사용량 변화
- 보고에 포함: PID, wandb run URL, 로그 경로, 예상 소요시간

위 셋 중 하나라도 이상이 있으면 즉시 디버깅. "백그라운드 띄웠으니 끝"이 아니라 첫 step이 정상적으로 시작된 걸 본 후에야 작업 완료로 간주.

## Research Context

Prior conversation history and paper analyses are stored in `EEG Preprocessing Methods in Research Papers.zip`. This contains markdown reports and paper excerpts covering: Brain-JEPA, EEG-VJEPA, S-JEPA, HEAR, LUNA, SPEED, DELTA, LaBraM, NeuroGPT, EEGPT, DeWave, Laya, LuMamba, and LeWM.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fbdeme) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
