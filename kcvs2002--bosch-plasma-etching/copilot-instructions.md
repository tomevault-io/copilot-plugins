## bosch-plasma-etching

> 이 문서는 Codex가 매 세션 자동 로드하는 프로젝트 헌법이다.

# 프로젝트 규칙 — BOSCH Plasma-Etching VM

이 문서는 Codex가 매 세션 자동 로드하는 프로젝트 헌법이다.
새 작업을 시작하기 전에 반드시 이 규칙을 참고한다.

---

## 1. 프로젝트 개요 (한 줄 요약)

BOSCH DRIE 플라즈마 에칭 공정의 Virtual Metrology — OES + Process 센서 → 웨이퍼 측정값 (si_etch, oxide_etch) 을 Cycle-Aware Deep Learning 으로 예측하는 학부 졸업논문 프로젝트.

자세한 도메인·전략은 [docs/연구계획서_초안.md](docs/연구계획서_초안.md) 참조.

### 1.1 현재 진행 상황 (2026-05-08)

- **Phase 1·2 완료, Phase 3 single-fold 완료, Phase 4 일부 완료, 중간 발표 마침.**
- **Best 모델:** [outputs/experiments/2026-05-01_00-56_dl-multimodal-singlefold/](outputs/experiments/2026-05-01_00-56_dl-multimodal-singlefold/) — Multimodal (FiLM + Fourier xy + mean pool). oxide RMSE 0.0399 / R² 0.734 (XGB 대비 -22%, 졸업논문 종료기준 1번 달성).
- **다음 작업 후보:** ① 5-fold 확장으로 안정성 (std/mean ≤ 10%) 검증, ② sequence/encoder ablation (연구계획서 Exp 5/6), ③ 해석 분석 확장 (현재 oxide fold0 만).
- **확정된 설계 결정** (다른 agent 가 다시 시도하지 말 것):
  - `pool=attention` 은 oxide 에서 악화 → mean pool 유지
  - `use_film=true`, `xy_n_freqs=6` 필수 (없으면 si RMSE 폭발)
  - OES-only, Proc-only ablation 완료. multimodal 이 둘보다 +0.094 R² 우월
- 자세한 진행 상황·실험 폴더 라벨링은 [memory/project_progress.md](memory/project_progress.md), [memory/reference_experiments.md](memory/reference_experiments.md), [memory/project_dl_design_decisions.md](memory/project_dl_design_decisions.md) 참조.

---

## 2. 폴더 구조와 각 폴더의 역할

```
BOSCH Plasma-Etching/
├── AGENTS.md                  # 이 파일 (프로젝트 규칙)
├── requirements.txt           # pip pinned (torch는 cu124 인덱스)
├── .gitignore
│
├── Dataset/                   # 원본 데이터. 절대 수정 금지. read-only.
│
├── src/                       # 재사용 라이브러리 코드. IMPORT 전용.
│   ├── data/                  #   로더, cycle 세그멘테이션, 캐시 I/O
│   ├── features/              #   cycle tensor 조립, 정규화, 피처 엔지니어링
│   ├── models/                #   모델 아키텍처 (CNN, LSTM, Transformer, baseline)
│   ├── training/              #   학습 루프, loss, optimizer, scheduler
│   ├── evaluation/            #   metrics, GroupKFold split
│   └── utils/                 #   make_experiment_dir, set_seed, config IO 등
│
├── configs/                   # 실험 config YAML. 1 실험 = 1 파일.
│
├── scripts/                   # 실행 엔트리. `python -m scripts.NN_name` 으로 실행.
│   ├── 01_build_cache.py      #   전처리: raw → cache/vN/
│   ├── 02_make_splits.py      #   GroupKFold 분할 저장
│   ├── 03_train.py            #   XGBoost baseline 학습
│   ├── 04_train_dl.py         #   Cycle-Aware DL 학습 (2D-CNN + Bi-LSTM)
│   ├── 05_interpret.py        #   해석 분석: XGBoost SHAP + DL gradient attribution
│   └── 06_draw_architecture.py#   아키텍처 다이어그램 생성
│
├── notebooks/
│   ├── eda/                   # 정식 EDA 스크립트 (재사용, 보존)
│   └── scratch/               # 일회성 탐색 (주기적 정리·삭제 OK)
│
├── cache/                     # 전처리 산출물. gitignored. 재생성 가능.
│   └── vN/                    #   전처리 파이프라인 바뀔 때 v2, v3 로 버전업
│
├── outputs/
│   ├── figures/               # EDA·분석용 독립 그림 (특정 실험 소속 아님)
│   └── experiments/           # ★ 모든 학습/평가 실행 결과 (규칙 3 참조)
│
├── docs/                      # 보고서, 계획서, 발표자료, 다이어그램
│
└── memory/                    # Codex auto-memory (건드리지 말 것)
```

