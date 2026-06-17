## coolrl

> This repository uses Korean as the default language for project communication.

## Repository language policy

This repository uses Korean as the default language for project communication.

Required:
- Commit messages must be written in Korean.
- PR titles and PR descriptions must be written in Korean.
- GitHub issue comments and review comments must be written in Korean.
- Documentation changes must be written in Korean unless the document is intentionally maintained in English.
- Changelog and release notes must be written in Korean.

Exceptions:
- Code identifiers, filenames, package names, command names, CLI flags, config keys, protocol fields, and public API names should remain in English.
- Do not translate quoted external errors, log messages, dependency names, or upstream API names.
- Preserve the language of existing English documentation unless the task explicitly asks to translate it.

## Commit message policy

Commit messages must be written in Korean. Use this structure by default:

```text
<한 줄 요약>

맥락:
- 왜 바꾸는지, 어떤 실험 해석이나 운영 판단이 있는지

변경:
- 실제 수정 내용

확인:
- 실행한 테스트, 확인한 metrics, 링크, checkpoint 상태
```

Keep small commits compact, but do not leave the body empty for commits that affect experiment settings, results, algorithm decisions, evaluation criteria, checkpoint/data retention policy, or documentation archive decisions.

For those research/operation commits, write enough concrete detail that `git show` alone can recover the decision:
- `맥락`: 관찰한 문제, 변경을 유도한 실험 결과, 실패 모드, 운영 제약을 적는다.
- `변경`: 바꾼 config/path/code/doc 동작과 중요한 이름, 값, 경로를 적는다.
- `확인`: 실행한 명령, 확인한 metric, checkpoint/link 상태, 또는 검증하지 못한 이유를 적는다.

Avoid generic filler such as "문서 수정" or "테스트 확인" when a concrete path, metric, or command is available.

## Branch workflow policy

`stable`은 배포와 무거운 CI 기준 브랜치로 취급한다. 일상적인 개발 작업과 연구 기록은 `stable`에 직접 커밋하지 않는다.

기본 브랜치 역할:
- `stable`: 배포 가능 상태, GitHub Pages/릴리스 기준
- `main`: 일반 개발 통합 브랜치
- `research`: 연구 작업 통합 브랜치
- `experiment/<experiment_slug>`: 개별 실험 작업 브랜치

브랜치 이름 규칙:
- 실험 브랜치는 `experiment/<experiment_slug>` 형식을 사용한다.
- `<experiment_slug>`는 experiment directory slug와 맞춰 snake_case로 쓴다.
- 예: `experiment/slot_aware_playability`

운영 규칙:
- 새 실험은 `research`에서 분기한다.
- 실험 구현, config, progress, report, plot 변경은 해당 실험 브랜치에 커밋한다.
- 실험 종료 후 해당 브랜치를 `research`로 머지한다.
- 실험 커밋은 커밋 메시지 자체가 중요한 연구 기록이므로 기본적으로 squash하지 않는다.
- `research`의 안정화된 변경만 선별해 `main`으로 올린다.
- `stable`로의 머지는 배포/CI 비용과 안정성을 고려해 명시적으로 요청되었을 때만 수행한다.
- 현재 브랜치가 불명확하면 작업 전 `git status --short --branch`로 확인한다.

## Experiment record policy

This repository treats individual experiments as research records, not only as reusable config presets.

Use `experiments/<domain>/<experiment_slug>/` for new experiment records that need design notes, progress notes, analysis scripts, or result summaries.

Recommended layout:

```text
experiments/<domain>/<experiment_slug>/
  README.md
  config.yaml
  plan.md                # optional
  progress.md            # optional
  analyze.py              # optional
  report.md              # optional
  report.json            # optional
```

Experiment records are nested for human navigation, but checkpoint directories stay flat for runtime operations:

- Experiment record path: `experiments/<domain>/<experiment_slug>/`
- Experiment config path: `experiments/<domain>/<experiment_slug>/config.yaml`
- Experiment name: `<domain>_<experiment_slug>`
- Checkpoint path: `checkpoints/<experiment_name>`

Do not mirror the nested `experiments/` path under `checkpoints/`.

Good:

```text
experiments/lost_cities/deep_cfr_pure_self_play_zero_pit_poc_eps1e4/config.yaml
experiment_name: lost_cities_deep_cfr_pure_self_play_zero_pit_poc_eps1e4
checkpoint.directory: checkpoints/lost_cities_deep_cfr_pure_self_play_zero_pit_poc_eps1e4
```

Avoid:

```text
checkpoint.directory: checkpoints/lost_cities/deep_cfr_pure_self_play_zero_pit_poc_eps1e4
```