### 폴더별 원칙

- **`src/`는 라이브러리, `scripts/`는 엔트리포인트.** 섞지 말 것. `src/` 모듈은 import-time 부작용 금지 (print, 파일쓰기, CUDA 초기화 등). 실행은 반드시 `scripts/`를 거친다.
- **`Dataset/`에는 절대 쓰지 않는다.** 원본은 수정도 파생파일 저장도 금지. 모든 산출물은 `cache/` 또는 `outputs/` 로 간다.
- **`notebooks/scratch/`는 일회용 쓰레기통.** 가설 검증·디버깅용. 보존 가치가 있으면 `notebooks/eda/`로 승격하거나 `src/`로 흡수한다.

---

## 3. 실험 결과 저장 규칙 ★ (최우선)

> **모든 학습/평가 실행은 `outputs/experiments/` 아래에 새 폴더를 만들어 그 안에 결과를 저장한다.**

### 3.1. 폴더 이름

```
outputs/experiments/<YYYY-MM-DD_HH-MM>_<slug>/
```

- **앞부분은 실행 시작 시각** (분 단위까지). 이름순 정렬 = 시간순 정렬이 되도록 한다.
- **뒷부분은 실험 제목 슬러그** (영문 소문자, 하이픈 구분). 예: `baseline-xgb`, `cnn-lstm-v1`, `oes-only-ablation`.
- 예시: `outputs/experiments/2026-04-17_15-30_baseline-xgb/`

### 3.2. 폴더 내부 구조

```
<experiment-dir>/
├── config.yaml        # 실행에 사용된 config 복사본 (필수)
├── metrics.json       # 최종 성능 지표 (필수)
├── NOTES.md           # 실험 목적·설정요약·결과·배운점 (필수, 자동 생성)
├── logs/              # stdout 로그, train/val 로스 곡선 csv
├── checkpoints/       # 모델 가중치 (best, last)
└── figures/           # 이 실험에서 나온 그림만
```

### 3.3. 폴더 생성은 반드시 `make_experiment_dir` 사용

```python
from src.utils import make_experiment_dir
exp_dir = make_experiment_dir("baseline xgb")
# → outputs/experiments/2026-04-17_15-30_baseline-xgb/ 생성 + 하위 폴더 + NOTES.md 시드
```

손으로 mkdir 하지 말 것. 타임스탬프 형식이 틀리면 정렬이 깨진다.

### 3.4. 기존 폴더 재사용 금지

실험을 다시 돌리면 **새 폴더**를 만든다. 덮어쓰기·이어쓰기는 선후관계를 파괴한다.
(예외: 중간에 죽은 학습을 체크포인트에서 이어갈 때만 같은 폴더 사용 가능. NOTES.md 에 기록.)

### 3.5. `outputs/figures/` vs 실험 폴더의 `figures/`

- **특정 실험에 속하는 그림** → `<experiment-dir>/figures/`
- **실험 독립적인 그림** (EDA, 데이터셋 개요, 전처리 검증) → `outputs/figures/`

EDA 그림은 숫자 prefix로 구분: `01_oes_cycle_overview.png`, `08_gasflow_cycles.png` 등.

---

## 4. 코드 규칙

### 4.1. 실행 방식

프로젝트 루트에서 모듈로 실행한다. **가상환경은 `.venv\python.exe`** 를 사용한다:

```bash
# XGBoost baseline
.venv\python.exe -m scripts.03_train --config configs/exp_baseline_xgb.yaml

# DL 학습 (Cycle-Aware BiLSTM)
.venv\python.exe -m scripts.04_train_dl --config configs/exp_dl_multimodal_singlefold.yaml

# 해석 분석 (SHAP + gradient attribution)
.venv\python.exe -m scripts.05_interpret --dl-exp <exp_dir> --xgb-exp <exp_dir> --target oxide_etch

# 아키텍처 다이어그램
.venv\python.exe -m scripts.06_draw_architecture
```

(`python scripts/...` 직접 실행도 되게 `sys.path` 조작은 하지 않는다 — `-m` 로 충분.)

### 4.2. Config 기반 재현성

- 하이퍼파라미터·경로·seed 등은 `configs/*.yaml`에 둔다.
- 스크립트는 `--config` 인자로 YAML 을 받는다.
- 실험 시작 시 **config를 experiment 폴더로 복사**해서 "무슨 설정으로 돌렸나"를 고정한다.

### 4.3. Seed

- `src.utils.set_seed(seed)` 를 모든 랜덤성 있는 스크립트 최상단에 호출.
- seed 는 config에 명시.

### 4.4. 타입·스타일

- Python 3.11+, `from __future__ import annotations`.
- `@dataclass` 선호, 튜플/딕셔너리 반환보다 타입이 있는 컨테이너.
- docstring 은 모듈·공개 함수에만. 내부 함수·명백한 코드에는 주석 달지 말 것.
- 주석은 "WHY"가 비자명할 때만.

### 4.5. 의존성

- 새 패키지 추가시 반드시 `requirements.txt` 에 pinned version 추가.
- torch 는 cu124 index 유지 (`--index-url https://download.pytorch.org/whl/cu124`).
- jupyter 계열은 설치하지 않는다 (사용자 선호 — .py 스크립트로 작업).

---

## 5. 데이터 & 캐시 규칙

### 5.1. Dataset 경로

- 절대 경로: `Dataset/` (프로젝트 루트 기준). 로더가 `DATASET_DIR` 상수로 관리.
- Windows 한글 경로 이슈: `netCDF4` 는 한글 경로를 열지 못한다. [src/data/loader.py](src/data/loader.py) 의 `_cwd()` 컨텍스트매니저가 chdir로 우회한다 — 새로 netCDF 를 열 땐 이 패턴을 따를 것.

### 5.2. Cache 버전 규칙

- `cache/v1/`, `cache/v2/`, ... — 전처리 파이프라인이 바뀌면 버전 올림.
- 각 버전 디렉터리 루트에 `README.md` 넣어 "v1 과 다른 점", "생성 스크립트·커밋 해시" 기록.
- **캐시는 재생성 가능해야 한다.** 캐시만 있고 생성 로직이 사라지면 안 된다 — 생성 스크립트 + config 를 명시.

### 5.3. CV split

- 반드시 **wafer 단위 GroupKFold** (같은 웨이퍼의 89 포인트가 train/val 에 분산되면 누수).
- split 인덱스는 `cache/vN/splits/kfold_K.npz` 로 저장, 학습 스크립트는 이걸 로드만 한다.

### 5.4. DL 전처리 캐시 (RunPod / cloud GPU)

- DL 학습 전 CPU-heavy 단계는 `scripts/10_prepare_dl_cache.py` 로 미리 생성한 캐시를 재사용한다.
- RunPod 업로드용 대표 생성 명령은 `normalizers` 레벨을 권장한다. `dl_tensors` 는 split seed와 무관하게 1회만 공유되고, fold별 normalizer/score 캐시는 작다:

```bash
.venv\Scripts\python.exe -m scripts.10_prepare_dl_cache --config configs/exp_dl_multimodal_oes_aux_mixup_ema_longrun_5fold.yaml --level normalizers
```