Naming and storage rules:
- Use snake_case for `<domain>`, `<experiment_slug>`, `experiment_name`, and checkpoint directory names.
- Keep raw artifacts such as model checkpoints, logs, and large runtime outputs in `checkpoints/`, which is git-ignored.
- If a run needs HDD/SSD/NVMe offload, create a local symlink at `checkpoints/<experiment_name>` and keep the real hardware path out of git.
- Keep `configs/` for reusable presets and legacy configs that have not moved yet.
- Use `README.md` as the experiment's anchor/index: concise purpose, current status, and links to the detailed files in the same directory.
- Prefer recording per-experiment design in `plan.md`, chronological run notes in `progress.md`, and final analysis/interpretation in `report.md` or `report.json` rather than growing `docs/` with one document per experiment.
- Use `docs/` for current usage guides, domain-level overviews, and archive material.

## Experiment runtime policy

긴 학습 실험은 가능하면 `tmux` session에서 실행한다.

- 1-5분 smoke run은 foreground에서 실행해 즉시 실패를 확인한다.
- 30분 이상 걸리는 학습 run은 `tmux new-session -d -s <experiment_slug> ...` 형태를 권장한다.
- 실험 시작 시 사용자에게 `tail -f checkpoints/<experiment_name>/train.log` 명령을 함께 제공한다.
- crash 후에도 tmux session이 살아있을 수 있으므로, 상태 확인 시 train process와 tmux session을 구분한다.
- 종료된 실험의 tmux session은 결과 확인 후 정리한다.

## Experiment analysis and plotting policy

실험별 분석 코드는 실험 record 안에 둔다. 새 실험은 필요하면 기존 실험의 `analyze.py`를 복사해 다음 위치에서 수정한다.

```text
experiments/<domain>/<experiment_slug>/analyze.py
```

실험별 `analyze.py`가 책임지는 내용:
- 해당 실험의 `metrics.jsonl` schema 해석
- metric, opponent, panel 선택
- stdout 요약
- 선택적 JSON/Markdown 리포트 생성
- 해당 실험의 plot 생성

권장 CLI 옵션:
- `--run`
- `--baseline-run`
- `--json-output`
- `--markdown-output`
- `--plot-output`
- `--write-report`
- `--smooth-window`
- `--no-plot`

실험별 `analyze.py`는 기본 실행에서 plot을 생성할 수 있다. 빠른 text-only 분석을 위해 `--no-plot`을 제공한다. 기본 plot 파일명은 `analysis_metrics.png`를 사용한다. `report.md`와 `report.json` 같은 기록 파일은 기본 검증 실행에서 덮어쓰지 말고, 명시적인 `--write-report` 옵션으로만 갱신한다.

실험별 monitoring plot은 `seaborn`과 `matplotlib`을 직접 사용해 단순하고 읽기 쉽게 만든다:
- `sns.set_theme(...)` 또는 local seaborn style/context 설정
- 단순 line plot
- 겹치지 않는 legend
- scale이 다른 metric의 panel 분리
- 명시적 smoothing 옵션이 필요한 경우 `--smooth-window`

실험별 metric 이름, opponent 의미, threshold, 해석 규칙을 공용 plotting abstraction으로 올리지 않는다.

`src/coolrl/lost_cities/deep_cfr/visualize.py`는 legacy compatibility 코드로 취급한다. 새 실험별 plot을 위해 이 파일을 확장하지 말고, 해당 실험의 `analyze.py`를 추가하거나 수정한다.

## Advisory workflow

Lost Cities Deep CFR처럼 실험 해석, 학습 안정성, 알고리즘 설계 판단이 애매한 작업에서는 필요할 때 MCP를 통해 Claude Opus 4.7 xhigh thinking에게 2차 의견을 구할 수 있다.

Guidelines:
- Claude도 이 repository에 직접 접근 권한이 있다고 가정한다. 긴 코드/문서 내용을 복사해 붙이기보다 관련 config, 코드 경로, checkpoint/metrics 경로, 현재 해석, 판단이 필요한 질문을 간결하게 전달한다.
- 자문 요청에는 가능한 한 실행 중인 실험 이름, 주요 metrics, 평가 command/path, 비교해야 할 후보를 포함한다.
- 자문은 의사결정 보조용이다. 최종 변경은 로컬에서 코드, 테스트, metrics를 직접 확인한 뒤 적용한다.
- 단순 구현 오류, 명확한 테스트 실패, 작은 리팩터링처럼 로컬에서 바로 판정 가능한 문제에는 자문을 기본값으로 쓰지 않는다.
- 자문 결과를 따를 때는 어떤 근거로 채택했는지와 어떤 부분은 보류했는지를 작업 요약에 남긴다.

---
> Source: [sebastianrcnt/coolrl](https://github.com/sebastianrcnt/coolrl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