- `--level normalized` 는 가장 빠르지만 full OES tensor를 fold별로 복제해 seed당 수십~100GB까지 커질 수 있으므로 로컬 SSD 검증용으로만 쓴다. RunPod 업로드 대상에서는 보통 제외한다.
- `scripts/04_train_dl.py` 의 기본값은 `data.dl_cache.mode: "auto"` 이다. 즉 config에 `dl_cache`가 없어도 사용 가능한 DL 캐시가 있으면 자동으로 가장 빠른 캐시(`normalized` → `normalizers` → `tensors`)를 사용하고, 없으면 기존 방식으로 실행한다.
- `data.dl_cache.mode: "normalized"` 를 명시한 학습은 `cache/vN/dl_tensors/`, `cache/vN/dl_normalizers/`, `cache/vN/dl_normalized/`, `cache/vN/features/oes_scores/` 를 로드한다.
- 다음 항목이 바뀌면 기존 DL 캐시는 무효일 수 있으므로 반드시 재생성해야 한다:
  - `cache_version`
  - `split_file`
  - `t_o`, `t_p`
  - `per_wafer_norm`
  - `xgb_feat_names`
  - 공통 process channel 선택/정렬 로직
  - OES/Process cycle tensor resampling 로직
  - OES log/normalization 방식, process normalization 방식, scalar XY/target normalization 방식
  - OES wavelength selection score 계산 로직 또는 `oes_band_selection` 설정
- 위 항목에 영향을 주는 코드나 config를 수정한 agent는 최종 답변에서 **반드시 사용자에게 DL 캐시 재생성 필요 여부와 실행 명령**을 알려야 한다.
- 학습 전용 설정만 바뀐 경우 (`epochs`, `lr`, `batch_size`, `dropout`, `mixup`, `ema`, 모델 hidden size 등)에는 일반적으로 DL 캐시 재생성이 필요 없다.
- 확실히 캐시를 강제하고 싶을 때만 config에 `data.dl_cache.require: true` 를 둔다. 기본 auto 모드는 캐시가 없거나 일부 누락되면 기존 계산 경로로 fallback 한다.

---

## 6. 작업 습관 (Codex 행동 규칙)

- **새 작업 시작 전에 이 파일을 읽는다.** 특히 "실험 결과 저장 규칙"은 매번 확인.
- **질문·수정 요청이 들어오면 먼저 관련 파일을 `Read`** 해서 현재 상태를 확인한 뒤 응답.
- **파일·폴더 새로 만들기 전에 이 문서의 구조와 일치하는지 확인.** 애매하면 사용자에게 묻는다.
- **실험 결과 폴더를 수동 mkdir 하지 말 것.** `make_experiment_dir` 경유.
- **한글 파일·폴더 이름 피하기.** Windows netCDF·일부 툴에서 경로 문제 유발. 문서(`docs/*.md`)는 예외.
- **커밋·push 같은 공유 변경은 사용자 명시 요청 시에만.** 로컬 파일 작업은 자유.

---

## 7. 워크플로 요약 (새 실험 실행 절차)

**XGBoost 실험:**
1. `configs/exp_xxx.yaml` 작성
2. `.venv\python.exe -m scripts.03_train --config configs/exp_xxx.yaml` 실행
3. 스크립트 내부에서 `make_experiment_dir("exp xxx")` 호출 → 실험 폴더 자동 생성
4. config 복사 + 학습 + metrics.json + checkpoint(.json) 저장
5. 실험 종료 후 `NOTES.md` 업데이트

**DL 실험:**
1. `configs/exp_dl_xxx.yaml` 작성
2. `.venv\python.exe -m scripts.04_train_dl --config configs/exp_dl_xxx.yaml` 실행
3. checkpoint(.pt) 에 모델 가중치 + normalizer stats 저장됨 (05_interpret 에서 재사용)
4. 실험 종료 후 `NOTES.md` 업데이트

**해석 분석:**
- `.venv\python.exe -m scripts.05_interpret --dl-exp <dir> --xgb-exp <dir> --target oxide_etch`
- XGBoost SHAP + DL gradient attribution 결과를 각 실험의 `figures/` 에 저장

---

## 8. 참고 문서

- 연구 계획: [docs/연구계획서_초안.md](docs/연구계획서_초안.md)
- 생성된 아키텍처 다이어그램: [outputs/figures/arch_xgboost.png](outputs/figures/arch_xgboost.png), [outputs/figures/arch_dl_multimodal.png](outputs/figures/arch_dl_multimodal.png)

---
> Source: [KCVS2002/BOSCH-Plasma-Etching](https://github.com/KCVS2002/BOSCH-Plasma-Etching) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
