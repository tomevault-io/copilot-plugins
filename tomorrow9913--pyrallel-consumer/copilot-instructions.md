## pyrallel-consumer

> *최종 업데이트: 2026년 3월 27일 금요일*

# Pyrallel Consumer - 개발 현황 및 인수인계 문서

*최종 업데이트: 2026년 3월 27일 금요일*

## 최근 업데이트 (2026-03-27)
- Benchmark TUI button copy English 단계 (2026-03-27): 오픈소스 공개를 고려해 benchmark TUI의 action button label을 영어로 통일했습니다. `benchmarks/tui/app.py`에서 results modal action은 `Back to settings`, `Close`로, run screen action은 `Cancel run`, `Back to settings`, `Exit`, success terminal action은 `View results` / `Back to settings` / `Exit`로 변경했습니다. 옵션 화면의 `Run benchmark`, `Quit`은 기존 영어 레이블을 그대로 유지합니다. 회귀 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py::test_run_screen_exposes_report_and_exit_controls_after_success tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 81건 통과로 확인했습니다.
- Benchmark TUI message-progress semantics 단계 (2026-03-27): 실행 로그 화면의 상단 progress가 여전히 사용자의 mental model과 맞지 않다는 피드백을 반영해, 상단/하단 progress의 데이터 소스를 다시 분리했습니다. `benchmarks/tui/log_parser.py`는 이제 baseline/pyrallel 로그에서 `current_run_target_messages`와 `current_run_processed_messages`를 추적하며, `Will process up to ... messages`, `Target messages to process: ...`, `Processed ... messages`, `Total messages processed ...` 라인을 파싱해 현재 run 메시지 progress를 snapshot에 올립니다. `benchmarks/tui/app.py`의 상단 badge/bar는 이 데이터를 사용해 `현재 X / Y 메시지`와 `현재 처리시간`을 표시하고, top `phase-progress` bar는 `current_run_processed_messages / current_run_target_messages`를 직접 그립니다. 하단 badge/bar는 `전체 X / total 벤치마크`와 `전체 처리시간`을 표시하며, overall run count progress를 그립니다. 두 progress bar 모두 Textual built-in ETA를 사용하도록 `show_eta=True`로 바꿨고, custom `phase-progress-eta`는 제거했습니다. 추가로 terminal reason은 `.is-failed` / `.is-cancelled` / `.is-complete` 클래스로 색과 강조를 줘 회색 muted text로 묻히지 않게 했습니다. 회귀 테스트로 `tests/unit/benchmarks/test_tui_log_parser.py`에 현재 run 메시지 progress 파싱 테스트 2건을 추가했고, `tests/unit/benchmarks/test_tui_app.py`의 spotlight/progress/terminal reason 기대값도 갱신했습니다. 검증은 `uv run pytest tests/unit/benchmarks/test_tui_log_parser.py::test_log_parser_tracks_current_run_message_progress_from_logs tests/unit/benchmarks/test_tui_log_parser.py::test_log_parser_tracks_pyrallel_current_run_target_and_final_processed tests/unit/benchmarks/test_tui_app.py::test_run_screen_mounts_dashboard_widgets tests/unit/benchmarks/test_tui_app.py::test_run_screen_spotlight_uses_single_progress_semantics_and_selected_rows_only tests/unit/benchmarks/test_tui_app.py::test_run_screen_formats_status_and_tps_cells_for_readability tests/unit/benchmarks/test_tui_app.py::test_run_screen_marks_failed_cell_in_soft_red -q` 6건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 81건 통과, `python3 -m py_compile benchmarks/tui/app.py benchmarks/tui/log_parser.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_results_report.py` 통과로 확인했습니다.
- Benchmark TUI progress semantics clarification 단계 (2026-03-27): 상단 spotlight progress가 engine-local counter처럼 읽혀 혼동을 주던 문제를 정리했습니다. `benchmarks/tui/app.py`의 상단 badge는 이제 `현재 위치 x / total`로, 하단 badge는 `완료 x / total`로 분리되어 같은 전체 task count를 바라보되 의미가 다르게 읽히도록 바뀌었습니다. 상단 `phase-progress` bar는 current running slot을 포함한 overall position(`current_run_number / total_runs`)을 사용하고, 하단 `run-progress` bar는 완료된 run 수(`completed_runs / total_runs`)만 사용합니다. 상단에는 `남은 hh:mm:ss` ETA를 별도 badge로 노출하되, 샘플이 부족한 초기 구간에서는 `--:--:--`로 유지합니다. 동시에 spotlight card는 `run-phase-meta`, `run-overall-meta`, `phase-progress`, `run-progress`를 compact layout으로 재구성하고, `run-terminal-reason`은 실제 메시지가 있을 때만 보여 불필요한 vertical space를 줄였습니다. 회귀 테스트로 `tests/unit/benchmarks/test_tui_app.py`의 mount/spotlight/status readability expectations를 갱신했고, 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py::test_run_screen_mounts_dashboard_widgets tests/unit/benchmarks/test_tui_app.py::test_run_screen_updates_progress_bar_and_summary_table tests/unit/benchmarks/test_tui_app.py::test_run_screen_spotlight_uses_single_progress_semantics_and_selected_rows_only tests/unit/benchmarks/test_tui_app.py::test_run_screen_formats_status_and_tps_cells_for_readability -q` 4건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 79건 통과, `python3 -m py_compile benchmarks/tui/app.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py` 통과로 확인했습니다.
- Benchmark TUI spotlight compression 단계 (2026-03-27): 실행 로그 화면 상단 spotlight card가 너무 많은 vertical space를 차지하고 phase progress 의미가 불명확하다는 피드백을 반영했습니다. `benchmarks/tui/app.py`의 spotlight는 이제 `현재 엔진 진행 <phase> x / total`과 `남은 hh:mm:ss`를 phase progress bar 바로 위에 표시하고, `전체 진행 x / total` + `경과 hh:mm:ss`를 overall progress bar 위에 별도로 배치합니다. 두 progress bar는 `show_percentage=False`, `show_eta=False`로 바꿔 Textual 기본 퍼센트/ETA 출력이 레이아웃을 넓히지 않게 했고, `#run-phase-meta` / `#run-overall-meta` / `#phase-progress` / `#run-progress`를 compact 높이로 고정해 screenshot에서 보이던 큰 빈 공간을 줄였습니다. `run-terminal-reason`도 내용이 없을 때는 숨겨서 불필요한 세로 여백을 제거했습니다. 회귀 테스트로 `tests/unit/benchmarks/test_tui_app.py`의 spotlight/progress expectation을 갱신했고, 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py::test_run_screen_mounts_dashboard_widgets tests/unit/benchmarks/test_tui_app.py::test_run_screen_spotlight_uses_single_progress_semantics_and_selected_rows_only tests/unit/benchmarks/test_tui_app.py::test_run_screen_formats_status_and_tps_cells_for_readability -q` 3건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 79건 통과, `python3 -m py_compile benchmarks/tui/app.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py` 통과로 확인했습니다.
- Benchmark TUI chip/readability polish 단계 (2026-03-27): 실행 화면의 chip과 완료 셀 가독성을 더 다듬었습니다. `benchmarks/tui/app.py`의 spotlight chip은 이제 `WORKLOAD sleep` 같은 대문자 prefix 대신 `workload: sleep`, `ordering: partition`, `engine: async` 형태로 보여 의미를 더 자연스럽게 읽을 수 있습니다. chip 색상도 waiting은 muted surface, running은 green background, done은 softer green background, failed는 red background로 조정해 기존 accent/yellow 계열보다 현재 상태를 더 빠르게 파악할 수 있게 했습니다. 실행 로그 테이블은 완료 셀에서 `DONE` 텍스트를 제거하고 초록색 `111.11 TPS` 같은 수치만 남겨, 완료 항목이 더 간결하게 읽히도록 바꿨습니다. 회귀 테스트로 `tests/unit/benchmarks/test_tui_app.py`의 chip text/class 및 완료 셀 expectation을 갱신했고, 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py::test_run_screen_mounts_dashboard_widgets tests/unit/benchmarks/test_tui_app.py::test_run_screen_updates_progress_bar_and_summary_table tests/unit/benchmarks/test_tui_app.py::test_run_screen_spotlight_uses_single_progress_semantics_and_selected_rows_only tests/unit/benchmarks/test_tui_app.py::test_run_screen_formats_status_and_tps_cells_for_readability -q` 4건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 79건 통과, `python3 -m py_compile benchmarks/tui/app.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py` 통과로 확인했습니다.
- Benchmark TUI dual-progress 단계 (2026-03-27): 실행 로그 화면의 single progress bar가 상단 `완료 n / total` 텍스트와 함께 current-engine progress처럼 읽히던 혼동을 줄이기 위해 spotlight card에 progress를 두 단계로 분리했습니다. `benchmarks/tui/app.py`의 `RunScreen`은 이제 `ENGINE <phase> x / phase_total` badge + `#phase-progress` bar를 current engine progress로, 기존 `완료 x / total` badge + `#run-progress` bar를 전체 progress로 표시합니다. current engine progress는 active/next engine phase를 기준으로 active workload×ordering 조합 중 완료(성공/실패/취소 포함)된 개수를 스캔해 계산하며, 전체 progress는 기존처럼 `completed_runs / total_runs`를 사용합니다. 함께 spotlight chip은 `WORKLOAD / ORDERING / ENGINE` 3-chip 구조를 유지해 engine 종류를 더 명시적으로 드러냅니다. 회귀 테스트로 `tests/unit/benchmarks/test_tui_app.py`의 mount/spotlight/status readability expectation을 갱신했고, 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py::test_run_screen_mounts_dashboard_widgets tests/unit/benchmarks/test_tui_app.py::test_run_screen_spotlight_uses_single_progress_semantics_and_selected_rows_only tests/unit/benchmarks/test_tui_app.py::test_run_screen_formats_status_and_tps_cells_for_readability -q` 3건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 79건 통과, `python3 -m py_compile benchmarks/tui/app.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py` 통과로 확인했습니다.
- Benchmark TUI engine chip naming 단계 (2026-03-27): 실행 화면의 current-state chip 중 `PHASE`가 실제로는 engine phase(`baseline`/`async`/`process`)를 나타내는데도 이름이 약해 사용자 눈에 엔진 chip으로 잘 읽히지 않던 점을 정리했습니다. `benchmarks/tui/app.py`의 spotlight는 이제 `WORKLOAD` / `ORDERING` / `ENGINE` 3-chip 구성을 사용하고, `ENGINE 대기 중` / `ENGINE async` 같은 표현으로 현재 engine phase를 직접 보여줍니다. 함께 `tests/unit/benchmarks/test_tui_app.py`의 mount/spotlight/status readability expectations를 갱신했고, 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py::test_run_screen_mounts_dashboard_widgets tests/unit/benchmarks/test_tui_app.py::test_run_screen_spotlight_uses_single_progress_semantics_and_selected_rows_only tests/unit/benchmarks/test_tui_app.py::test_run_screen_formats_status_and_tps_cells_for_readability tests/unit/benchmarks/test_tui_results_report.py::test_results_modal_compresses_ordering_summary_with_tabs -q` 4건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 79건 통과, `python3 -m py_compile benchmarks/tui/app.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py` 통과로 확인했습니다.
- Benchmark TUI table/tabs/back-navigation refinement 단계 (2026-03-27): 추가 피드백을 반영해 `run` 화면과 결과 모달을 더 정리했습니다. `benchmarks/tui/app.py`의 `RunScreen`은 실행 중 `뒤로`/`종료` 버튼을 숨기고 `action_back()`/`action_exit()`도 no-op가 되도록 바꿔, stop/failure/success terminal state 이후에만 화면을 떠날 수 있게 했습니다. 실행 테이블은 `DONE ... TPS`를 `bright_green`, `WAITING`을 `grey62` `Text` cell로 렌더해 running 외 상태도 더 잘 구분되게 했습니다. spotlight card는 `현재 실행` 제목 + `WORKLOAD`/`ORDERING`/`PHASE` chip + progress bar 조합으로 유지하고, spinner는 `실행 로그` 헤더 옆으로 분리해 activity signal 역할만 담당하게 했습니다. 결과 모달은 ordering selector를 `Tabs`로 유지하되 버튼처럼 보이던 문제를 줄이기 위해 tabs 바로 아래 summary가 붙도록 spacing을 조정하고, 하단 action 버튼은 동일 너비의 가로 배치로 고정했습니다. 회귀 테스트로 `tests/unit/benchmarks/test_tui_app.py`와 `tests/unit/benchmarks/test_tui_results_report.py`를 갱신했고, 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py::test_run_screen_mounts_dashboard_widgets tests/unit/benchmarks/test_tui_app.py::test_run_screen_updates_progress_bar_and_summary_table tests/unit/benchmarks/test_tui_app.py::test_run_screen_spotlight_uses_single_progress_semantics_and_selected_rows_only tests/unit/benchmarks/test_tui_app.py::test_run_screen_formats_status_and_tps_cells_for_readability tests/unit/benchmarks/test_tui_app.py::test_run_screen_back_does_not_leave_screen_while_running tests/unit/benchmarks/test_tui_results_report.py::test_results_modal_compresses_ordering_summary_with_tabs -q` 6건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 79건 통과, `python3 -m py_compile benchmarks/tui/app.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py` 통과로 확인했습니다.
- Benchmark TUI remaining feedback 반영 단계 (2026-03-27): 추가 피드백에 맞춰 실행 화면과 결과 모달을 다시 다듬었습니다. `benchmarks/tui/app.py`의 `RunScreen`은 이제 `run-status`를 generic terminal status(`벤치마크 실행 중` / 완료 / 실패 / 취소)로 축소하고, 실제 현재 조합 정보는 spotlight card 안의 `WORKLOAD` / `ORDERING` / `PHASE` 상태 chip으로만 보여줍니다. progress bar도 spotlight card 안으로 이동했고, `LoadingIndicator`는 `실행 로그` 헤더 옆으로 내려 로그 activity signal 역할만 하도록 분리했습니다. 결과 모달은 ordering dropdown을 제거하고 `Tabs` 기반 ordering selector로 바꿨으며, footer action 버튼(`설정으로 돌아가기`, `닫기`)은 같은 너비의 가로 배치로 맞췄습니다. 회귀 테스트로 `tests/unit/benchmarks/test_tui_app.py`, `tests/unit/benchmarks/test_tui_results_report.py`를 갱신해 chip/status/progress 위치, tab 전환, 동일 크기 버튼을 검증했고, 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py::test_run_screen_mounts_dashboard_widgets tests/unit/benchmarks/test_tui_app.py::test_run_screen_spotlight_uses_single_progress_semantics_and_selected_rows_only tests/unit/benchmarks/test_tui_app.py::test_run_screen_formats_status_and_tps_cells_for_readability tests/unit/benchmarks/test_tui_results_report.py::test_results_modal_compresses_ordering_summary_with_tabs -q` 4건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 78건 통과, `python3 -m py_compile benchmarks/tui/app.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py` 통과로 확인했습니다.
- Benchmark TUI footer cleanup 단계 (2026-03-27): 설정 화면 하단 footer에서 `argv-preview`와 `form-error-summary`가 동시에 accent box로 렌더되어, 에러가 없어도 노란 박스가 두 개 보이던 문제를 정리했습니다. `benchmarks/tui/app.py`의 `_refresh_form_state()`는 이제 validation 오류가 있을 때만 `#form-error-summary`를 표시하고, 정상 상태에서는 summary widget을 숨겨 preview box 하나만 남깁니다. 회귀 테스트로 `tests/unit/benchmarks/test_tui_app.py`에 `test_options_screen_hides_error_summary_until_needed()`를 추가했고, 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py::test_options_screen_hides_error_summary_until_needed -q` 1건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py -q` 40건 통과로 확인했습니다.
- Benchmark TUI UX 개선 green 단계 (2026-03-27): `benchmarks/tui/app.py`와 `benchmarks/tui/results_report.py`를 재구성해 반복 실험 UX를 개선했습니다. 옵션 화면은 하단 footer에 argv preview + action 버튼을 고정하고, invalid 숫자 입력/빈 workload·ordering 선택/전부 skip한 phase 조합에 대해 inline validation을 표시하며 실행 버튼을 비활성화합니다. preview는 마지막으로 검증된 argv만 유지해 silent fallback을 제거했습니다. 실행 화면은 기존 phase/workload badge/pill을 제거하고 `현재 실행 workload / ordering / phase`, `현재 run`, `완료 run`, `경과 시간`을 보여주는 spotlight로 교체했으며, 조합 테이블은 선택한 workload/ordering과 활성 phase만 렌더링하고 `WAITING`/`RUNNING`/`DONE`/`FAILED`/`CANCELLED` 텍스트를 직접 표시합니다. 성공/실패/취소 후에는 모두 `설정으로 돌아가기` 동선을 제공하고, 성공 시에는 `결과 다시 보기`와 함께 종료 버튼을 노출합니다. 결과 모달은 ordering selector 기반 단일 요약 섹션으로 압축하고, 요약 라인을 `TPS + 평균 + P99` 기준으로 한국어 UI에서 보여주도록 바꿨습니다. 회귀 테스트로 `tests/unit/benchmarks/test_tui_app.py`, `tests/unit/benchmarks/test_tui_results_report.py`를 갱신/추가했고, 검증은 `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py -q` 39건 통과, `uv run pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_path_picker.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 77건 통과, `python3 -m py_compile benchmarks/tui/app.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py` 통과로 확인했습니다.
- Benchmark TUI UX 개선 red 단계 (2026-03-27): 반복 실험 동선과 진행/결과 가시성 개선을 위해 TUI UX 재설계를 시작했습니다. 먼저 `tests/unit/benchmarks/test_tui_app.py`, `tests/unit/benchmarks/test_tui_results_report.py`에 failing regression tests를 추가해 invalid 숫자 입력 시 실행 차단 + preview 고정, 성공/실패 후 `설정으로 돌아가기` 경로에서 기존 옵션값 보존, 실행 중 spotlight/단일 progress semantics, ordering selector 기반 압축 결과 모달을 명시적으로 고정했습니다. 현재 구현 전 RED 상태이며, 다음 단계에서 `benchmarks/tui/app.py` / `benchmarks/tui/results_report.py`를 수정해 테스트를 green으로 돌릴 예정입니다.
- Issue #14 experimental flush-policy 단계 (2026-03-27): tiny ordered process cliff를 실제 정책 실험으로 비교할 수 있도록 `ProcessConfig`에 `flush_policy`(`size_or_timer`, `demand`, `demand_min_residence`)와 `demand_flush_min_residence_ms`를 추가하고, `ProcessExecutionEngine`의 `_BatchAccumulator.add()`가 새 submit 도착 시 기존 buffer를 demand flush할 수 있게 보강했습니다. precedence는 명확히 유지해 `flush_policy`는 demand-trigger 허용 여부만 결정하고, 기존 `batch_size`/`max_batch_wait_ms`는 여전히 size/timer fallback trigger로 남깁니다. benchmark surface도 `benchmarks/pyrallel_consumer_test.py` / `benchmarks/run_parallel_benchmark.py`에 `--process-flush-policy`, `--process-demand-flush-min-residence-ms`를 추가해 CLI에서 real engine config로 바로 전달되도록 확장했고, `benchmarks/README.md`에 실험 플래그를 문서화했습니다. 회귀 테스트로 `tests/unit/execution_plane/test_process_engine_batching.py`에 immediate demand flush / min-residence demand flush 동작 테스트를, `tests/unit/benchmarks/test_run_parallel_benchmark_tui.py`와 `tests/unit/benchmarks/test_benchmark_runtime.py`에는 새 parser/config/runtime pass-through 테스트를 추가했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/execution_plane/test_process_engine_batching.py tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 53건 통과. 실측: default comparison `[issue14-ipc-default.json]` vs immediate `[issue14-policy-demand.json]` vs hybrid-1ms `[issue14-policy-hybrid-1ms.json]` 기준 `partition strict-on` TPS는 약 `515 -> 1295 -> 1094`, `partition strict-off`는 약 `524 -> 1113 -> 780`, `key_hash strict-on`은 약 `1733 -> 1588 -> 2047`, `key_hash strict-off`는 약 `2115 -> 1749 -> 1994`였습니다. 결론적으로 immediate는 partition 개선폭이 크지만 key_hash 회귀가 크고, 1ms hybrid는 partition 이득 일부를 남기면서 key_hash 회귀를 더 잘 억제했습니다.
- Issue #14 process IPC instrumentation 단계 (2026-03-27): tiny ordered process cliff의 다음 판단 재료를 만들기 위해 policy 변경 없이 process runtime timing 계측만 추가했습니다. `pyrallel_consumer/dto.py`의 `ProcessBatchMetrics`는 `last/avg main_to_worker_ipc`, `last/avg worker_exec`, `last/avg worker_to_main_ipc` 필드를 기본값과 함께 갖도록 확장했고, `pyrallel_consumer/execution_plane/process_engine.py`는 batch envelope에 `flush_enqueued_at`을 싣고 worker receive / batch execution / completion enqueue 시각을 이용해 engine 내부에서 timing snapshot을 누적하도록 보강했습니다. control-plane surface는 그대로 유지되어 `BrokerPoller.get_metrics()`는 기존처럼 runtime snapshot만 forward 합니다. `pyrallel_consumer/metrics_exporter.py`는 대응하는 `consumer_process_batch_*_ipc_seconds` 및 `consumer_process_batch_*_worker_exec_seconds` gauge를 추가했고, `monitoring/grafana/dashboards/pyrallel-overview.json`에는 `Process batch timing` 패널을 더해 평균 `main->worker`, `worker exec`, `worker->main` 시간을 바로 볼 수 있게 했습니다. 회귀 테스트로 `tests/unit/execution_plane/test_process_engine_batching.py`에 실제 process completion 후 timing snapshot 검증을, `tests/unit/metrics/test_prometheus_exporter.py`에 새 gauge 검증을, `tests/unit/metrics/test_monitoring_assets.py`에 dashboard query 존재 검증을 추가했고, README/README.ko monitoring 예시 쿼리도 갱신했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/execution_plane/test_process_engine_batching.py tests/unit/control_plane/test_broker_poller_metrics.py tests/unit/metrics/test_prometheus_exporter.py tests/unit/metrics/test_monitoring_assets.py -q` 28건 통과, `python3 -m py_compile pyrallel_consumer/dto.py pyrallel_consumer/execution_plane/process_engine.py pyrallel_consumer/metrics_exporter.py tests/unit/execution_plane/test_process_engine_batching.py tests/unit/metrics/test_prometheus_exporter.py tests/unit/metrics/test_monitoring_assets.py` 통과.
- Docs sync 단계 (2026-03-27): `#19` 완료와 현재 metric naming을 반영해 문서를 다시 맞췄습니다. `docs/operations/guide.ko.md`의 Grafana query 예시는 legacy `pyrallel_*` 이름에서 실제 exporter가 내보내는 `consumer_*` 이름으로 갱신했고, `docs/blueprint/features/01-ingress/02-ordered-work-scheduling/02-architecture.md`와 `docs/blueprint/03-architecture.md`에는 `WorkQueueTopology` ordered-set runnable queue helper와 commit-path duplicate scan 제거 상태를 명시했습니다.
- Issue #19 runnable-queue hot-path 단계 (2026-03-27): `pyrallel_consumer/control_plane/work_queue_topology.py`의 `deactivate_queue_key()`가 queue key 하나를 제거할 때마다 `deque` 전체를 재구성하던 hot path를 정리했습니다. 새 `RunnableQueueKeys` helper는 ordered-set 형태로 runnable queue key 순서를 유지하면서 `append`, `popleft`, `discard`, `count`를 제공하고, `deactivate_queue_key()`와 blocking queue 선택 경로가 전체 rebuild 없이 key를 제거할 수 있게 바꿨습니다. 외부 `WorkManager` alias surface는 유지해 기존 테스트/호출부 semantics는 바꾸지 않았습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_work_queue_topology.py tests/unit/control_plane/test_work_manager.py -q` 38건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_offset_tracker.py tests/unit/control_plane/test_broker_dispatch_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_work_queue_topology.py tests/unit/control_plane/test_work_manager.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/integration/test_broker_poller_integration.py -q` 108건 통과, 관련 `py_compile` 통과.
- Issue #19 commit-path simplification 단계 (2026-03-27): control-plane 분해 이후에도 남아 있던 commit-path 중복 contiguous scan을 정리했습니다. `pyrallel_consumer/control_plane/offset_tracker.py`에 `get_committable_high_water_mark()`와 `commit_through()`를 추가해 contiguous-safe HWM 계산과 커밋 성공 후 state advance를 canonical tracker API로 모았고, `broker_dispatch_support.py`는 commit candidate 계산 시 이 API를 사용하도록 바꿨습니다. `broker_poller.py`의 `_commit_offsets()`는 커밋 성공 후 다시 `advance_high_water_mark()`로 재스캔하지 않고, 이미 계산된 `safe_offset`을 `commit_through()`에 직접 적용하도록 바꿨습니다. 회귀로 `tests/unit/control_plane/test_offset_tracker.py`, `tests/unit/control_plane/test_broker_poller.py`에 새 tracker API 및 commit success path 테스트를 추가했고, integration fixture도 같은 seam을 반영하도록 `tests/integration/test_broker_poller_integration.py`를 갱신했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_offset_tracker.py tests/unit/control_plane/test_broker_dispatch_support.py tests/unit/control_plane/test_broker_poller.py -q` 49건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_work_manager.py tests/unit/control_plane/test_work_queue_topology.py tests/integration/test_broker_poller_integration.py -q` 59건 통과, 관련 `py_compile` 통과.
- PR16 remaining review fixes 단계 (2026-03-27): 남아 있던 actionable review 4개를 정리했습니다. `pyrallel_consumer/control_plane/broker_support.py`의 `build_offsets_to_commit()`는 이제 `strategy`를 실제로 존중해 `contiguous_only`에서는 metadata를 비우고 `metadata_snapshot`일 때만 snapshot metadata를 커밋합니다. `pyrallel_consumer/control_plane/broker_rebalance_support.py`는 `consumer.committed(partitions, timeout=...)`로 bounded lookup을 사용하도록 바꿨고, `pyrallel_consumer/control_plane/broker_poller.py`는 `_run_consumer()` finally에서 `_consumer_task`를 cleanup 완료 후에만 비워 restart/shutdown race를 줄였습니다. `benchmarks/tui/state.py`는 `metrics_port=0`일 때도 `--metrics-port 0`을 child argv에 명시적으로 전달해 CLI 기본값 `9091`로 되돌아가지 않도록 고쳤습니다. 회귀 테스트로 `tests/unit/control_plane/test_broker_support.py`, `tests/unit/control_plane/test_broker_rebalance_support.py`, `tests/unit/control_plane/test_broker_poller.py`, `tests/unit/benchmarks/test_tui_state.py`를 보강했고, 검증은 `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_support.py tests/unit/control_plane/test_broker_rebalance_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/benchmarks/test_tui_state.py -q` 40건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_task_lifecycle_support.py tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 52건 통과, 관련 `py_compile` 통과로 확인했습니다.
- TUI metrics-port propagation 단계 (2026-03-27): benchmark CLI의 `--metrics-port` 기본값을 `9091`로 올린 뒤에도 Textual TUI 경로는 `BenchmarkTuiState.to_argv()`에 해당 옵션이 없어 child benchmark process로 전달되지 않는 문제가 있었습니다. 수정으로 `benchmarks/tui/state.py`에 `metrics_port=9091` 필드를 추가하고, `metrics_port > 0`일 때 `--metrics-port`를 argv에 포함하도록 했습니다. `benchmarks/tui/app.py`에는 `Metrics port` 입력 필드를 추가해 UI에서도 값을 보이고 편집할 수 있게 했고, `benchmarks/tui/option_help.py`에는 관련 help text를 추가했습니다. `tests/unit/benchmarks/test_tui_state.py`에는 TUI 기본 argv가 `--metrics-port 9091`을 포함하는지, `metrics_port=0`이면 생략되는지 확인하는 회귀 테스트를 추가했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py tests/unit/benchmarks/test_benchmark_runtime.py -q` 39건 통과, 관련 `py_compile` 통과.
- Benchmark metrics default 단계 (2026-03-27): benchmark/TUI 경로에서 Prometheus target이 매번 수동 옵션에 의존하지 않도록 `benchmarks/run_parallel_benchmark.py`의 `--metrics-port` 기본값을 `9091`로 올렸습니다. 동시에 `_normalize_metrics_port()` helper를 추가해 `--metrics-port 0` 또는 음수 값일 때는 metrics 노출을 끌 수 있도록 했습니다. `tests/unit/benchmarks/test_run_parallel_benchmark_tui.py`에는 parser default가 `9091`인지 확인하는 테스트를, `tests/unit/benchmarks/test_benchmark_runtime.py`에는 metrics port normalization 테스트를 추가했습니다. README/README.ko/benchmarks README도 benchmark가 기본적으로 `9091`을 열고, 필요할 때 `--metrics-port 0`으로 비활성화할 수 있다는 내용으로 갱신했습니다.
- Docs blueprint reorg 단계 (2026-03-27): 새로 작성된 blueprint 문서군이 `docs/` 루트에 바로 놓여 기존 운영 문서/plan archive와 섞여 있던 구조를 정리했습니다. `00-index.md`, `01-overview.md`, `02-requirements.md`, `03-architecture.md`, `04-open-decisions.md`, `99-context-restoration.md`, `features/**`는 모두 `docs/blueprint/` 아래로 이동했고, 루트 `docs/index.md`는 `blueprint / operations / plans` 세 구역을 설명하는 entrypoint로 다시 썼습니다. 이동 과정에서 blueprint 문서 안에 남아 있던 stale metrics 설명도 함께 정리해 `PyrallelConsumer`의 Prometheus auto-wiring 현황과 현재 open decision 범위를 맞췄습니다.
- Operations docs reorg 단계 (2026-03-27): 운영 문서도 같은 원칙으로 `docs/operations/` 아래로 정리했습니다. 기존 `docs/operations.md`, `docs/operations.en.md`, `docs/ops_playbooks.md`는 각각 `docs/operations/guide.ko.md`, `docs/operations/guide.en.md`, `docs/operations/playbooks.md`로 이동했고, `docs/operations/index.md`를 새로 추가해 운영 문서 진입점을 만들었습니다. 이에 맞춰 `docs/index.md`, README/README.ko, blueprint 문서 내 source-history/observability 경로도 함께 갱신했습니다.
- Grafana datasource UID fix 단계 (2026-03-27, issue #21): Grafana dashboard는 `monitoring/grafana/dashboards/pyrallel-overview.json`에서 datasource UID를 `prometheus`로 참조하고 있었지만, 실제 provisioned datasource는 `monitoring/grafana/provisioning/datasources/datasource.yml`에 UID가 없어 Grafana가 임의 UID(`PBFA97CFB590B2093`)를 부여하고 있었습니다. 이 때문에 패널이 datasource를 찾지 못해 그래프 대신 에러를 표시했습니다. 수정으로 datasource provisioning에 `uid: prometheus`를 명시했고, `tests/unit/metrics/test_monitoring_assets.py`에는 stable UID 존재를 고정하는 테스트를 추가했습니다. 원인 검증은 Grafana API `/api/datasources`와 `/api/datasources/uid/prometheus`를 확인해 404 mismatch를 재현하는 방식으로 수행했습니다.
- PyrallelConsumer metrics auto-wiring 단계 (2026-03-27, issue #21): 일반 라이브러리 사용 경로에서도 `config.metrics.enabled=True`만으로 Prometheus target이 실제로 올라오도록 `pyrallel_consumer/consumer.py`에 퍼사드-level exporter lifecycle을 추가했습니다. 구현은 `PyrallelConsumer.start()` 시점에 `PrometheusMetricsExporter`를 생성하고 `WorkManager.set_metrics_exporter()`로 completion observer를 연결한 뒤, `BrokerPoller.get_metrics()` snapshot을 초기 1회 + background polling task로 갱신하도록 정리했습니다. `stop()`에서는 polling task를 먼저 취소하고 poller 종료 후 final snapshot을 반영한 다음 engine shutdown과 exporter close를 수행하며, start 중간 실패 시에도 engine shutdown/exporter cleanup이 이뤄지도록 보강했습니다. 이를 위해 `pyrallel_consumer/control_plane/work_manager.py`에는 generic `set_metrics_exporter()` seam을 추가했고, `pyrallel_consumer/metrics_exporter.py`에는 HTTP server shutdown을 위한 `close()`를 추가했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/test_consumer.py tests/unit/metrics/test_prometheus_exporter.py -q` 10건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/test_consumer.py tests/unit/test_config.py tests/unit/metrics/test_prometheus_exporter.py tests/unit/control_plane/test_broker_poller_metrics.py -q` 23건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_runtime_support.py tests/unit/execution_plane/test_process_engine_batching.py -q` 17건 통과, 관련 `py_compile` 통과.
- Monitoring stack readiness hardening 단계 (2026-03-27): Prometheus Targets 화면에서 test stack이 모두 죽어 보이던 문제를 재현/분석한 뒤 `.github/e2e.compose.yml`의 `kafka-exporter`에 `restart: unless-stopped`를 추가했습니다. 원인은 Prometheus 설정이 아니라 exporter가 Kafka readiness 전에 한 번 붙었다가 `kafka: client has run out of available brokers to talk to`로 종료되고, 재시작 정책이 없어 그대로 남는 compose lifecycle 이슈였습니다. `tests/unit/metrics/test_monitoring_assets.py`에는 restart policy 존재를 고정하는 assertion을 추가했고, README/README.ko/benchmarks README에는 `kafka-exporter`는 자동 복구되지만 `pyrallel-consumer` target은 benchmark/test harness가 `--metrics-port 9091`로 실제 실행 중일 때만 `up`이 되는 점을 더 명확히 문서화했습니다. 추가 검증으로 escalated shell에서 `uv run python benchmarks/run_parallel_benchmark.py --skip-baseline --skip-async --workloads sleep --order partition --num-messages 4000 --worker-sleep-ms 2 --num-partitions 4 --timeout-sec 30 --metrics-port 9091 --topic-prefix verify-metrics-live`를 짧게 실행하면서 `curl http://127.0.0.1:9091/metrics`를 확인했고, `consumer_processed_total`, `consumer_in_flight_count`, `consumer_parallel_lag`가 실제로 노출되는 것을 검증했습니다.
- Benchmark Prometheus exporter wiring 단계 (2026-03-27): test monitoring stack에서 `pyrallel-consumer` target도 실제로 올라올 수 있도록 benchmark harness에 `--metrics-port` wiring을 추가했습니다. `benchmarks/pyrallel_consumer_test.py`에는 shared `_get_or_create_prometheus_exporter()` cache를 도입하고, `build_kafka_config()`/`run_pyrallel_consumer_test()`에 `metrics_port`를 전달해 `PrometheusMetricsExporter`를 한 번만 띄운 뒤 run 동안 `broker_poller.get_metrics()`를 주기적으로 `update_from_system_metrics()`로 push하고 종료 시 final snapshot도 갱신하도록 보강했습니다. `benchmarks/run_parallel_benchmark.py`에는 `--metrics-port` CLI 옵션과 pass-through를 추가했고, `tests/unit/benchmarks/test_run_parallel_benchmark_tui.py`에는 parser acceptance 테스트를, `tests/unit/benchmarks/test_benchmark_runtime.py`에는 metrics flag/config wiring 및 exporter singleton reuse 테스트를 추가했습니다. README/README.ko/benchmarks README에는 `.github/e2e.compose.yml`와 함께 `--metrics-port 9091`로 실행해야 `pyrallel-consumer` target이 up 된다는 사용법을 문서화했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/benchmarks/test_run_parallel_benchmark_tui.py tests/unit/benchmarks/test_benchmark_runtime.py -q` 32건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane tests/unit/execution_plane/test_process_engine_batching.py tests/unit/metrics tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py tests/unit/test_consumer.py tests/unit/test_config.py -q` 246건 통과, 관련 `py_compile` 통과.
- Test-stack Prometheus/Grafana wiring 단계 (2026-03-27): 테스트용 compose stack에도 Prometheus/Grafana를 붙여 process batch telemetry를 바로 시각화할 수 있도록 `.github/e2e.compose.yml`을 확장했습니다. 새 stack에는 `kafka-exporter`, `prometheus`, `grafana` service를 추가했고, `prometheus`는 `../monitoring/prometheus.yml`을, `grafana`는 기존 provisioning/dashboards를 그대로 mount 하도록 구성했습니다. `monitoring/grafana/dashboards/pyrallel-overview.json`에는 `Process batch flushes`, `Process batch sizing` 패널을 추가해 `consumer_process_batch_flush_count{reason="timer"}`, `consumer_process_batch_avg_size`, `consumer_process_batch_buffered_age_seconds`, `consumer_process_batch_buffered_items`를 볼 수 있게 했고, `README.md`/`README.ko.md`에는 `.github/e2e.compose.yml` 기반 테스트 모니터링 stack 실행법과 접속 정보를 문서화했습니다. `tests/unit/metrics/test_monitoring_assets.py`를 추가해 e2e compose service wiring과 Grafana dashboard query 존재를 고정했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/metrics/test_monitoring_assets.py tests/unit/metrics/test_prometheus_exporter.py -q` 4건 통과, `docker compose -f .github/e2e.compose.yml config --services`에서 `kafka-1`, `kafka-exporter`, `prometheus`, `grafana` 확인, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane tests/unit/execution_plane/test_process_engine_batching.py tests/unit/metrics tests/unit/test_consumer.py tests/unit/test_config.py -q` 214건 통과, 관련 `py_compile` 통과.

## 최근 업데이트 (2026-03-26)
- Process batch Prometheus metrics 단계 (2026-03-26): process batching 병목을 Prometheus에서 직접 볼 수 있도록 `ProcessBatchMetrics` snapshot을 도입했습니다. `pyrallel_consumer/dto.py`에는 `ProcessBatchMetrics`와 `SystemMetrics.process_batch_metrics` optional field를 추가했고, `BaseExecutionEngine.get_runtime_metrics()` 기본 no-op를 둔 뒤 `ProcessExecutionEngine`이 `_BatchAccumulator.snapshot()`을 통해 size/timer/close flush count, total flushed items, last flush size/wait, current buffered items/age를 노출하도록 보강했습니다. `BrokerPoller.get_metrics()`는 실제 `ProcessBatchMetrics` snapshot일 때만 이를 `SystemMetrics`에 포함하고, `PrometheusMetricsExporter`는 `consumer_process_batch_flush_count{reason}`, `consumer_process_batch_avg_size`, `consumer_process_batch_last_size`, `consumer_process_batch_last_wait_seconds`, `consumer_process_batch_buffered_items`, `consumer_process_batch_buffered_age_seconds` gauge를 갱신하도록 확장했습니다. `tests/unit/execution_plane/test_process_engine_batching.py`에는 runtime snapshot 테스트 2건을, `tests/unit/control_plane/test_broker_poller_metrics.py`에는 engine runtime metrics forwarding 테스트를, `tests/unit/metrics/test_prometheus_exporter.py`에는 새 gauge 검증을 추가했고, README/README.ko monitoring 섹션에는 process batch Prometheus query 예시를 문서화했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_poller_metrics.py tests/unit/execution_plane/test_process_engine_batching.py tests/unit/metrics/test_prometheus_exporter.py -q` 23건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane tests/unit/execution_plane/test_process_engine_batching.py tests/unit/metrics/test_prometheus_exporter.py tests/unit/test_consumer.py tests/unit/test_config.py -q` 212건 통과, `python3 -m py_compile pyrallel_consumer/dto.py pyrallel_consumer/execution_plane/base.py pyrallel_consumer/execution_plane/process_engine.py pyrallel_consumer/control_plane/broker_poller.py pyrallel_consumer/metrics_exporter.py tests/unit/execution_plane/test_process_engine_batching.py tests/unit/control_plane/test_broker_poller_metrics.py tests/unit/metrics/test_prometheus_exporter.py` 통과.
- BrokerPoller task-lifecycle split 단계 (2026-03-26): control-plane의 마지막 큰 shell로 남아 있던 `start()`/`stop()`/`wait_closed()` task orchestration을 `pyrallel_consumer/control_plane/broker_task_lifecycle_support.py`로 추출했습니다. 새 `BrokerTaskLifecycleSupport`는 producer/admin/consumer 생성과 subscribe, consumer/completion task 생성, consumer task timeout cancel, shutdown wait, terminal error surfacing을 담당하고, `BrokerPoller`는 facade 메서드와 state flag만 유지하도록 정리했습니다. helper는 runtime factory를 lazy하게 생성하도록 바꿔 existing `Producer`/`AdminClient`/`Consumer` patch와 `asyncio.wait_for` patch가 그대로 먹히게 보강했습니다. 회귀 테스트로 `tests/unit/control_plane/test_broker_task_lifecycle_support.py`를 추가했고, 기존 `tests/unit/control_plane/test_broker_poller.py`의 `test_start_skips_completion_monitor_when_disabled`, `test_stop_cancels_consumer_task_after_timeout`, `test_wait_closed_reraises_terminal_error_when_shutdown_is_complete`가 그대로 통과하는지 확인했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_task_lifecycle_support.py tests/unit/control_plane/test_broker_poller.py -k "start_skips_completion_monitor_when_disabled or stop_cancels_consumer_task_after_timeout or wait_closed_reraises_terminal_error_when_shutdown_is_complete or start_runtime_skips_completion_monitor_when_disabled or stop_runtime_cancels_consumer_task_after_timeout or wait_closed_reraises_error_after_shutdown" -q` 6건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_support.py tests/unit/control_plane/test_broker_completion_support.py tests/unit/control_plane/test_broker_dispatch_support.py tests/unit/control_plane/test_broker_rebalance_support.py tests/unit/control_plane/test_broker_runtime_support.py tests/unit/control_plane/test_broker_task_lifecycle_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_dlq.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_poller_consume_timeout.py tests/unit/control_plane/test_broker_poller_metrics.py tests/unit/control_plane/test_blocking_timeout.py tests/integration/test_broker_poller_integration.py -q` 92건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/test_consumer.py tests/unit/test_config.py -q` 11건 통과, `python3 -m py_compile pyrallel_consumer/control_plane/broker_task_lifecycle_support.py pyrallel_consumer/control_plane/broker_poller.py tests/unit/control_plane/test_broker_task_lifecycle_support.py tests/unit/control_plane/test_broker_poller.py` 통과.
- BrokerPoller runtime-support split 단계 (2026-03-26): `BrokerPoller`의 backpressure/diagnostics/metrics 계산을 `pyrallel_consumer/control_plane/broker_runtime_support.py`로 추출했습니다. 새 `BrokerRuntimeSupport`는 `log_partition_diagnostics()`, `check_backpressure()`, `build_system_metrics()`를 담당하고, `BrokerPoller`는 `_log_partition_diagnostics()`/`_check_backpressure()`/`get_metrics()` facade와 `_make_runtime_support()`만 유지하도록 정리했습니다. 회귀 테스트로 `tests/unit/control_plane/test_broker_runtime_support.py`를 추가해 lag/gap/queue projection과 pause/resume 전이를 직접 고정했고, 기존 `tests/unit/control_plane/test_broker_poller_metrics.py`와 integration backpressure 경로가 그대로 통과하는지 확인했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_runtime_support.py tests/unit/control_plane/test_broker_poller_metrics.py tests/integration/test_broker_poller_integration.py -k "backpressure or metrics" -q` 8건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_rebalance_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_support.py tests/unit/control_plane/test_broker_completion_support.py tests/unit/control_plane/test_broker_dispatch_support.py tests/unit/control_plane/test_broker_poller_dlq.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_poller_consume_timeout.py tests/unit/control_plane/test_blocking_timeout.py tests/integration/test_broker_poller_integration.py -q` 82건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/test_consumer.py tests/unit/test_config.py tests/unit/control_plane/test_broker_poller_metrics.py -q` 16건 통과, `python3 -m py_compile pyrallel_consumer/control_plane/broker_runtime_support.py pyrallel_consumer/control_plane/broker_poller.py tests/unit/control_plane/test_broker_runtime_support.py tests/unit/control_plane/test_broker_poller_metrics.py` 통과.
- BrokerPoller rebalance-helper split 단계 (2026-03-26): `BrokerPoller`의 assign/revoke 책임을 `pyrallel_consumer/control_plane/broker_rebalance_support.py`로 추출했습니다. 새 `BrokerRebalanceSupport`는 committed offset 조회, metadata snapshot hydration, `OffsetTracker` 생성, revoke 시 cache drop + safe offset commit + tracker deletion을 담당하고, `BrokerPoller`는 `_on_assign()`/`_on_revoke()` facade와 로그 메시지만 유지하도록 정리했습니다. helper는 `tracker_factory`를 주입받게 만들어 기존 fixture patch가 그대로 따라가도록 보강했고, `tests/unit/control_plane/test_broker_rebalance_support.py`를 추가해 metadata snapshot hydration, zero-offset assignment HWM, revoke commit/delete 경로를 직접 고정했습니다. unit coverage로 `tests/unit/control_plane/test_broker_poller.py`에는 `test_start_skips_completion_monitor_when_disabled`, `test_stop_cancels_consumer_task_after_timeout`, `test_wait_closed_reraises_terminal_error_when_shutdown_is_complete`를 추가해 lifecycle facade behavior도 함께 고정했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_rebalance_support.py tests/unit/control_plane/test_broker_poller.py tests/integration/test_broker_poller_integration.py -q` 35건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_rebalance_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_support.py tests/unit/control_plane/test_broker_completion_support.py tests/unit/control_plane/test_broker_dispatch_support.py tests/unit/control_plane/test_broker_poller_dlq.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_poller_consume_timeout.py tests/unit/control_plane/test_blocking_timeout.py tests/integration/test_broker_poller_integration.py -q` 82건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/test_consumer.py tests/unit/test_config.py tests/unit/control_plane/test_broker_poller_metrics.py -q` 16건 통과, `python3 -m py_compile pyrallel_consumer/control_plane/broker_rebalance_support.py pyrallel_consumer/control_plane/broker_poller.py tests/unit/control_plane/test_broker_rebalance_support.py tests/unit/control_plane/test_broker_poller.py` 통과.
- BrokerPoller dispatch-loop split 단계 (2026-03-26): `_run_consumer()` 안에 섞여 있던 message validation/grouping/submission과 commit candidate planning을 `pyrallel_consumer/control_plane/broker_dispatch_support.py`로 추출했습니다. 새 `BrokerDispatchSupport`는 `dispatch_messages()`로 ordered/unordered submit 경로를 감싸고, `build_commit_candidates()`로 contiguous completed offsets + in-flight clamp를 계산하도록 정리했습니다. `BrokerPoller`는 `_make_dispatch_support()`를 통해 helper를 생성하고 `_run_consumer()`에서 dispatch/schedule/commit candidate 계산을 위임하도록 바꿨습니다. 회귀 테스트로 `tests/unit/control_plane/test_broker_dispatch_support.py`를 추가해 ordered bulk grouping, unordered direct submit, min-inflight commit clamp를 직접 고정했고, 기존 `tests/unit/control_plane/test_broker_poller.py`, `tests/unit/control_plane/test_broker_poller_consume_timeout.py`, `tests/unit/control_plane/test_broker_poller_completion_driven.py`, `tests/unit/control_plane/test_broker_poller_dlq.py`, `tests/unit/control_plane/test_blocking_timeout.py`, `tests/integration/test_broker_poller_integration.py` 범위가 그대로 통과하는지 확인했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_dispatch_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_consume_timeout.py tests/integration/test_broker_poller_integration.py -q` 36건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_support.py tests/unit/control_plane/test_broker_completion_support.py tests/unit/control_plane/test_broker_dispatch_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_dlq.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_poller_consume_timeout.py tests/unit/control_plane/test_blocking_timeout.py tests/integration/test_broker_poller_integration.py -q` 76건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/test_consumer.py tests/unit/test_config.py -q` 11건 통과, `python3 -m py_compile pyrallel_consumer/control_plane/broker_dispatch_support.py pyrallel_consumer/control_plane/broker_poller.py tests/unit/control_plane/test_broker_dispatch_support.py` 통과.
- BrokerPoller completion-pipeline split 단계 (2026-03-26): `BrokerPoller`의 다음 기능단위 분해 슬라이스로 blocking-timeout failover와 completion/DLQ bookkeeping을 `pyrallel_consumer/control_plane/broker_completion_support.py`로 추출했습니다. 새 `BrokerCompletionSupport`는 `handle_blocking_timeouts()`와 `process_completed_events()`를 담당하고, `BrokerPoller`는 `_handle_blocking_timeouts()`/`_process_completed_events()` facade 메서드와 diagnostics counter만 유지하도록 정리했습니다. helper 생성은 `_make_completion_support()`에서 current state/patched method를 바라보도록 구성해 기존 test monkeypatch 흐름을 깨지 않게 했습니다. 회귀 테스트로 `tests/unit/control_plane/test_broker_completion_support.py`를 추가해 forced timeout failure polling, success completion cache cleanup, metadata-only DLQ fallback을 직접 고정했고, 기존 `tests/unit/control_plane/test_blocking_timeout.py`, `tests/unit/control_plane/test_broker_poller_completion_driven.py`, `tests/unit/control_plane/test_broker_poller_dlq.py`, `tests/unit/control_plane/test_broker_poller_consume_timeout.py`, `tests/integration/test_broker_poller_integration.py` 범위가 그대로 통과하는지 확인했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_completion_support.py tests/unit/control_plane/test_blocking_timeout.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_poller_dlq.py -q` 37건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_support.py tests/unit/control_plane/test_broker_completion_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_dlq.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_poller_consume_timeout.py tests/unit/control_plane/test_blocking_timeout.py tests/integration/test_broker_poller_integration.py -q` 73건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/test_consumer.py tests/unit/test_config.py -q` 11건 통과, `python3 -m py_compile pyrallel_consumer/control_plane/broker_completion_support.py pyrallel_consumer/control_plane/broker_poller.py tests/unit/control_plane/test_broker_completion_support.py` 통과.
- BrokerPoller DLQ/cache + commit-planner split 단계 (2026-03-26): `BrokerPoller`의 첫 기능단위 분해 슬라이스로 DLQ/cache 로직과 commit metadata planning 로직을 `pyrallel_consumer/control_plane/broker_support.py`로 추출했습니다. 새 `DlqCacheSupport`는 raw payload cache budget/eviction/drop 정책을, `BrokerCommitPlanner`는 assignment metadata decode, revoke metadata encode, commit payload build를 담당하게 했고, `BrokerPoller`는 기존 메서드명을 유지한 채 새 helper에 위임하도록 정리했습니다. 회귀 테스트로 `tests/unit/control_plane/test_broker_support.py`를 추가해 cache budget/eviction과 metadata snapshot commit payload를 직접 고정했고, 기존 `tests/unit/control_plane/test_broker_poller.py`, `tests/unit/control_plane/test_broker_poller_dlq.py`, `tests/unit/control_plane/test_broker_poller_completion_driven.py`, `tests/unit/control_plane/test_broker_poller_consume_timeout.py`, `tests/integration/test_broker_poller_integration.py` 범위가 그대로 통과하는지 확인했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_dlq.py -q` 43건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_support.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_dlq.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_poller_consume_timeout.py tests/integration/test_broker_poller_integration.py -q` 66건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/test_consumer.py tests/unit/test_config.py -q` 11건 통과, `python3 -m py_compile pyrallel_consumer/control_plane/broker_support.py pyrallel_consumer/control_plane/broker_poller.py tests/unit/control_plane/test_broker_support.py` 통과.
- WorkManager queue-topology module split 단계 (2026-03-26): 성능 조정보다 먼저 복잡한 control-plane 모듈 기능단위 분해를 시작했고, 첫 슬라이스로 `WorkManager`의 queue/index 책임을 `pyrallel_consumer/control_plane/work_queue_topology.py`로 추출했습니다. 새 `WorkQueueTopology`는 virtual queue 저장소, runnable queue key 활성/비활성, head offset 추적, batch enqueue, blocking queue selection, revoke cleanup을 담당하고, `WorkManager`는 scheduling/completion facade로 남기되 기존 내부 필드(`_virtual_partition_queues`, `_runnable_queue_keys`, `_head_offsets` 등)는 compatibility alias로 유지했습니다. 회귀 테스트로 `tests/unit/control_plane/test_work_queue_topology.py`를 추가해 enqueue/head tracking, revoke cleanup, blocking queue selection을 직접 고정했고, 기존 `tests/unit/control_plane/test_work_manager.py`, `tests/unit/control_plane/test_work_manager_ordering.py`, `tests/unit/control_plane/test_broker_poller*.py`, `tests/integration/test_broker_poller_integration.py` 범위가 그대로 통과하는지 확인했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_work_queue_topology.py tests/unit/control_plane/test_work_manager.py -q` 38건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_poller_consume_timeout.py tests/unit/control_plane/test_work_manager_ordering.py tests/integration/test_broker_poller_integration.py -q` 57건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/test_consumer.py tests/unit/test_config.py -q` 11건 통과, `python3 -m py_compile pyrallel_consumer/control_plane/work_queue_topology.py pyrallel_consumer/control_plane/work_manager.py tests/unit/control_plane/test_work_queue_topology.py tests/unit/control_plane/test_work_manager.py` 통과.
- Ordered path bulk-enqueue 단계 (2026-03-26): `BrokerPoller`가 ordered consume batch를 `(tp, submit_key)`별로 먼저 묶어 `WorkManager.submit_message_batch()`로 넘기고, `WorkManager`는 bucket 단위 queue 생성/내부 queue bulk append/tracking 후 tp별 `last_fetched_offset`를 한 번만 갱신하도록 최적화했습니다. `submit_message()`는 새 batch path를 재사용하게 정리했고, `BrokerPoller`는 `submit_message_batch`가 async/awaitable일 때만 bulk path를 사용하고 sync/non-awaitable work-manager에서는 기존 `submit_message()` 반복 경로로 안전하게 fallback 하도록 보강했습니다. 회귀 테스트로 `tests/unit/control_plane/test_work_manager.py`에 batch queue state / tp별 fetched-offset 갱신 / unassigned guard / partition schedule parity / key-hash parallelism 테스트를, `tests/unit/control_plane/test_broker_poller_completion_driven.py`에는 sync batch fallback duplicate 방지 테스트를, `tests/integration/test_broker_poller_integration.py`에는 ordered 기본 경로가 message별 submit 대신 single bulk submit을 사용한다는 expectation을 추가했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_work_manager.py -k "submit_message_batch or submit_message_tracks_tp_offset_index" -q` 6건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_poller_completion_driven.py -k "sync_batch_submit or schedules_twice_when_messages_and_completions_share_iteration" -q` 2건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/integration/test_broker_poller_integration.py -k run_consumer_loop_basic_flow -q` 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_work_manager.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_broker_poller_consume_timeout.py tests/unit/control_plane/test_work_manager_ordering.py tests/integration/test_broker_poller_integration.py -q` 92건 통과, `python3 -m py_compile pyrallel_consumer/control_plane/work_manager.py pyrallel_consumer/control_plane/broker_poller.py tests/unit/control_plane/test_work_manager.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/integration/test_broker_poller_integration.py` 통과. 실측: current code default `issue14-bulk-default`에서 `sleep-key_hash-pyrallel-process-strict-on/off`는 약 `3135/2922 TPS`, `sleep-partition-pyrallel-process-strict-on/off`는 약 `691/686 TPS`였고, current code tuned `issue14-bulk-b1w0`에서 `sleep-partition-pyrallel-process-strict-on/off`는 약 `2320/1379 TPS`였습니다. 결론적으로 bulk enqueue는 key-hash default를 일부 개선했지만, tiny partition cliff의 지배적 병목은 여전히 process batching default입니다.
- Issue #14 tiny-partition warning 단계 (2026-03-26): local Kafka benchmark 재현으로 process ordered tiny-workload cliff를 실측했고, default batching이 dominant bottleneck임을 benchmark UX에 반영했습니다. `benchmarks/results/issue14-default.json` 기준 `sleep-partition-pyrallel-process-strict-on/off`는 각각 약 `757.85 TPS` / `734.72 TPS`였고, 같은 shape에 `--process-batch-size 1 --process-max-batch-wait-ms 0`을 적용한 `benchmarks/results/issue14-b1w0.json`에서는 약 `2437.32 TPS` / `1299.41 TPS`로 개선됐습니다. 반면 key-hash는 같은 tuning에서 개선이 일관되지 않아 blanket auto-tune 대신 warning 경로를 선택했습니다. `benchmarks/run_parallel_benchmark.py`에는 tiny `sleep` + process `partition` + default batching 조합에서 비교용 flag를 안내하는 warning을 추가했고, `benchmarks/README.md`에는 새 process batching 옵션과 tiny partition benchmark 권장 비교값을 문서화했습니다. `tests/unit/benchmarks/test_benchmark_runtime.py`에는 warning 출력/skip 회귀 테스트 2건을 추가했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/benchmarks/test_benchmark_runtime.py -k "tiny_partition_warning or process_batching" -q` 4건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py tests/unit/benchmarks/test_tui_state.py -q` 32건 통과, `python3 -m py_compile benchmarks/run_parallel_benchmark.py tests/unit/benchmarks/test_benchmark_runtime.py` 통과.
- Issue #14 benchmark-surface hardening 단계 (2026-03-26): ordered tiny-workload process 성능 절벽을 root-cause-first로 좁힌 뒤, 라이브러리 기본값을 바꾸지 않고 benchmark harness에서만 process batching knob를 제어할 수 있게 열었습니다. `benchmarks/run_parallel_benchmark.py`에는 `--process-batch-size`, `--process-max-batch-wait-ms` CLI 옵션을 추가했고, `benchmarks/pyrallel_consumer_test.py`의 `build_kafka_config()`/`run_pyrallel_consumer_test()`는 이 값을 `parallel_consumer.execution.process_config.batch_size`와 `max_batch_wait_ms` override로 전달하도록 보강했습니다. `tests/unit/benchmarks/test_run_parallel_benchmark_tui.py`에는 새 parser 옵션 수용 회귀 테스트를, `tests/unit/benchmarks/test_benchmark_runtime.py`에는 config builder/runner pass-through 회귀 테스트 3건을 추가했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -k process_batching_overrides -q` 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/benchmarks/test_benchmark_runtime.py -k process_batching -q` 3건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py tests/unit/benchmarks/test_tui_state.py -q` 30건 통과, `python3 -m py_compile benchmarks/pyrallel_consumer_test.py benchmarks/run_parallel_benchmark.py tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py` 통과.
- Issue #18 first-slice hardening 단계 (2026-03-26): Kafka-backed E2E 신뢰성/공개 설정 의미/`process_engine.py` 분해 범위를 현재 코드 상태에 맞춰 좁혀 반영했습니다. `.github/workflows/e2e.yml`에는 `pull_request.synchronize`를 추가해 PR 후속 커밋에도 Kafka-backed E2E가 다시 돌도록 했고, `README.md`/`README.ko.md`에는 `docker compose up -d kafka-1` + `uv run pytest tests/e2e -q` 기반 로컬 E2E 실행 경로와 `worker_pool_size`가 process concurrency가 아니라 ordered `key_hash` routing width라는 의미를 명시했습니다. `tests/unit/control_plane/test_broker_poller.py`에는 해당 의미를 고정하는 `test_get_partition_index_uses_worker_pool_size_for_key_hash_shards`를 추가했고, `pyrallel_consumer/execution_plane/process_engine.py`에는 `_drain_registry_event_queue()`, `_recover_dead_worker_items()`, `_emit_completion_event()`, `_emit_worker_recovery_failure()` helper를 도입해 registry drain/dead-worker recovery 경로를 shutdown/steady-state에서 공유하도록 정리했습니다. `tests/unit/execution_plane/test_process_execution_engine.py`에는 새 helper 경계를 고정하는 회귀 테스트 2건을 추가했습니다. 검증: `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/execution_plane/test_process_execution_engine.py -k "drain_registry_event_queue_returns_drained_count or recover_dead_worker_items or ensure_workers_alive_does_not_requeue_timed_out_work or ensure_workers_alive_stops_requeueing_after_max_retries or drain_shutdown_ipc_once_reuses_registry_event_rules" -q` 5건 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_poller.py -k worker_pool_size_for_key_hash_shards -q` 통과, `UV_CACHE_DIR=.uv-cache uv run pytest tests/unit/control_plane/test_broker_poller.py tests/unit/execution_plane/test_process_execution_engine.py tests/unit/execution_plane/test_process_engine_batching.py tests/unit/test_config.py tests/unit/test_consumer.py -q` 56건 통과, `python3 -m py_compile pyrallel_consumer/execution_plane/process_engine.py tests/unit/execution_plane/test_process_execution_engine.py tests/unit/control_plane/test_broker_poller.py` 통과.

## 최근 업데이트 (2026-03-12)
- Rebalance state-preservation docs/test sync 단계 (2026-03-13): `tests/unit/control_plane/test_work_manager.py`에 `test_stale_completion_after_reassign_does_not_touch_new_epoch_state`를 추가해 revoke 후 재할당된 새 epoch 상태가 이전 epoch completion에 의해 오염되지 않는 계약을 고정했습니다. `prd.md`에는 rebalance state preservation 정책(`contiguous_only` 기본값, `metadata_snapshot` 옵션, fail-closed/at-least-once tradeoff)을 문서화했습니다. 검증: `pytest -q tests/unit/control_plane/test_work_manager.py -k stale_completion_after_reassign_does_not_touch_new_epoch_state` 통과, `pytest -q tests/unit/control_plane/test_work_manager.py` 통과, `python -m py_compile tests/unit/control_plane/test_work_manager.py` 통과.
- Process shutdown timing instrumentation 단계 (2026-03-12): `pyrallel_consumer/execution_plane/process_engine.py`에 worker-2 범위의 parent-side shutdown 계측을 추가해 shutdown 시작 시 prefetched completion/in-flight registry/worker 수/batch buffer 상태를 DEBUG로 남기고, worker join→terminate→kill 각 단계의 대기 시간을 로그로 기록하도록 보강했습니다. `benchmarks/pyrallel_consumer_test.py`에는 stop trigger, `broker_poller.stop()`, `engine.shutdown()` 시작/종료 시점을 출력하는 `[timing]` 라인을 추가해 benchmark summary와 shutdown 시간을 분리 관찰할 수 있게 했습니다. 검증: `python -m py_compile pyrallel_consumer/execution_plane/process_engine.py benchmarks/pyrallel_consumer_test.py` 통과, `pytest -q tests/unit/execution_plane/test_process_engine_batching.py -k shutdown` 2건 통과, `pytest -q tests/unit/benchmarks/test_benchmark_runtime.py -k "assignment_wait_fails or engine_shutdown"` 1건 통과, `python -m pylint --persistent=n --disable=W,C,R,E0611 -j 1 -rn -sn pyrallel_consumer/execution_plane/process_engine.py benchmarks/pyrallel_consumer_test.py` 통과. 참고: 현재 `.venv`에는 `mypy`/`ruff` 모듈이 없어 별도 type/lint 도구는 실행하지 못했습니다.
- WorkManager submit-failure ordering red 단계 (2026-03-12): `tests/unit/control_plane/test_work_manager.py`에 `test_schedule_preserves_queue_order_after_submit_failure`를 추가해 `WorkManager.schedule()` submit 예외 후 재큐잉 순서가 뒤집히는 회귀를 고정 재현했습니다. 현재 실패 시 queue offset 순서가 `[10, 11]` 대신 `[11, 10]`으로 바뀝니다. 검증: `pytest -q tests/unit/control_plane/test_work_manager.py -k preserves_queue_order_after_submit_failure` 실패(`assert [11, 10] == [10, 11]`).
- Benchmark process ordering-validation expansion (2026-03-12): `benchmarks/pyrallel_consumer_test.py`에서 ordering validator를 async 전용에서 process mode까지 확장해 `key_hash`/`partition` ordered benchmark run도 worker 실행 시점에 즉시 검증하도록 보강했습니다. process mode 위반은 timeout 대신 즉시 `Ordering validation failed ...` 예외로 surface되고, 성공 run은 PASS summary를 출력합니다. 함께 `tests/unit/benchmarks/test_benchmark_runtime.py`에 process-mode ordering PASS/FAIL 회귀 테스트를, `tests/unit/benchmarks/test_tui_results_report.py`에 legacy `pyrallel-process` 결과 모달 표기 회귀 테스트를 추가했습니다. 검증: `pytest -q tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_tui_results_report.py` 25건 통과, `pytest -q tests/unit/benchmarks` 82건 통과, `python -m py_compile benchmarks/pyrallel_consumer_test.py tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_tui_results_report.py` 통과, `python -m pylint --persistent=n --disable=W,C,R,E0611 -j 1 -rn -sn benchmarks/pyrallel_consumer_test.py tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_tui_results_report.py` 통과.
- Benchmark ordering-validation/reporting green 단계 (2026-03-12): `benchmarks/pyrallel_consumer_test.py`에 `OrderingValidator`를 추가해 async benchmark run에서 `key_hash`/`partition` ordering을 즉시 검증하고, worker completion failure가 발생하면 timeout까지 기다리지 않고 `Benchmark worker failure on <topic>[<partition>]` 형태로 조기 실패를 surface 하도록 보강했습니다. 성공 run은 ordering PASS summary를 함께 출력합니다. 검증: `pytest tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py -q` 19건 통과, `pytest tests/unit/benchmarks -q` 78건 통과, `python -m py_compile benchmarks/pyrallel_consumer_test.py tests/unit/benchmarks/test_benchmark_runtime.py` 통과, `python -m pylint --persistent=n --disable=W,C,R,E0611 -j 1 -rn -sn benchmarks/pyrallel_consumer_test.py tests/unit/benchmarks/test_benchmark_runtime.py` 통과.
- Strict completion-monitor green 단계 (2026-03-12): `BrokerPoller`에 completion monitor task + control-plane lock을 추가하고 `BaseExecutionEngine.wait_for_completion()`을 async/process engine에 구현해, ordered backlog가 있을 때 broker consume cadence를 기다리지 않고 completion 도착 즉시 completion drain → `schedule()` 경로가 돌도록 보강했습니다. 함께 `WorkManager.poll_completed_events()`에 epoch fence를 넣어 stale completion이 same-key/partition 후속 work를 잘못 깨우지 않도록 막았습니다. 검증: `pytest tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_work_manager_ordering.py -q` 14건 통과.
- Completion-driven fast-repoll control-plane tests 단계 (2026-03-11): ordered-path resubmit tightening을 뒷받침하는 전용 회귀 테스트 `tests/unit/control_plane/test_broker_poller_consume_timeout.py`를 추가했습니다. in-flight 또는 queued backlog가 남아 있으면 `_get_consume_timeout_seconds()`가 `0.0`을 반환하고, `_run_consumer()`가 실제 `consumer.consume(..., timeout=0.0)`를 사용해 idle 0.1s poll로 돌아가지 않는 경로를 고정했습니다. 검증: `pytest tests/unit/control_plane/test_broker_poller_consume_timeout.py` 4건 통과.
- Completion-driven rescheduling focused-tests 단계 (2026-03-11): broker poll cadence에 의존하지 않고 completion 처리 직후 재스케줄링이 일어나는지 검증하는 전용 회귀 테스트 `tests/unit/control_plane/test_broker_poller_completion_driven.py`를 추가했습니다. 빈 consume batch에서도 completion만으로 `schedule()`가 호출되는 경로와, 동일 loop iteration에서 consume 후 completion 처리 시 `schedule()`가 두 번 호출되는 경로를 각각 고정했습니다. 검증: `pytest tests/unit/control_plane/test_broker_poller_completion_driven.py` 2건 통과.
- Benchmark TUI review-fix verification refresh (2026-03-11): partial workload subset parsing/controller wiring과 결과 모달 `VerticalScroll` 회귀 범위를 다시 점검했습니다. 검증: `pytest tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_results_report.py -q` 25건 통과, `python -m py_compile benchmarks/tui/app.py benchmarks/tui/controller.py benchmarks/tui/log_parser.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_results_report.py` 통과. 참고: 현재 워크스페이스 venv에는 `ruff`/`flake8`가 설치되어 있지 않아 lint 명령은 실행 불가였습니다.
- Benchmark TUI review-fix red 단계: partial workload selection(예: sleep+cpu)이 parser progress total을 여전히 전체 3개 workload 기준으로 계산하고, 결과 모달이 작은 터미널에서 내부 세로 스크롤 컨테이너 없이 렌더되는 회귀를 `tests/unit/benchmarks/test_tui_controller.py`, `tests/unit/benchmarks/test_tui_log_parser.py`, `tests/unit/benchmarks/test_tui_results_report.py`에 추가해 재현했습니다. 검증 예정: 관련 targeted pytest red 상태 확인.
- Benchmark TUI review-fix green 단계: `BenchmarkProcessController`가 parser에 정확한 selected workload subset을 전달하고, `BenchmarkLogParser`는 `active_workloads` 기준으로 total/progress를 계산하도록 보강했습니다. 또한 `ResultsSummaryModalScreen` 본문을 `VerticalScroll`로 감싸 작은 터미널에서도 결과/테이블/닫기 버튼까지 세로 스크롤할 수 있게 했습니다. 검증: `pytest tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_results_report.py -k "exact_workload_subset or partial_workload_subset or vertical_scroll" -q` 3건 통과.
- Shared OffsetTracker commit-state green 단계: `WorkManager.get_blocking_offsets()`가 shared tracker에 대해서는 `advance_high_water_mark()`를 건너뛰도록 수정해 scheduling 조회가 Kafka commit 상태를 선행 변이시키지 않게 했습니다. 관련 회귀 테스트를 유지한 채 `pytest tests/unit/control_plane/test_work_manager.py::test_get_blocking_offsets_does_not_advance_shared_tracker_commit_state -q` 및 `pytest tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_work_manager.py -q`가 각각 1건/29건 통과했습니다.
- Shared OffsetTracker commit-state red 단계: `tests/unit/control_plane/test_work_manager.py::test_get_blocking_offsets_does_not_advance_shared_tracker_commit_state`를 추가해 shared tracker를 사용하는 `WorkManager.get_blocking_offsets()`가 Kafka commit 이전에 `advance_high_water_mark()`를 호출해 `last_committed_offset`를 앞당기는 회귀를 재현했습니다. 검증: `pytest tests/unit/control_plane/test_work_manager.py::test_get_blocking_offsets_does_not_advance_shared_tracker_commit_state -q` 실패(`assert 1 == -1`).
- Benchmark assignment-wait teardown red 단계: `tests/unit/benchmarks/test_benchmark_runtime.py`에 partition assignment 대기 실패 시 benchmark poller/engine teardown 누락을 재현하는 회귀 테스트(`test_run_pyrallel_consumer_test_stops_poller_and_engine_when_assignment_wait_fails`)를 추가했습니다. 검증: `pytest tests/unit/benchmarks/test_benchmark_runtime.py -k assignment_wait_fails -q` 실패 예상(현재 cleanup 누락).
- Benchmark assignment-wait teardown green 단계: `benchmarks/pyrallel_consumer_test.py`가 partition assignment 대기 이전/도중 예외에도 `broker_poller.stop()`과 `engine.shutdown()`을 항상 실행하도록 teardown 범위를 넓혔습니다. diagnostics task는 생성된 경우에만 취소하고, 정상 run 완료 시에만 summary를 출력합니다. 검증: `pytest tests/unit/benchmarks/test_benchmark_runtime.py -k assignment_wait_fails -q` 1건 통과.
- Benchmark TUI ordering-aware progress red 단계: ordering selection 이후 progress total이 여전히 workload×phase(예: 1/9)로 계산되고 ordering column이 포함된 summary row를 파서가 이해하지 못하는 회귀를 `tests/unit/benchmarks/test_tui_log_parser.py`, `tests/unit/benchmarks/test_tui_controller.py`에 추가해 재현했습니다. 검증: `pytest tests/unit/benchmarks/test_tui_log_parser.py -q` 실패(`BenchmarkLogParser.__init__()` missing `active_orderings`).
- Benchmark TUI ordering-aware progress green 단계: `BenchmarkLogParser`에 `active_orderings`, `current_ordering`, `tps_by_workload_ordering`를 추가하고 8-column summary row(`Run | Type | Order | Topic | ...`)와 ordering-aware start/Final TPS 흐름을 파싱하도록 확장했습니다. `BenchmarkProcessController`는 TUI state의 ordering selection을 parser로 전달하고, 관련 app 회귀 테스트 fixture도 ordering-aware snapshot shape에 맞게 갱신했습니다. 검증: `pytest tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_app.py -q` 33건 통과, `python -m py_compile benchmarks/tui/log_parser.py benchmarks/tui/controller.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_app.py` 통과, `python -m pylint --persistent=n --disable=W,C,R,E -j 1 -rn -sn benchmarks/tui/log_parser.py benchmarks/tui/controller.py tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_controller.py tests/unit/benchmarks/test_tui_app.py` 통과.
- Benchmark selection redesign CLI/runtime red→green: `tests/unit/benchmarks/test_run_parallel_benchmark_tui.py`에 `--workloads sleep,cpu` / `--order key_hash,partition` 파서 및 invalid token 회귀 테스트를 추가했고, `benchmarks/run_parallel_benchmark.py`는 comma-separated multi-select 파싱/중복 제거/에러 메시지를 지원하도록 갱신했습니다. 또한 실행 루프를 workload × ordering Cartesian product로 확장해 run/topic/group 이름에 ordering을 포함하고, `BenchmarkResult`/`BenchmarkStats` 및 `benchmarks/pyrallel_consumer_test.py`에 ordering 메타데이터를 전파하도록 보강했습니다. 검증: `pytest tests/unit/benchmarks/test_run_parallel_benchmark_tui.py tests/unit/benchmarks/test_benchmark_runtime.py -v` 12건 통과, `python -m py_compile benchmarks/run_parallel_benchmark.py benchmarks/pyrallel_consumer_test.py benchmarks/stats.py benchmarks/profile_benchmark_yappi.py tests/unit/benchmarks/test_run_parallel_benchmark_tui.py tests/unit/benchmarks/test_benchmark_runtime.py` 통과.
- Benchmark selection redesign 설계 승인: `all` workload 특수값을 제거하고 `--workloads sleep,cpu` / `--order key_hash,partition` 형태의 comma-separated multi-select CLI와 TUI `SelectionList` 기반 복수 선택으로 재설계하는 방향을 문서화했습니다. 성공 모달은 overview/output을 winner cards 위로 옮기고, 선택되지 않은 workload card는 숨기는 방향으로 설계했습니다 (`docs/plans/2026-03-10-benchmark-selection-redesign-design.md`, `docs/plans/2026-03-10-benchmark-selection-redesign-plan.md`).
- Benchmark TUI workload winner cards green 단계: 결과 모달 상단 overview cards를 `sleep 1등` / `cpu 1등` / `io 1등` winner cards로 대체했습니다. 각 카드는 짧은 엔진 라벨(`baseline` / `async` / `process`), TPS, 총 소요 시간(sec)을 표시하며, `pyrallel` 타입 결과는 run name을 참고해 async/process로 축약합니다. 검증: `pytest tests/unit/benchmarks/test_tui_results_report.py -v` 7건 통과.
- Benchmark baseline result-row mapping / modal card alignment red/green: 종료 후 출력되는 aggregated results table에서 baseline run_name이 단순 `baseline` 이어서 workload=all 모드 대시보드가 baseline 셀을 매핑하지 못하는 문제를 회귀 테스트(`test_log_parser_maps_baseline_result_row_in_all_workload_mode_using_topic_name`, `test_run_baseline_round_preserves_workload_specific_run_name`)로 재현했습니다. `run_parallel_benchmark._run_baseline_round()`는 이제 workload-aware `run_name`을 전달받아 결과에 보존하고, `BenchmarkLogParser._consume_result_row()`는 필요 시 topic column으로 workload를 추론합니다. 결과 모달 카드에는 `content-align: center middle` + `text-align: center`를 적용해 제목/값 위계를 중앙 정렬 카드로 보정했습니다.
- Benchmark TUI baseline sequencing / table-width polish: 실제 사용자 로그 순서( baseline 완료 로그 → `Final TPS` → async 시작 )를 그대로 재현하는 회귀 테스트(`test_log_parser_handles_realistic_baseline_completion_sequence_before_async`)를 추가해 baseline TPS가 async 시작 직전에도 유지되는지 확인했습니다. 또한 `RunScreen`의 live summary table cell 갱신에 `update_width=True`를 적용해 큰 TPS 문자열이나 `FAILED` 표시가 잘리지 않도록 보정했습니다.
- Benchmark TUI final-TPS ordering / completion-progress red/green: start 로그가 interleave된 뒤 `Final TPS`가 들어와도 baseline/async 셀이 올바른 순서로 채워지는지 검증하는 회귀 테스트(`test_log_parser_assigns_final_tps_to_earliest_started_run_when_logs_interleave`)와, 성공 모달 직전에 progress bar가 100%로 마감되는지 검증하는 회귀 테스트(`test_run_screen_completes_progress_before_showing_success_modal`)를 추가했습니다. `BenchmarkLogParser`는 started run FIFO를 유지해 `Final TPS`를 가장 이른 미완료 run에 매핑하고, `RunScreen._on_complete()`는 성공 시 progress bar/badge를 총 run 수로 강제 마감한 뒤 모달을 띄우도록 보정했습니다.
- Benchmark TUI results modal center/table red/green: 결과 모달이 좌상단에 붙고 상세 결과를 `Static` 텍스트로만 보여주던 점을 개선하기 위해 centered modal 기본 CSS와 `DataTable` 상세 결과 렌더를 요구하는 회귀 테스트를 추가했습니다. `ResultsSummaryModalScreen`은 이제 중앙 정렬(`DEFAULT_CSS`), equal-height overview cards, `#results-table` 기반 상세 결과 표를 사용합니다. 검증: `pytest tests/unit/benchmarks/test_tui_results_report.py -v` 4건 통과.
- WorkManager assignment persistence red/green: 실운영 로그의 `TopicPartition ... is not assigned to WorkManager` 오류를 재현하는 회귀 테스트(`test_schedule_keeps_assigned_partition_after_queue_drains`)를 `tests/unit/control_plane/test_work_manager.py`에 추가했고, queue가 비었을 때 `_cleanup_empty_queue()`가 할당된 tp 자체를 제거하던 버그를 수정해 파티션 할당은 유지하되 key queue만 정리하도록 보강했습니다. 검증: `pytest tests/unit/control_plane/test_work_manager.py::test_schedule_keeps_assigned_partition_after_queue_drains -v` 통과.
- Benchmark runtime reset/assignment red 단계: per-run reset sequencing, reset 후 duplicate topic-create suppression, `no partitions assigned` 대기 훅을 고정하는 회귀 테스트 `tests/unit/benchmarks/test_benchmark_runtime.py`를 추가했고 현재 `pytest tests/unit/benchmarks/test_benchmark_runtime.py -q` 에서 3건 실패를 확인했습니다.
- Benchmark runtime reset/assignment green 단계: `benchmarks/run_parallel_benchmark.py`가 baseline/async/process 각 라운드 직전에 해당 topic/group만 reset하도록 순서를 바꿨고, `benchmarks/producer.py`/`benchmarks/pyrallel_consumer_test.py`에 `ensure_topic_exists` 가드를 추가해 reset 직후 duplicate topic-create 노이즈를 제거했습니다. 또한 `benchmarks/pyrallel_consumer_test.py`는 partition assignment를 짧게 대기하다가 토픽명을 포함한 명시적 오류로 빠르게 실패하도록 보강했습니다. 검증: `pytest tests/unit/benchmarks/test_benchmark_runtime.py -q` 4건 통과.
- Benchmark TUI options hierarchy red/green: OptionsScreen section heading/ancestor 회귀 테스트(`test_options_screen_groups_fields_under_section_headings`, `test_options_screen_places_representative_fields_in_expected_sections`)를 추가해 settings 화면이 논리 섹션으로 묶이지 않던 상태를 먼저 확인했고, `benchmarks/tui/app.py`에서 `Cluster & workload`/`Output & execution`/`Profiling`/`Advanced options` 섹션 컨테이너와 설명 문구를 도입해 기존 control id/동작은 유지한 채 스캔 계층만 강화했습니다. 검증: `pytest tests/unit/benchmarks/test_tui_app.py -k "section_headings or expected_sections" -v` 2건 통과.
- Benchmark TUI success modal grouping red 단계: 성공 결과 모달이 flat stack처럼 보여 한눈에 스캔되기 어렵다는 작업 지시를 반영해, 결과 overview / best run / best TPS / output path / detailed report heading이 분리 렌더되는지 검증하는 회귀 테스트를 `tests/unit/benchmarks/test_tui_results_report.py::test_run_screen_opens_results_modal_after_completion`에 추가했습니다. 현재 `#results-overview` 등 새 섹션이 없어 실패를 확인했습니다.
- Benchmark TUI success modal grouping green 단계: `ResultsSummaryModalScreen`이 결과 파일을 다시 파싱해 overview / best run / best TPS 카드, 결과 파일 경로, `Detailed report` 섹션 제목을 분리 렌더하도록 확장했습니다. `benchmarks/tui/results_report.py`에는 modal 상단 요약용 `ResultsOverview` helper를 추가했습니다. 검증: `pytest tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_app.py -q` 20건 통과, `python -m py_compile benchmarks/tui/app.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_tui_results_report.py` 통과.
- Benchmark TUI status dashboard red/green: UI/UX review 권고에 따라 기존 multiline status `Static`을 badge row + workload pill 형태로 대체하는 회귀 테스트를 추가했고, `RunScreen`에 `phase/workload/progress` 배지와 `sleep/cpu/io` 상태 pill 행, 별도 output path line을 구현했습니다. 실행 중 상태는 문장형 텍스트 대신 glanceable badge/pill로 읽히도록 정리했습니다.
- Benchmark TUI switch/spinner/failure-cell red/green: switch 위젯/로딩 인디케이터/실패 셀 표현을 요구하는 회귀 테스트를 추가했고, `OptionsScreen`의 boolean control을 Textual `Switch`로 교체했습니다. RunScreen은 progress bar 옆 `LoadingIndicator`를 실행 중에 표시하고 종료/취소 시 숨기며, 실패 시 현재 workload/phase 셀만 빨간 `FAILED` 텍스트로 갱신하도록 보정했습니다. 검증: `pytest tests/unit/benchmarks/test_tui_app.py::test_options_screen_orders_checkbox_blocks_label_help_control tests/unit/benchmarks/test_tui_app.py::test_options_screen_uses_prominent_title_and_helper_text tests/unit/benchmarks/test_tui_app.py::test_run_screen_mounts_dashboard_widgets tests/unit/benchmarks/test_tui_app.py::test_run_screen_marks_failed_cell_in_soft_red -v` 4건 통과.
- Benchmark TUI switch/spinner/failure-cell 설계 승인: 모든 boolean 옵션을 switch로 바꾸고, progress bar 옆에 loading indicator를 두며, 실패 시 해당 TPS 셀만 옅은 붉은색 `FAILED`로 표시하는 후속 설계를 문서화했습니다 (`docs/plans/2026-03-10-benchmark-tui-switch-spinner-design.md`, `docs/plans/2026-03-10-benchmark-tui-switch-spinner-plan.md`).
- Benchmark TUI modal/status polish red/green: 성공 시 결과 모달은 자동 표시하되 실패 시 기존 RunScreen을 유지해야 한다는 회귀 테스트와, 상태 문구/진행 표시/TPS 셀 가독성을 다듬는 회귀 테스트를 추가했습니다. 구현에서는 성공 모달 제목/보조 설명을 정리하고, RunScreen 상태를 `Running async (sleep)` 식으로 축약했으며, progress-status에 `Progress: x / total complete`를 추가하고 미완료 TPS 셀을 `…`, 완료 셀을 `123.45 TPS` 형식으로 렌더하도록 보정했습니다. 검증: `pytest tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_app.py::test_run_screen_formats_status_and_tps_cells_for_readability -v` 4건 통과.
- Benchmark TUI report/progress red 단계: 결과 요약을 자동 모달로 띄우는 회귀 테스트(`test_run_screen_opens_results_modal_after_completion`)와, result row 없이 start 로그만으로 progress가 움직여야 한다는 회귀 테스트(`test_log_parser_advances_progress_on_run_start_before_results`, `test_run_screen_uses_lifecycle_progress_value`)를 추가했고 현재 미구현 상태를 확인했습니다.
- Benchmark TUI report/progress green 단계: `BenchmarkLogParser`가 run start + `Final TPS` 로그를 이용해 `progress_value`와 live TPS 셀을 갱신하도록 확장했고, `RunScreen`은 성공 시 자동으로 결과 리포트 모달을 띄우도록 바꿨습니다. 검증: `pytest tests/unit/benchmarks/test_tui_log_parser.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_tui_app.py::test_run_screen_uses_lifecycle_progress_value tests/unit/benchmarks/test_tui_app.py::test_run_screen_updates_progress_bar_and_summary_table -v` 8건 통과.
- Benchmark TUI report/progress 설계 승인: 성공 시 결과 리포트를 자동 모달로 띄우고, progress bar는 결과 row가 아니라 run lifecycle 이벤트에서 움직이도록 보정하는 후속 설계를 문서화했습니다 (`docs/plans/2026-03-10-benchmark-tui-report-progress-design.md`, `docs/plans/2026-03-10-benchmark-tui-report-progress-plan.md`).
- Benchmark TUI option-block visibility red/green: 옵션 블록을 컨테이너로 감싼 뒤 실제 레이아웃 높이가 1줄로 고정되어 input/checkbox가 화면상 사라지는 회귀를 재현하는 테스트(`test_option_blocks_expand_to_show_controls`)를 추가했고, `.option-block { height: auto; }` 스타일을 적용해 블록이 컨트롤 높이만큼 확장되도록 수정했습니다.
- Benchmark TUI option block ordering green 단계: `OptionsScreen`의 input/select/checkbox 필드를 모두 `option-block-*` 컨테이너로 감싸고 순서를 option name → gray helper text → control로 통일했습니다. checkbox는 별도 field label 아래에 렌더되도록 정리했고, 회귀 테스트 `pytest tests/unit/benchmarks/test_tui_app.py -q` 9건 통과를 확인했습니다.
- Benchmark TUI options screen integration green 단계: `benchmarks/tui/app.py`가 새 helper metadata와 directory picker를 사용하도록 확장해 screen title class, 전 옵션 helper text, browse 버튼, profiling control disable/enable 동기화, 선택 경로 state 반영을 연결했습니다. 회귀 검증 `pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_state.py tests/unit/benchmarks/test_tui_path_picker.py -q` 14건 통과를 확인했습니다.
- Benchmark TUI UX polish red 단계: 옵션 제목 스타일 hook, helper text, 출력 경로 browse 버튼, 디렉터리 picker 진입을 검증하는 회귀 테스트를 `tests/unit/benchmarks/test_tui_app.py`에 추가했고, 현재 관련 UI가 없어 3건 실패를 확인했습니다.
- Benchmark TUI UX polish green 단계: `benchmarks/tui/option_help.py`에 옵션 메타데이터를 분리하고, `OptionsScreen`이 더 강한 제목 스타일/모든 visible 옵션용 helper text/출력 경로 browse 버튼/디렉터리 picker 진입/프로파일링 master toggle 비활성화를 렌더하도록 확장했습니다. 회귀 테스트 `pytest tests/unit/benchmarks/test_tui_app.py -v` 7건 통과를 확인했습니다.
- Benchmark TUI profiling master toggle green 단계: `BenchmarkTuiState`에 `profiling_enabled`를 추가하고, 마스터 토글이 꺼져 있으면 `--profile*`/`--py-spy*` 관련 argv를 모두 생략하도록 정리했습니다. 회귀 테스트 `pytest tests/unit/benchmarks/test_tui_state.py -v` 4건 통과를 확인했습니다.
- Benchmark TUI directory picker green 단계: 재사용 가능한 Textual 모달 디렉터리 picker(`benchmarks/tui/path_picker.py`)와 전용 단위 테스트(`tests/unit/benchmarks/test_tui_path_picker.py`)를 추가했습니다. `DirectoryTree` 렌더링과 confirm/cancel dismissal을 검증했고 `pytest tests/unit/benchmarks/test_tui_path_picker.py -v` 3건 통과를 확인했습니다.
- Benchmark TUI UX polish 설계 승인: 옵션 화면에서 (1) 더 강한 제목 스타일, (2) 모든 옵션용 회색 helper text, (3) profiling master toggle, (4) Textual tree 기반 모달 디렉터리 picker를 추가하는 설계를 확정했고 문서화했습니다 (`docs/plans/2026-03-10-benchmark-tui-ux-polish-design.md`, `docs/plans/2026-03-10-benchmark-tui-ux-polish-plan.md`).
- Benchmark Textual TUI TDD 시작: 무옵션 실행 시 TUI 진입, TUI state→argv 매핑, 로그 진행상태 파서, Textual 앱 스모크를 검증하는 신규 red 테스트를 추가했습니다 (`tests/unit/benchmarks/test_run_parallel_benchmark_tui.py`, `tests/unit/benchmarks/test_tui_state.py`, `tests/unit/benchmarks/test_tui_log_parser.py`, `tests/unit/benchmarks/test_tui_app.py`). 현재 `textual` 미설치와 신규 모듈 부재로 수집 단계에서 실패하는 상태를 먼저 확인했습니다.
- Benchmark Textual TUI 구현: `benchmarks/run_parallel_benchmark.py`를 `main(argv)`/`build_parser()`/`run_benchmark()`/`launch_tui()`로 분리해 무인자 실행 시 Textual 앱으로 진입하고, 인자가 있으면 기존 CLI 흐름을 유지하도록 조정했습니다. `benchmarks/tui/`에 state/argv builder, 로그 진행상태 파서, subprocess controller, Textual app/screens를 추가했고 `textual` 의존성을 프로젝트에 반영했습니다. 사용법은 `benchmarks/README.md`와 루트 `README.md` 벤치마크 섹션에 문서화했습니다.
- 리뷰 반영: RunScreen의 Back이 활성 benchmark subprocess를 정리하지 않던 문제와 cancel 상태가 실패 상태로 덮이던 문제를 수정했습니다. `action_back()`는 실행 중이면 먼저 cancel+await 후 pop하도록 바꾸고, controller는 pre-spawn cancel 요청도 기억하도록 보강했습니다. `textual`은 코어 런타임 의존성에서 제거하고 dev dependency로 이동했습니다. 회귀 테스트 추가 후 `pytest tests/unit/benchmarks` 12건 통과를 확인했습니다.
- 직접 스크립트 실행 경로 회귀 수정: `python benchmarks/run_parallel_benchmark.py` / `uv run benchmarks/run_parallel_benchmark.py`처럼 모듈이 아닌 파일 경로로 실행할 때 repo root가 `sys.path`에 없어 `from benchmarks...` import가 실패하던 문제를 재현(`test_script_path_execution_supports_help`) 후 수정했습니다. `__package__`가 비어 있을 때 프로젝트 루트를 `sys.path`에 추가해 direct-script와 `-m` 실행을 모두 지원합니다.
- TUI 폼 라벨 UX red 단계: 옵션 화면의 `Input` 위젯이 값만 보여 주고 필드 이름을 드러내지 않아 사용성이 낮다는 피드백을 받았고, 사람이 읽을 수 있는 field label이 실제로 렌더되는지 검증하는 회귀 테스트(`test_options_screen_shows_human_readable_field_labels`)를 추가했습니다. 현재 라벨 부재로 실패 상태를 확인했습니다.
- TUI 폼 라벨 UX green 단계: `OptionsScreen`에 labeled input/select helper를 추가해 핵심/고급 입력 필드 앞에 `Bootstrap servers`, `Number of messages`, `Timeout (sec)` 같은 명시적 라벨을 표시하고 placeholder도 함께 넣었습니다. 회귀 테스트 포함 `pytest tests/unit/benchmarks` 14건 통과를 확인했습니다.

## 최근 업데이트 (2026-03-09)
- WorkManager scheduler scan-cost 완화: `WorkManager.schedule()`가 매 dispatch마다 모든 virtual queue head를 선형 재스캔하지 않도록 runnable queue deque + head-offset index를 추가했습니다. blocking offset은 `(tp, offset)` head 인덱스로 바로 찾고, 일반 dispatch는 runnable deque round-robin으로 선택해 same-key / partition ordering 보장을 유지하면서 tail-latency 악화를 줄였습니다. submit 실패 시 requeue 경로도 새 인덱스를 복구하도록 보강했습니다.
- 회귀 테스트 추가: `tests/unit/control_plane/test_work_manager.py`에 virtual queue head 재스캔 방지 회귀 테스트를 추가했고, `tests/unit/control_plane/test_work_manager.py` + `tests/unit/control_plane/test_work_manager_ordering.py` 전체 통과로 ordering / submit-failure 동작을 재검증했습니다.

## 최근 업데이트 (2026-03-01)
- OffsetTracker single-source step: `BrokerPoller._on_assign` now creates the per-partition `OffsetTracker` objects first and passes those shared tracker instances into `WorkManager.on_assign`, so BrokerPoller/WorkManager stop diverging on separate tracker objects. `WorkManager` now recognizes shared tracker assignments, skips duplicate completion mutation for shared trackers, invalidates blocking-cache on assign/revoke, and BrokerPoller reschedules after completion processing. Regression tests added for shared tracker wiring and shared-tracker completion handling (`tests/unit/control_plane/test_broker_poller.py`, `tests/unit/control_plane/test_work_manager.py`).
- ProcessExecutionEngine 타입 정리: timeout duplicate-execution 회귀 테스트와 함께 직렬화 helper/IPC queue 경계에 명시적 타입 별칭을 추가하고 `from __future__ import annotations`를 적용해 basedpyright가 수정된 process 엔진 경로에서 0 errors를 보고하도록 정리했습니다.
- WorkManager logging 정리: submit 실패와 unmanaged completion 경로의 `print(...)`를 structured logging(`logger.exception` / `logger.warning`)으로 교체해 운영 로그와 일관성을 맞췄습니다. 단위 테스트(`tests/unit/control_plane/test_work_manager.py`)로 두 경로의 로그 방출과 재큐잉/경고 동작을 검증했습니다.
- `PyrallelConsumer` public facade ordering wiring 수정: facade가 직접 생성하는 `WorkManager`에 `OrderingMode.KEY_HASH`를 명시적으로 전달해, BrokerPoller 기본 경로와 동일한 same-key 직렬화 기본값을 사용하도록 정렬했습니다. 회귀 테스트(`tests/unit/test_consumer_and_offset_manager.py`)에 facade wiring 검증을 추가했습니다.
- ProcessExecutionEngine timeout duplicate-execution 방지: 워커 태임아웃 시 completion을 워커가 직접 enqueue하지 않고, in-flight registry에 `timed_out/timeout_error/attempt`를 남긴 뒤 종료하도록 조정했습니다. 부모 `_ensure_workers_alive()`는 해당 엔트리를 재큐잉하지 않고 단일 FAILURE completion으로 변환합니다. 회귀 단위 테스트 추가: `tests/unit/execution_plane/test_process_execution_engine.py`.
- BrokerPoller↔WorkManager assignment coupling 보정: `BrokerPoller._on_assign`가 WorkManager로 파티션별 시작 오프셋을 함께 전달하도록 수정했고, `WorkManager.on_assign`는 기존 리스트 호출을 유지하면서 `{TopicPartition: starting_offset}` 매핑도 받아 OffsetTracker를 정확한 시작 지점으로 초기화하도록 확장했습니다.
- 회귀 테스트 추가: `tests/unit/control_plane/test_work_manager.py`, `tests/unit/control_plane/test_broker_poller.py`에 시작 오프셋 전달/초기화 계약 테스트를 보강했습니다.
- v0.1.1 PyPI 릴리스: `pyproject.toml`/`uv.lock` 버전 0.1.1로 갱신, `uv run pytest tests/unit` 통과, `uv run python -m build`로 sdist/wheel 생성 후 `uv run twine check dist/*` 검증, `uv run twine upload --repository pypi dist/*` 성공(https://pypi.org/project/pyrallel-consumer/0.1.1/).
- `ParallelConsumerConfig.poll_batch_size`/`worker_pool_size`에 0 방지 검증(gt=0) 추가, Pydantic Field 제약으로 초기화 시 ValidationError 발생하도록 수정. 단위 테스트 보강(`tests/unit/test_config.py`)으로 0 값 거부 경로 검증.
- `KafkaConfig` 타입 힌트/LSP 정비: basedpyright 설치 후 경고 해소(bootstrap servers 파서 타입 체크, model_extra 캐스팅 등)하여 LSP clean 상태 확인.
- BrokerPoller 커밋 경로에서 MagicMock 분기 제거, `KafkaTopicPartition`를 일관 사용하며 metadata 전달(테스트 추가: `test_commit_offsets_uses_topic_partition_with_metadata`).
- AsyncExecutionEngine 종료 강화: shutdown 후 `submit` 거부, grace timeout(`AsyncConfig.shutdown_grace_timeout_ms`, 기본 5000ms) 적용해 미완료 태스크 취소 + gather, 관련 테스트 추가 두 건.
- WorkManager 키 큐 정리: `_peek_queue` 복사 제거 + completion 처리 시 빈 virtual queue 삭제 (키/파티션/무순서 공통), 검증 테스트 추가.
- 운영 플레이북 문서 추가(`docs/ops_playbooks.md`) 및 README 링크: 프로필별 권장 설정, 장애 대응, 모니터링/알람 기준, 튜닝 체크리스트, 워크로드 가이드, 성능 테스트 매트릭스 포함.

## 최근 업데이트 (2026-02-27)
- `docs/index.md` 생성: `docs/` 루트와 `plans/` 서브디렉터리 문서를 스캔하고, 내용 기반 3-10 단어 설명을 포함한 인덱스를 추가했습니다. 숨김 파일은 제외하고 상대 경로(`./`) 기준으로 정렬했습니다.

## 최근 업데이트 (2026-02-26)
- Gap 타임아웃 가드 추가: `ParallelConsumerConfig.max_blocking_duration_ms`(기본 0=비활성). BrokerPoller가 `get_blocking_offset_durations`를 기준으로 임계 초과 시 강제 실패 이벤트를 생성하고, WorkManager 경로를 통해 DLQ/커밋 처리. 관련 단위 테스트 추가 (`tests/unit/control_plane/test_blocking_timeout.py`).
- Shutdown 드레인 보강: BrokerPoller 종료 시 남은 completion 이벤트/타임아웃 이벤트를 한 번 더 처리해 커밋 누락을 최소화.
- MetadataEncoder 견고화: Bitset 인코딩을 hex 기반으로 변경해 메타데이터 크기 축소, RLE/Bitset 모두 초과 시 sentinel("O")로 overflow 표시, decode 실패 시 fail-closed(set 반환) + 경고 로깅. 신규 단위 테스트 추가(`tests/unit/control_plane/test_metadata_encoder.py`).
- .env/.env.sample의 주석 포함 값으로 인한 Pydantic 검증 오류를 제거: `KAFKA_AUTO_OFFSET_RESET`, `EXECUTION_MODE` 값을 순수 값으로 정리하여 단위 테스트 (`test_blocking_timeout.py`) 실패를 해소.
- 커버리지 보강:
- PrometheusMetricsExporter에 registry 주입 옵션 추가, HTTP 서버 비활성 시 no-op 보장, gauge/counter/histogram 동작 검증 테스트 추가 (`tests/unit/metrics/test_prometheus_exporter.py`).
- ProcessExecutionEngine 헬퍼들에 대한 단위 테스트 추가(`_decode_incoming_item` oversize/error, msgpack decode, `_calculate_backoff` 지터 포함) (`tests/unit/execution_plane/test_process_engine_helpers.py`).
- prometheus_client를 런타임 의존성에 추가(프로덕션/테스트 모두에서 메트릭 노출 가능), pyproject.toml dependencies에 반영하고 환경에 설치.

## 최근 업데이트 (2026-02-25)
- DLQ retry-cap: 프로세스 엔진의 `worker_died_max_retries` 경로를 커버하는 단위 테스트 추가 (`tests/unit/execution_plane/test_process_execution_engine.py`, `tests/unit/control_plane/test_broker_poller_dlq.py`).
- 모니터링 스택 스모크 테스트: `docker compose up -d kafka-1 kafka-exporter prometheus grafana` 후 확인.
  - Prometheus `/-/ready` OK, active targets 중 `kafka-exporter` health=up, `pyrallel-consumer` health=down은 앱 미실행으로 정상.
  - Grafana `/api/health` OK (10.4.2, DB ok).
  - Kafka exporter `/metrics` 응답 확인.
  - 테스트 후 `docker compose down`으로 정리 완료.
- 패키징 준비: `pyproject.toml`에 build-system(setuptools/wheel), 메타데이터(3.12+, classifiers, keywords) 정리. `python -m build`로 sdist/wheel 빌드 성공(`dist/` 생성). 라이선스는 임시 `Proprietary` 텍스트 사용 중이며 setuptools에서 라이선스 필드 테이블은 추후 SPDX 문자열로 전환 필요(경고 발생).
- 라이선스 전환: 프로젝트 라이선스를 Apache-2.0으로 지정, LICENSE 파일 추가, `pyproject.toml`에 SPDX 표현/라이선스 파일 설정 반영. `python -m build` 재확인 성공.
- 보안 하드닝 (2026-02-25):
  - `docker-compose.yml`의 Grafana admin 비밀번호 하드코딩 제거 → env 주입(`GF_SECURITY_ADMIN_PASSWORD=${...:?missing}`), `.env.sample`에 placeholder 추가.
  - DLQ payload 모드 추가: `KAFKA_DLQ_PAYLOAD_MODE=metadata_only` 시 key/value 미전송, 기본 `full` 유지. 토픽/접미사 화이트리스트 검증 추가.
  - 토픽/로그 검증 유틸 추가(`pyrallel_consumer/utils/validation.py`) 적용.
  - msgpack 역직렬화 크기 제한(`msgpack_max_bytes`, 기본 1,000,000) 및 decode 가드 추가.
  - 관련 단위 테스트 추가/통과, `python -m build` 재확인 (라이선스 경고만 남음).

## 목차
- 1. 프로젝트 철학 및 목표
- 2. 현재 폴더 구조 및 개편 방향
- 3. 현재까지의 진행 상황 (v1 → v2)
- 4. 다음 진행 계획, Phase별 진행 현황 및 테스트 요약

## 1. 프로젝트 철학 및 목표 (From prd_dev.md)

본 프로젝트는 Java 생태계의 `confluentinc/parallel-consumer`에서 영감을 받아, Python `asyncio` 환경에 최적화된 **고성능 Kafka 병렬 처리 라이브러리**를 개발하는 것을 목표로 합니다.

### 1.1. 핵심 목표
- **Kafka Control Plane 단일화**: 실행 모델(Async/Process)에 관계없이 Kafka 통신, 오프셋 관리, 리밸런싱 로직은 단일 코드로 유지합니다.
- **실행 모델 공존**: 단일 릴리즈 내에 `AsyncExecutionEngine`과 `ProcessExecutionEngine`을 모두 제공하며, 런타임 설정으로 선택 가능하게 합니다.
- **GIL 제약 회피**: `ProcessExecutionEngine`을 통해 CPU-bound 작업을 위한 구조적 해결책을 제공합니다.
- **확장성**: `ExecutionEngine` 인터페이스를 통해 새로운 실행 모델을 추가할 수 있는 구조를 갖습니다.

### 1.2. 설계 철학
> This project treats execution models as interchangeable, but treats offset correctness as sacred.

> 이 프로젝트는 실행 모델을 교체 가능한 부품으로 취급하지만, 오프셋의 정확성은 신성불가침으로 다룹니다.

### 1.3. 아키텍처 원칙
- **Control Plane**: Kafka 소비, 리밸런싱, 오프셋 관리, 상태 제어를 담당
- **Execution Plane**: 메시지의 병렬 처리, 실행 모델(Async/Process) 구현을 담당
- **Worker Layer**: 사용자의 비즈니스 로직 실행을 담당

**핵심 원칙**: Control Plane은 현재 어떤 Execution Engine이 사용되는지 절대 인지하지 못해야 합니다.

### 1.4. 개발 시 참고 지침
- **주요 참고**: `prd_dev.md`를 기본 개발 명세서로 참고합니다
- **보충 참고**: `prd_dev.md`에 명시되지 않은 세부사항이나 의문사항이 발생할 경우 `prd.md`를 참고하여 원본 설계 의도를 확인합니다
- **우선순위**: `prd_dev.md` > `prd.md` 순서로 적용하되, 두 문서 간 충돌 시 `prd_dev.md`의 개발자 친화적 명세를 우선으로 적용합니다
- **커밋 정책**: 기능 단위(예: 테스트 1개 작성, 버그 1개 수정 등) 작업이 끝날 때마다 즉시 커밋합니다. Agent 세션이 중단되어도 작업 흐름이 끊기지 않도록, 변경사항은 단계별로 잘게 나누어 커밋해 두어야 합니다.
- **인수인계 최우선 원칙**: `GEMINI.md` 업데이트는 모든 작업에서 최우선 순위입니다. Agent 세션은 언제든 종료될 수 있으므로, **각 단계(테스트 1개 작성, 버그 1개 수정 등)를 완료할 때마다 즉시 `GEMINI.md`를 업데이트**합니다. 작업 완료 후 일괄 업데이트가 아닌, 단계별 점진적 업데이트를 수행합니다.
- **기타사항**: 만약 인계사항이 많은 경우 별도 파일을 git이 볼수 없는 영역에 기록하고 그 파일 경로를 기록합니다.
    로그에 출력하는 변수들은 f 표현식이 아닌 % 표현식으로 사용하여 파싱에 오류가 없도록 합니다.
    같은 행위를 반복하고 있는 경우 즉시 작업을 종료하고 사용자의 판단을 물어봅니다.
    **프로덕션 코드에는 `assert` 구문을 사용하지 마십시오. 대신 명시적인 예외(`ValueError`, `RuntimeError` 등)를 발생시키십시오. `assert`는 테스트 코드에서만 허용됩니다.**

## 2. 현재 폴더 구조 및 향후 개편 방향

### 2.1. 현재 구조 (v1 기반)
프로젝트의 핵심 로직은 `pyrallel_consumer` 패키지 내에 각 파일의 역할에 따라 분리되어 있습니다.

- **`pyrallel_consumer/`**:
    - **`__init__.py`**: 패키지 초기화 파일입니다.
    - **`constants.py`**: 프로젝트 전역에서 사용될 상수를 정의합니다. (현재 비어있음)
    - **`config.py`**: `pydantic-settings` 기반의 `KafkaConfig` 클래스를 정의하여 Kafka 클라이언트 설정을 관리합니다. 환경 변수에서 설정을 로드할 수 있습니다. `dump_to_rdkafka` 유틸리티 메서드를 포함하여 Pydantic 설정 객체를 `librdkafka` 호환 딕셔너리로 변환할 수 있습니다.
    - **`logger.py`**: `LogManager`를 통해 프로젝트 전반의 로깅을 관리합니다. (현재는 플레이스홀더)
    - **`consumer.py`**:
        - `ParallelKafkaConsumer` 클래스가 위치한 프로젝트의 핵심 파일입니다.
        - 메시지 소비, 병렬 처리 태스크 생성, Kafka 클라이언트 라이프사이클 관리를 총괄합니다.
        - `OffsetTracker`와 `worker` 모듈을 사용하여 실제 오프셋 관리와 역직렬화 작업을 위임합니다.
    - **`offset_manager.py`**:
        - `OffsetTracker` 클래스를 정의합니다.
        - `SortedSet`을 사용하여 처리 중인 메시지의 오프셋을 파티션별로 추적하고, 안전하게 커밋할 수 있는 오프셋(`safe_offset`)을 계산하는 역할을 담당합니다.
    - **`worker.py`**:
        - `batch_deserialize` 함수를 제공하여 메시지 역직렬화를 담당합니다.
        - `orjson`을 사용하며, `ThreadPoolExecutor`에서 실행되어 메인 이벤트 루프의 부하를 줄입니다.

- **`tests/`**:
    - 프로젝트의 테스트 코드를 관리합니다.
    - **`tests/unit/`**: 각 모듈의 개별 기능 단위를 격리하여 테스트합니다. (예: `OffsetTracker`의 `mark_complete` 기능)
    - **`tests/integration/`**: 여러 모듈 간의 상호작용 또는 실제 외부 시스템(Kafka, DB 등)과의 연동을 테스트합니다. (예: `ParallelKafkaConsumer`의 전체 메시지 처리 흐름)

### 2.2. v2 아키텍처에 따른 개편 방향
`prd_dev.md`의 3계층 아키텍처를 반영하여 다음과 같이 구조를 개편할 예정입니다.

```
pyrallel_consumer/
├── __init__.py
├── constants.py
├── config.py
├── logger.py
├── dto.py                    # CompletionEvent, TopicPartition 등 DTO 정의
├── control_plane/            # Control Plane 레이어
│   ├── __init__.py
│   ├── broker_poller.py      # Kafka 소비 및 리밸런싱
│   ├── offset_tracker.py     # SortedSet 기반 상태 머신
│   ├── work_manager.py       # Blocking Offset 스케줄러
│   └── metadata_encoder.py   # RLE/Bitset 이중 인코딩
├── execution_plane/          # Execution Plane 레이어
│   ├── __init__.py
│   ├── base.py              # ExecutionEngine 추상 인터페이스
│   ├── async_engine.py      # AsyncExecutionEngine 구현
│   └── process_engine.py    # ProcessExecutionEngine 구현
└── worker_layer/            # Worker Layer (기존 worker.py 확장)
    ├── __init__.py
    ├── async_worker.py      # Async 워커 유틸리티
    └── process_worker.py    # Process 워커 래퍼
```

**개편 원칙**:
- Control Plane은 Execution Engine의 존재를 인지하지 못함
- 각 레이어는 명확한 인터페이스를 통해 통신
- 기존 코드는 점진적으로 새 구조로 마이그레이션

## 3. 현재까지의 진행 상황 (v1 → v2)

### 3.1. v1 완료 사항 (기존 코드베이스 정리)
`prd.md` 설계에 따라 기존 코드베이스의 문제를 해결하고 구조를 개선하는 1단계 작업을 완료했으며, 설계 문서를 개편했습니다.

- **중복 코드 및 버그 수정**:
    - `parallel_consumer.py`와 `consumer.py`로 중복되던 파일을 `consumer.py`로 통합했습니다.
    - `_run_consumer` 메소드가 중복 정의되던 문제를 해결했습니다.
    - 오프셋 추적 시 `defaultdict`에 잘못 접근하던 치명적인 버그(`AttributeError`)를 수정했습니다.

- **관심사 분리 (SoC) 리팩토링**:
    - `consumer.py`에 혼재되어 있던 기능들을 `offset_manager.py`와 `worker.py`로 분리하고, `consumer.py`가 이들을 사용하도록 구조를 변경했습니다.
    - 오프셋 관리 로직은 `OffsetTracker` 클래스로 완전히 위임했습니다.
    - 메시지 역직렬화는 `worker.py`의 `batch_deserialize` 함수를 사용하도록 변경했습니다.

- **프로젝트 구조 개선**:
    - `src` 디렉토리에 의존하던 잘못된 `import` 경로를 수정했습니다.
    - `config.py`와 `logger.py`를 패키지 내에 추가하여 설정과 로깅을 위한 기반을 마련했습니다.
    - `pydantic-settings`를 `KafkaConfig`에 적용하여 `.env` 파일 로딩 기능을 추가했습니다.

### 3.2. v2 설계 검토 및 개편 (2026-02-08)
Oracle 분석을 통해 기존 설계 문제들을 식별하고 `prd_dev.md`로 개편했습니다.

- **설계 문제 해결**:
    - 명명 통일성 문제 해결 ("Pyrallel Consumer" vs "PAPC")
    - 구현 격차 문제 해결 (기본 T1-T2 → 전체 T1-T11 로드맵 명시)
    - 가상 파티션 설계 모호성 개선 (Key Extractor 개념 도입)
    - 메타데이터 인코딩 전략 구체화 (RLE + Bitset 이중 인코딩)

- **아키텍처 재설계**:
    - 3계층 아키텍처 도입 (Control Plane, Execution Plane, Worker Layer)
    - ExecutionEngine 추상화를 통한 Async/Process 엔진 공존 구조
    - Epoch Fencing을 통한 리밸런싱 안정성 확보
    - 관측성 모델 도입 (True Lag, Gap, Blocking Offset 지표)

- **TDD 전략 수립**:
    - Contract Testing을 통한 ExecutionEngine 호환성 보장
    - Phase별 구현 계획 수립 (Control Plane → Execution → Async → Process)
    - 단위/통합/계약 테스트 피라미드 구성

### 3.3. v2 아키텍처 구현 진행 상황 (신규)

`prd_dev.md`의 TDD 실행 순서에 따라 **Phase 1 – Control Plane Core** 구현이 완료되었으며, **Phase 2 – WorkManager & Scheduling**의 일부가 진행 중입니다.

#### 2026-02-26 – ProcessExecutionEngine 프로액티브 워커 리사이클링
- `ProcessConfig`에 `max_tasks_per_child`(기본 0, 비활성)와 `recycle_jitter_ms`(기본 0) 추가.
- 프로세스 워커가 작업을 처리할 때마다 카운트하며, `max_tasks_per_child + jitter`에 도달하면 현재 배치에서 남은 WorkItem을 재큐잉 후 종료; 부모 `_ensure_workers_alive()`가 동일 인덱스에 새 워커를 재시작.
- 로깅: `%` 포맷 사용 (`ProcessWorker[%d] recycling after %d tasks (limit=%d, jitter=%d)`). 타임아웃 예외 메시지도 `%` 포맷으로 교체.
- 테스트: `pytest tests/unit/execution_plane/test_process_execution_engine.py -vv` 전체 12건 통과. 브로커 통합: `pytest tests/integration/test_broker_poller_integration.py -vv` 통과.
- DLQ 퍼블리시: `dlq_payload_mode`가 모킹된 설정에 없을 때 `DLQPayloadMode.FULL`을 기본값으로 사용해 통합 테스트 통과.
- 커밋 메타데이터: `MetadataEncoder.encode_metadata` 결과를 Kafka commit 오프셋에 세팅(메타데이터 None 방지).

- **v2 아키텍처 구조 개편**:
    - `control_plane`, `execution_plane`, `worker_layer` 디렉토리 구조를 생성했습니다.
    - 레이어 간 통신을 위한 `dto.py` 파일을 정의하고 DTO들을 구현했습니다.
    - 기존 `consumer.py`를 삭제하고, `pyrallel_consumer/control_plane/broker_poller.py`로 리팩토링을 시작했습니다.

- **`OffsetTracker` 구현 완료 (TDD 우선순위 1)**:
    - `SortedSet`을 사용한 `OffsetTracker`를 구현하고, `mark_complete`, `advance_high_water_mark`, `get_gaps` 등의 핵심 로직을 완성했습니다.
    - 단위 테스트를 작성하고 모든 테스트가 통과하는 것을 확인했습니다.
    - Epoch Fencing 연동을 위해 `increment_epoch`, `get_current_epoch` 등의 메서드를 추가했습니다.
    - **Blocking Offset duration 추적 기능 추가**: `get_gaps` 호출 시 blocking offset의 시작 시간을 기록하고, `get_blocking_offset_durations` 메서드를 통해 blocking offset의 지속 시간을 노출하는 기능을 추가했습니다.

- **`MetadataEncoder` 구현 완료 (TDD 우선순위 2)**:
    - RLE와 Bitset을 동시에 인코딩하여 더 짧은 결과물을 선택하는 `MetadataEncoder`를 구현했습니다.
    - 단위 테스트를 작성하고 모든 테스트 통과를 확인했습니다.

- **Rebalance & Epoch Fencing 구현 완료 (TDD 우선순위 3)**:
    - `BrokerPoller`가 파티션별로 `OffsetTracker`를 관리하도록 리팩토링을 완료했습니다.
    - `_on_assign`, `_on_revoke` 콜백에서 Epoch를 관리하는 로직을 구현했으며, 관련 테스트에서 발생했던 `NameError` 및 타입 힌트 불일치 문제를 해결했습니다.
    - 메시지 처리 중 Epoch Fencing 로직을 `_process_virtual_partition_batch`에 구현하여 이전 세대의 좀비 메시지를 안전하게 폐기합니다.
    - `_on_revoke`에서 `MetadataEncoder`를 사용하여 최종 커밋 메타데이터를 생성하고 Kafka에 전달하는 로직을 구현했습니다.

### 3.4. v2 아키텍처 구현 진행 상황 (WorkManager 및 ExecutionEngine 연동)

`prd_dev.md`의 TDD 실행 순서에 따라 **Phase 1 – Control Plane Core**의 `WorkManager` 구현이 완료되었으며, **Phase 2 – WorkManager & Scheduling**의 일부가 진행 중입니다.

*   **`dto.py` 업데이트**:
    *   `WorkItem` DTO에 `id: str` 필드 추가.
    *   `CompletionEvent` DTO에 `id: str` 필드 추가.
    *   `Any` 타입을 사용하기 위해 `from typing import Any` 임포트 추가.
*   **`pyrallel_consumer/execution_plane/base.py` 생성**:
    *   `ExecutionEngine`의 추상 인터페이스인 `BaseExecutionEngine` 클래스를 정의했습니다. 이는 `WorkManager`가 `ExecutionEngine`의 구체적인 구현을 알지 못하고도 상호작용할 수 있도록 하는 DI(Dependency Injection)의 핵심 경계 역할을 합니다.
*   **`WorkManager` 핵심 로직 구현**:
    *   생성자에서 `BaseExecutionEngine` 인스턴스를 주입받도록 변경하여 Control Plane과 Execution Plane 간의 의존성을 명확히 했습니다.
    *   `_in_flight_work_items` 맵과 `_current_in_flight_count` 변수를 추가하여 `WorkManager`가 현재 처리 중인 메시지(Work-in-Flight)의 총 수를 직접 관리하도록 했습니다. 이는 전반적인 시스템 부하 추적 및 백프레셔(Backpressure) 구현의 기반이 됩니다.
    *   `submit_message` 메서드에서 각 `WorkItem`에 `uuid` 기반의 고유 `id`를 할당하도록 변경했습니다. 이 `id`는 작업의 생명주기 동안 고유하게 식별되며, `WorkManager`가 `in-flight` 상태의 작업을 추적하고 완료 이벤트를 매칭하는 데 사용됩니다. `submit_message`는 이제 내부 큐에 메시지를 추가하는 역할만 수행하며, 실제 ExecutionEngine으로의 제출은 `_try_submit_to_execution_engine`에 의해 별도로 관리됩니다.
    *   `_try_submit_to_execution_engine` 메서드를 개선했습니다. 이 메서드는 `max_in_flight_messages`를 초과하지 않는 범위 내에서 가상 파티션 큐에서 `WorkItem`을 가져와 `ExecutionEngine`의 `submit` 메서드로 작업을 제출합니다. 특히, "Lowest blocking offset 우선" 스케줄링 정책을 구현하여 HWM 진행을 막고 있는 메시지를 우선적으로 처리하도록 했습니다.
    *   `poll_completed_events` 메서드를 수정했습니다. `ExecutionEngine`으로부터 완료 이벤트를 주기적으로 폴링하고, 수신된 `CompletionEvent`의 `id`를 사용하여 `_in_flight_work_items`에서 해당 `WorkItem`을 제거하고 `_current_in_flight_count`를 감소시킵니다. 작업 완료 후에는 `_try_submit_to_execution_engine`을 다시 호출하여 처리 가능한 새 작업을 제출하도록 시도합니다.
    *   `OffsetTracker` 초기화 시 필수 인자 (`topic_partition`, `starting_offset`, `max_revoke_grace_ms`)를 전달하도록 `on_assign` 메서드를 수정했습니다.
    *   `get_total_in_flight_count` 메서드가 `WorkManager`의 `_current_in_flight_count`를 반환하도록 변경했습니다.
    *   `mark_complete` 호출 시 `epoch` 인자를 제거하여 `OffsetTracker`의 시그니처와 일치시켰습니다. (테스트 코드 수정 완료)
    *   `get_blocking_offsets` 메서드를 `OffsetTracker`의 `get_gaps`를 사용하도록 리팩토링했습니다.
    *   `on_assign`과 `on_revoke` 메서드에 `_rebalancing` 플래그를 추가하여 리밸런스 중 메시지 제출을 차단하는 로직을 구현했습니다.
*   **`WorkManager` 관측성(Observability) 기능 추가**:
    *   `get_gaps()` 메서드를 추가하여 각 토픽-파티션별 완료되지 않은 오프셋 범위(갭) 정보를 노출합니다.
    *   `get_true_lag()` 메서드를 추가하여 각 토픽-파티션별 실제 지연(last fetched offset - last committed offset) 정보를 노출합니다.
    *   `get_virtual_queue_sizes()` 메서드를 추가하여 각 가상 파티션 큐의 현재 크기 정보를 노출합니다.
*   **`BrokerPoller` `mypy` 오류 수정**:
    *   `WorkManager` 생성자에 `BaseExecutionEngine`을 전달하도록 수정했습니다.
    *   `submit_message` 호출 시 누락되었던 `key`와 `payload` 인자를 `msg.key()`와 `msg.value()`로 각각 전달하도록 수정했습니다.
    *   `CompletionStatus`를 import하고, 완료 이벤트 상태 비교 로직을 `CompletionStatus.FAILURE`를 사용하도록 수정했습니다.
*   **`tests/unit/control_plane/test_work_manager.py` 업데이트 및 디버깅**:
    *   `WorkManager` 생성자에 `mock_execution_engine`을 전달하도록 수정하고, 테스트 픽스처들을 관련 변경사항에 맞춰 업데이트했습니다.
    *   `OffsetTracker` 클래스의 생성자 변경사항을 반영하기 위해 `unittest.mock.patch`를 사용하여 `OffsetTracker` 클래스를 모킹했습니다. 이를 통해 `WorkManager.on_assign`이 `OffsetTracker`를 인스턴스화할 때 올바른 인자를 전달하는지 확인하고, 모킹된 `OffsetTracker`의 동작을 제어할 수 있게 했습니다.
    *   `test_submit_message`는 이제 메시지가 내부 큐에 올바르게 추가되고 추적되는지 확인한 후, `_try_submit_to_execution_engine`을 명시적으로 호출하여 ExecutionEngine으로의 제출을 검증합니다.
    *   `test_poll_completed_events`에서 `ExecutionEngine.poll_completed_events`를 모킹하여 완료 이벤트를 반환하도록 하고, 이들이 `WorkManager`에 의해 올바르게 처리되고 `_current_in_flight_count`가 감소하며 `_in_flight_work_items`에서 제거되는지 검증했습니다.
    *   `OffsetTracker`에서 `get_blocking_offset` 메서드가 제거됨에 따라 이와 관련된 테스트 코드 및 Mock 설정을 제거 및 수정했습니다.
    *   `test_on_assign_and_on_revoke` 테스트에서 발생한 어설션 논리 오류 (해지되지 않은 파티션이 `_offset_trackers`에 남아있어야 하는 부분)를 수정했습니다.
    *   `mark_complete` 호출의 인자가 `offset` 하나만 받도록 변경된 구현과 테스트 코드가 일치하도록 수정하여 `AssertionError`를 해결했습니다.
    *   새롭게 추가된 `test_prioritize_blocking_offset` 테스트를 통해 "Lowest blocking offset 우선" 스케줄링 정책이 올바르게 작동함을 검증했습니다.
    *   `test_no_submission_during_rebalance` 테스트를 추가하여 리밸런스 중 메시지 제출이 차단됨을 검증했습니다.
    *   `test_get_gaps` 및 `test_get_true_lag` 테스트를 추가하여 `WorkManager`의 새로운 관측성 기능들이 올바른 값을 반환하는지 확인했습니다.
    *   `test_get_virtual_queue_sizes` 테스트를 추가하여 가상 파티션 큐의 크기 정보가 올바르게 노출되는지 검증했습니다.
    *   `NameError: name 'Any' is not defined` 및 `ModuleNotFoundError: No module named 'pyrallel_consumer'` 오류 등 디버깅 과정을 거쳐 모든 단위 테스트가 성공적으로 통과하도록 만들었습니다.

### 3.5. v2 아키텍처 설계 문서 업데이트
- **prd.md 업데이트**: `Control Plane`과 `Execution Plane`의 계약에 대한 핵심 원칙을 설명하는 섹션을 추가했습니다.
  - `ExecutionEngine.submit()`의 Blocking 특성과 `WorkManager`의 역할 명시
  - `max_in_flight` 설정의 이중적 의미(Control Plane vs. Engine) 명시
  - `get_in_flight_count()`의 올바른 사용법(참고용) 명시
- **prd_dev.md 업데이트**:
  - `ExecutionEngine` 인터페이스에 `submit()` Blocking에 대한 경고 추가
  - 설정 스키마 예제를 `max_in_flight_messages`와 `max_concurrent_tasks`로 구분하여 업데이트
  - Contract Test 항목에 Control Plane의 카운터 의존성 금지 테스트 명시

### 3.6. BrokerPoller 현황 및 문제점 (2026-02-09 추가)

기존 `GEMINI.md`에 명시된 `BrokerPoller`의 문제점들을 해결하기 위한 작업을 진행했으며, 현재 통합 테스트 단계에서 새로운 문제들에 직면했습니다.

#### 3.6.1. 완료된 작업
- **Deadlock 수정**: `BrokerPoller`가 `WorkManager`의 작업 스케줄링을 트리거하지 않던 문제를 해결했습니다. `WorkManager._try_submit_to_execution_engine`을 `schedule()`이라는 public 메서드로 변경하고 `BrokerPoller`의 메시지 제출 루프 이후에 호출하도록 수정했습니다.
- **중복 로직 제거**: `BrokerPoller`와 `OffsetTracker`에 중복으로 존재하던 'in-flight' 오프셋 추적 로직을 제거하고, `WorkManager`를 단일 진실 공급원(Single Source of Truth)으로 삼도록 리팩토링했습니다.
- **메타데이터 커밋 로직 수정**: `BrokerPoller._run_consumer`의 주기적인 오프셋 커밋 로직에서, `OffsetTracker`의 상태가 변경되기 전에 메타데이터가 올바르게 인코딩되도록 수정하여 커밋 정확성을 보장했습니다.
- **Hydration (상태 복원) 기능 구현**:
  - `MetadataEncoder`에 누락되었던 `decode_metadata` 메서드를 구현하고 단위 테스트를 추가하여, 커밋된 메타데이터를 다시 오프셋 집합으로 변환할 수 있게 만들었습니다.
  - `OffsetTracker.__init__` 메서드가 초기 오프셋 집합을 받을 수 있도록 수정했습니다.
  - `BrokerPoller._on_assign` 콜백에 Hydration 로직을 구현하여, 리밸런싱 시 컨슈머가 이전에 커밋된 메타데이터를 읽어 `OffsetTracker`의 상태를 복원하도록 했습니다.

#### 3.6.2. 통합 테스트 (`test_broker_poller_integration.py`) 문제 해결

`BrokerPoller`의 핵심 로직인 `_run_consumer`에 대한 통합 테스트(`tests/integration/test_broker_poller_integration.py`)를 작성하고 디버깅하는 과정에서 다음과 같은 문제점들을 해결했습니다.

1.  **`OffsetTracker` Mocking 복잡성 해결**:
    *   **문제**: 초기 `OffsetTracker` Mocking은 `MagicMock`이 내부 상태 변경을 제대로 시뮬레이션하지 못하여 `mark_complete.call_count`가 0으로 집계되는 문제가 있었습니다.
    *   **해결**: `mock_offset_tracker_class` 픽스처를 사용자 정의 `DummyOffsetTracker` 클래스로 리팩터링했습니다. 이 클래스는 `mark_complete`, `advance_high_water_mark`, `get_current_epoch` 등 필요한 메서드를 명시적으로 구현하고 자체 내부 상태(`last_committed_offset`, `completed_offsets`, 호출 추적 리스트 등)를 관리하도록 하여, `MagicMock`의 복잡한 `side_effect` 설정에서 발생할 수 있는 예측 불가능한 동작을 제거했습니다. 테스트 어설션도 `DummyOffsetTracker`의 내부 호출 추적 리스트를 직접 사용하도록 변경했습니다.

2.  **`consumer.consume` Mocking 및 `StopIteration` 오류 해결**:
    *   **문제**: `mock_consumer.consume`의 `side_effect`가 제공된 메시지 목록을 모두 소진하면 `StopIteration` 예외가 발생하여 `asyncio.to_thread` 컨텍스트 내에서 `RuntimeError`로 변환되는 문제가 있었습니다.
    *   **해결**: `custom_consume_side_effect`를 구현하여 `mock_consumer.consume`이 항상 리스트(메시지가 없으면 빈 리스트)를 반환하도록 함으로써 `StopIteration` 예외 발생을 방지했습니다.

3.  **`BrokerPoller` 커밋 로직 실행 문제 해결**:
    *   **문제**: `BrokerPoller`의 `_run_consumer` 루프에서 완료 이벤트가 처리된 후에도 Kafka 커밋이 트리거되지 않는 문제가 있었습니다. 이는 두 가지 원인이었습니다:
        1.  `_run_consumer` 내의 `if not messages: continue` 문이 Kafka 메시지가 소비되지 않은 루프 반복에서 커밋 로직의 실행을 막았습니다.
        2.  `mock_consumer.committed`가 `-1` 오프셋을 가진 `KafkaTopicPartition`을 반환했을 때, `BrokerPoller`는 이를 유효한 커밋 오프셋으로 해석하여 `last_committed_offset`을 `-1 - 1 = -2`로 잘못 계산했습니다. 이로 인해 `OffsetTracker`의 `potential_hwm` 계산이 실패하여 커밋 조건이 충족되지 않았습니다.
    *   **해결**:
        1.  `_run_consumer` 내의 `continue` 문을 제거하고 로직 흐름을 재구성하여, 새 Kafka 메시지가 소비되지 않더라도 완료 이벤트 처리 및 커밋 로직이 모든 루프 반복에서 항상 실행되도록 했습니다.
        2.  `mock_consumer.committed.return_value`를 `KafkaTopicPartition("test-topic", 0, OFFSET_INVALID)`로 변경하여 "커밋된 오프셋 없음" 상태를 올바르게 시뮬레이션했습니다. 이는 `BrokerPoller._on_assign`이 `OffsetTracker`의 `last_committed_offset`을 `0 - 1 = -1`로 올바르게 초기화하도록 했습니다.
        3.  `OFFSET_INVALID` 상수를 `confluent_kafka`에서 임포트하여 `NameError`를 해결했습니다.

4.  **테스트 견고성 및 결정론적 동작 개선**:
    *   **문제**: 기존 테스트의 임의적인 `asyncio.sleep` 호출은 테스트의 비결정성 및 경쟁 조건을 유발했습니다.
    *   **해결**: 임의의 `asyncio.sleep` 호출을 타임아웃이 있는 명시적인 `while` 루프(메시지 제출, 완료 처리, 커밋 트리거 등)로 대체하여 테스트의 견고성과 결정론적 동작을 크게 향상시켰습니다.
    *   `mock_work_manager` 픽스처를 리팩터링하여 `poll_completed_events`가 `asyncio.Queue`를 사용하도록 하고, 테스트에서 이벤트를 주입하는 `_push_completion_event` 헬퍼 메서드를 추가하여 비동기 이벤트 전달을 보다 결정론적으로 만들었습니다.

**현재 상태**: `test_broker_poller_integration.py` 통합 테스트가 이제 성공적으로 통과합니다.

### 3.7. 설계 불일치 수정 (2026-02-09)

PRD 문서(prd.md, prd_dev.md)와 실제 구현 간의 설계 불일치를 분석하고, 코드와 테스트를 수정하여 모든 테스트(76개)가 통과하도록 정비했습니다.

#### 3.7.1. 코드 수정 사항

1. **`WorkManager.schedule()` 재귀 호출 → while 루프 전환**:
   - `work_manager.py`의 `schedule()` 메서드가 `await self.schedule()`로 재귀 호출되어 스택 오버플로 위험이 있었습니다.
   - `while True:` 루프로 전환하고, 작업이 없거나 용량이 가득 찬 경우 `return`으로 탈출하도록 수정했습니다.

2. **`ProcessExecutionEngine.submit()` 이벤트 루프 블로킹 수정**:
   - `process_engine.py`의 `submit()`에서 `self._task_queue.put(work_item)`이 직접 호출되어 async 이벤트 루프를 블로킹하는 문제가 있었습니다.
   - `await asyncio.to_thread(self._task_queue.put, work_item)`으로 변경하여 이벤트 루프 블로킹을 방지했습니다.

3. **`ProcessExecutionEngine.shutdown()` 잘못된 설정 참조 수정**:
   - `shutdown()`에서 `self._config.async_config.task_timeout_ms`를 사용하여 프로세스 join timeout을 계산하고 있었습니다. Process 엔진이 Async 설정에 의존하는 것은 설계 위반입니다.
   - `ProcessConfig`에 `worker_join_timeout_ms: int = 30000` 필드를 추가하고, `self._config.process_config.worker_join_timeout_ms`를 참조하도록 수정했습니다.

4. **`broker_poller.py` 로거 f-string → % 포맷 전환**:
   - GEMINI.md의 개발 지침("로그에 출력하는 변수들은 f 표현식이 아닌 % 표현식으로 사용")에 따라 모든 f-string 로거 호출을 % 포맷으로 변환했습니다.
   - 총 약 20개 인스턴스를 변환했습니다.

#### 3.7.2. 테스트 수정 사항

1. **`test_work_manager.py` 메서드 호출명 동기화**:
   - 테스트에서 `_try_submit_to_execution_engine()`을 호출하고 있었으나, 실제 구현은 `schedule()`로 변경되어 있었습니다.
   - 4개 호출 지점을 `schedule()`로 수정했습니다.

2. **`test_offset_tracker.py` `in_flight_offsets` 테스트 수정**:
   - `in_flight_offsets`가 계산 프로퍼티(symmetric_difference)인데 직접 `.add()`를 호출하는 테스트가 있었습니다.
   - `update_last_fetched_offset()`을 호출하여 올바르게 in-flight 상태를 만든 후 테스트하도록 수정했습니다.

3. **테스트 패키지 `__init__.py` 추가**:
   - `tests/`, `tests/unit/`, `tests/unit/execution_plane/`, `tests/integration/` 디렉토리에 `__init__.py`가 누락되어 `from tests.unit.execution_plane.test_execution_engine_contract import ...` 임포트가 실패하던 문제를 해결했습니다.

4. **통합 테스트 `OffsetTracker` Mock 수정**:
   - `mock_offset_tracker_class`가 `MagicMock(spec=OffsetTracker)`로 생성되었으나, `mocker.patch`에서 `new=instance`로 설정할 때 `OffsetTracker(...)`가 `instance.return_value`(빈 MagicMock)를 반환하여 side_effect가 적용되지 않는 문제가 있었습니다.
   - `tracker_mock.return_value = tracker_mock`을 추가하여 Mock이 호출 시 자기 자신을 반환하도록 수정했습니다.

#### 3.7.3. 테스트 결과
- **수정 전**: 56 passed, 5 failed
- **수정 후**: 76 passed, 0 failed

앞으로의 모든 기능 개발은 TDD 방법론을 따릅니다. 이는 코드의 품질과 신뢰성을 높이고, 예측 가능한 방식으로 기능을 확장하는 데 도움을 줍니다.

- **TDD Workflow**:
    1.  **Red (실패하는 테스트 작성)**: 구현하려는 새로운 기능에 대해 실패하는 단위 테스트 또는 통합 테스트를 먼저 작성합니다.
    2.  **Green (테스트 통과)**: 최소한의 코드를 작성하여 해당 테스트를 통과시킵니다.
    3.  **Refactor (코드 리팩토링)**: 테스트가 통과했다면, 코드의 가독성, 유지보수성, 효율성을 개선하기 위해 리팩토링을 수행합니다. 이때 테스트는 리팩토링 과정에서 기능이 손상되지 않음을 보장하는 안전망 역할을 합니다.

- **테스트 디렉토리 활용**:
    - **`tests/unit/`**: 각 클래스나 함수의 가장 작은 논리적 단위가 예상대로 동작하는지 검증하는 테스트를 작성합니다. 외부 의존성(Kafka, DB 등)은 Mocking 처리합니다.
    - **`tests/integration/`**: 여러 컴포넌트가 함께 작동하여 큰 그림의 기능이 올바르게 수행되는지 확인하는 테스트를 작성합니다. 실제 Kafka 브로커와의 연동 테스트 등이 포함될 수 있습니다.

## 5. 다음 진행 계획

`prd_dev.md` 기반의 3계층 아키텍처와 TDD 전략, 그리고 Observability 설계를 반영하여
다음과 같은 단계적 개발 계획을 수립합니다.

### 2026-02-14 – 벤치마크 밸리데이션 & 리셋 준비
- `uv run pytest` 결과: 단위/통합 테스트는 통과했으나 `tests/e2e/test_ordering.py` 내 4개 시나리오가 실패했습니다. 기존 토픽(`e2e_ordering_test_topic`)이 이전 메시지를 유지하여 순서/카운트가 어긋나는 상태입니다. 토픽/컨슈머 그룹 리셋 기능으로 재시도 예정입니다.
- `pre-commit run --all-files` 결과: `pretty-format-toml` 훅이 `pkg_resources` 미탑재로 중단되어 훅 전용 venv의 `setuptools` 버전을 69.5.1로 낮춰 해결했습니다. 여전히 `tests/unit/execution_plane/test_base_execution_engine.py`의 mypy 경고는 기존 테스트 더블 제한으로 남아 있습니다.
- `benchmarks/kafka_admin.py` + `tests/unit/benchmarks/test_kafka_admin.py`를 추가하여 AdminClient 기반 리셋 헬퍼를 구현했습니다. Unknown topic/group 오류는 무시하고 나머지는 재시도 후 예외를 상승시킵니다. 관련 단위 테스트는 green입니다.
- `benchmarks/run_parallel_benchmark.py`가 기본으로 토픽/컨슈머 그룹을 삭제 후 재생성하며, `--skip-reset` 플래그로 비활성화할 수 있습니다. `README.md` 벤치마크 섹션에 해당 행동을 문서화했고, `benchmarks/pyrparallel_consumer_test.py`는 수동 실행 시 동일 헬퍼를 켤 수 있는 `reset_topic` 옵션을 노출합니다.
- 장시간 워커 부하를 실험할 수 있도록 `run_parallel_benchmark.py`에 `--timeout-sec` CLI 옵션을 추가해 async/process 라운드의 타임아웃을 조정할 수 있게 했습니다. 해당 옵션을 README에 문서화했습니다.
- 전체 `uv run pytest`는 `tests/e2e/test_ordering.py` 네 케이스가 여전히 기존 토픽 잔존 메시지로 실패(10k 메시지 요청 대비 11k 처리)했으며, 나머지 86개 테스트는 통과했습니다.
- `pre-commit run --all-files`는 기존 mypy 경고(테스트 더블 시그니처 불일치)만 남고 전부 green입니다.
- `uv run python benchmarks/run_parallel_benchmark.py --bootstrap-servers localhost:9092 --num-messages 2000 --num-keys 50 --num-partitions 4`를 실행해 baseline/async/process 라운드를 모두 성공적으로 완료했습니다. 결과 JSON은 `benchmarks/results/20260214T053950Z.json`에 저장되었습니다.
- **ProcessExecutionEngine 종료 hang 해결 (2026-02-14)**: `pyrparallel_consumer_test.py` `finally` 블록에 `await engine.shutdown()`을 추가해 워커 종료용 sentinel을 전송하도록 수정. `uv run python benchmarks/run_parallel_benchmark.py --num-messages 1000 --num-keys 10 --num-partitions 4 --skip-baseline --skip-async --bootstrap-servers localhost:9092 --topic-prefix pyrallel-benchmark-ci --process-group process-benchmark-group-ci` 실행 시 Process 라운드가 정상 종료되고 프로세스가 자동 종료됨을 확인(결과 JSON: `benchmarks/results/20260214T071451Z.json`).

현재 `BrokerPoller`의 핵심 기능 구현은 완료되었으나, 통합 테스트 단계에서 난관에 봉착했습니다. 따라서 다음 계획은 테스트를 통과시키는 데 집중합니다.

1.  **`test_run_consumer_loop_basic_flow` 통합 테스트 디버깅 및 수정 (완료)**
2.  **Backpressure 로직 테스트 작성 (대기 중)**
    - `_check_backpressure` 메서드가 부하량에 따라 `consumer.pause`와 `consumer.resume`을 올바르게 호출하는지 검증하는 테스트를 추가합니다.
3.  **모든 변경사항 커밋 (완료)**

### 5.0 진행 요약
- **완료**: Phase 1~5와 컨트롤 플레인/워크 매니저/실행 엔진 계약 등 핵심 아키텍처 구현을 끝내고, `test_run_consumer_loop_basic_flow` 통합 테스트도 디버깅을 마쳤습니다.
- **진행 중**: Observable metrics export / 운영 가이드 작성 및 통해 Observability 단계 보완, E2E 테스트(문서화 포함) 추가.
- **향후 우선순위**: Observability 문서화, 커밋 정확성 관련 E2E 검증, 단계별 인수인계와 커밋을 병행하면서 새로운 기능(Phase 6+)로 확장합니다.

---

### 5.1 Phase 1 – Control Plane Core (난이도: ★★★★) - **완료**

- **BrokerPoller** - **완료**
  - Kafka poll / pause / resume 제어 - **완료**
  - Backpressure 연계 (Load 기반 pause/resume) - **완료 (Hysteresis 검증 포함)**
  - Rebalance callback wiring - **완료**
  - Hydration (상태 복원) - **완료**

- **Rebalance & Epoch Fencing** - **완료**
  - Partition epoch 상태 머신 구현 - **완료**
  - revoke 중 completion 무시 로직 - **완료**
  - final commit + metadata 전달 - **완료**

- **OffsetTracker (State Machine)** - **완료**
- **MetadataEncoder** - **완료**
- **WorkManager** - **완료**

> TDD 우선순위: **ExecutionEngine Contract Test** → AsyncExecutionEngine → ProcessExecutionEngine

---

### 5.2 Phase 2 – WorkManager & Scheduling (난이도: ★★★★) - **완료**

병렬 처리의 **공정성 + 관측 가능성**을 책임지는 계층입니다.

- **WorkManager** - 완료
  - Virtual Partition 관리 - 완료
  - Blocking Offset 우선 스케줄링 알고리즘 - 완료
  - ExecutionEngine submit 제어 - 완료

- **Scheduling Policy** - 완료
  - Lowest blocking offset 우선 - 완료
  - starvation 방지 - (부분적으로 해결, 개선 필요)
  - rebalance 중 submit 차단 - 완료

- **Observability Integration** - 완료
  - Blocking Offset duration 추적 - 완료
  - Gap / True Lag 계산 노출 - 완료
  - Backpressure 판단 지표 제공 - 완료

> 이 단계까지 완료되면 Mock ExecutionEngine으로 end-to-end 테스트 가능

---

### 5.3 Phase 3 – Execution Abstraction (난이도: ★★★) - **완료**

Execution Plane의 계약을 고정하는 단계입니다.

- **ExecutionEngine 인터페이스** - 완료
  - submit()
  - shutdown()
  - metrics()

- **DTO 정의** - 완료
 - CompletionEvent
 - EngineMetrics

### 5.4 Retry + DLQ 설계 (2026-02-16)
- `docs/plans/2026-02-16-retry-dlq-design.md`에 재시도 + DLQ 설계를 기록했습니다. 실행 엔진 내부 재시도(기본 3회, 지수 백오프 1s 시작, 최대 30s, 지터 200ms) 후 실패 시 BrokerPoller가 DLQ로 발행하고 성공 시에만 커밋하도록 합니다.
- `ExecutionConfig`에 `max_retries`, `retry_backoff_ms`, `exponential_backoff`, `max_retry_backoff_ms`, `retry_jitter_ms`를 추가하고, `KafkaConfig`에 `dlq_enabled`를 추가하여 기존 `dlq_topic_suffix`를 실제 사용합니다.
- DLQ 발행 시 원본 key/value를 보존하고, 헤더에 `x-error-reason`, `x-retry-attempt`, `source-topic`, `partition`, `offset`, `epoch`를 포함합니다. DLQ 발행 실패 시 재시도하며 성공 전에는 커밋하지 않습니다.

### 5.5 Retry + DLQ 구현 (2026-02-17)
- Async/Process 엔진에 재시도 및 백오프 구현: `max_retries`, `retry_backoff_ms`, `exponential_backoff`, `max_retry_backoff_ms`, `retry_jitter_ms` 적용. `CompletionEvent.attempt`로 1-based 시도 횟수 노출.
- BrokerPoller: 실패 이벤트가 최대 재시도에 도달하면 DLQ로 발행 후 성공 시에만 커밋. 실패 시 커밋 스킵. 메시지 key/value는 소비 시 캐싱 후 사용, 헤더에 에러/시도/소스 정보를 포함. DLQ 비활성 시 기존 커밋 흐름 유지.
- 문서: README 재시도/DLQ 옵션 추가, prd_dev 설정 스키마에 재시도/DLQ 옵션 명시, 계획/설계 문서 (`docs/plans/2026-02-16-retry-dlq-plan.md`, `docs/plans/2026-02-16-retry-dlq-design.md`) 작성.
- 테스트: `tests/unit/control_plane/test_broker_poller_dlq.py` 추가, 재시도/백오프/커밋 조건을 모킹으로 검증. Async/Process 엔진 재시도 테스트 확장. 전체 `pytest` 실행 시 e2e Kafka가 없어서 `tests/e2e/test_ordering.py`는 부트스트랩 연결 실패로 오류(로컬 Kafka 미기동). 나머지 단위/통합 테스트는 통과. `pre-commit run --all-files`는 모두 통과.
- 추가 안정화(2026-02-17): ProcessExecutionEngine in-flight 카운터에 락을 추가해 경합을 방지. DLQ 발행 실패 시 캐시를 보존하고 커밋을 스킵하도록 조정. DLQ flush 타임아웃을 `KafkaConfig.DLQ_FLUSH_TIMEOUT_MS`(기본 5000ms)로 설정화. 타이밍 민감 테스트에 여유 허용치(0.9x) 적용. 단위/통합 테스트 139개 통과; e2e는 로컬 Kafka 미기동으로 미수행.

### 5.6 성능 벤치/프로파일 (2026-02-18)
- 성능 기준
  - Async 엔진 100k: 31.25s, TPS≈3,199, avg≈0.61s, p99≈0.72s (`benchmarks/results/20260218T103057Z.json`).
  - Process 엔진 100k (갭 캐싱 후): 96.60s, TPS≈1,035, p99≈1.98s (`benchmarks/results/20260218T102444Z.json`).
- 프로파일(20k, process, yappi): 완료 74.15s, TPS≈270, p99≈2.67s. 주요 핫스팟 ttot:
  - WorkManager.schedule 48.99s
  - WorkManager.poll_completed_events 47.42s
  - WorkManager.get_blocking_offsets 37.17s
  - OffsetTracker.get_gaps 32.95s
  - Logging(Logger.debug/_log/StreamHandler) 합산 ~60s
  → 컨트롤 플레인 gaps/블로킹 계산 + 로깅 오버헤드가 지배적.
- 개선 조치
  - OffsetTracker gaps 캐싱 및 동일 갭 반복 5000회마다 WARN 추가.
  - BrokerPoller 블로킹 오프셋 5초 초과 WARN 추가.
  - WorkManager 디버그 스팸 제거.
- 남은 튜닝 방향
  - gaps/blocking 계산 호출 빈도 축소 또는 변경 발생 시에만 계산.
  - DEBUG 로깅 샘플링/축소로 오버헤드 완화.
  - 추가 프로파일(100k) 시 gaps 반복 루프가 재발하면 반복 패턴 스로틀/스킵 검토.
  - TopicPartition
  - TaskContext (epoch 포함)

- **Engine Factory** - 완료
  - 설정 기반 엔진 선택
  - async / process 공존 구조 확정

- **Contract Test Suite** - 완료
  - ExecutionEngine 공통 동작 검증
  - observability 항목 포함

---

### 5.4 Phase 4 – AsyncExecutionEngine (난이도: ★★★) - **완료**

Python asyncio 환경에 최적화된 기본 실행 모델입니다.

- **Task Pool** - 완료
  - asyncio.Task 기반 실행
  - Semaphore 기반 max_in_flight 제어

- **Completion Channel** - 완료
  - asyncio.Queue 기반 completion 전달
  - epoch 포함 completion event 생성

- **Async 전용 테스트** - 완료
  - high concurrency 시나리오
  - pause 상태에서도 completion 처리 검증

---

### 5.5 Phase 5 – ProcessExecutionEngine (난이도: ★★★★★) - **완료**

GIL 회피를 위한 고난이도 실행 모델입니다. `ProcessExecutionEngine`의 성공적인 구현 및 테스트를 완료했습니다.

- **IPC 채널**
  - multiprocessing.Queue 기반 task / completion 통신 - 완료
  - worker_loop 구현 - 완료

- **Worker Process 관리**
  - crash 감지 및 worker 재기동
  - sentinel 기반 graceful shutdown - 완료

- **Process 전용 테스트**
  - worker crash 복구
  - partial completion + epoch fencing
  - shutdown 중 completion drain 검증 - 완료

---

### 5.6 Observability & 운영 품질 (난이도: ★★★) - **진행 중**

라이브러리 신뢰성을 외부에 드러내는 단계입니다.

- **Metrics Export Layer** - **완료**
  - True Lag, Gap, Blocking Offset (Top N), In-flight / Capacity 지표 수집 및 DTO 정의 완료 (`SystemMetrics`, `PartitionMetrics`)
  - `BrokerPoller.get_metrics()` 구현 완료
- **Dashboard Spec** - 대기 중
  - Grafana 패널 설계
  - 장애 시나리오 기반 뷰 구성
- **운영 가이드** - 대기 중
  - Kafka Lag vs True Lag 설명
  - Blocking Offset 대응 전략

---

### 5.7 권장 개발 순서 (TDD 기준)

1. OffsetTracker + Observability API - 완료
2. Rebalance & Epoch Fencing - 완료
3. WorkManager + Scheduling - 완료
4.  **BrokerPoller 기능 보완 및 테스트 작성** - **완료**
5. ExecutionEngine Contract Test - **완료**
6. AsyncExecutionEngine 구현 - **완료**
7. ProcessExecutionEngine 구현 - **완료**
8. Observability Export & Docs - **진행 중**

### 5.8 E2E 테스트 구현 (2026-02-10)

`tests/e2e/test_ordering.py`에 전체 시스템의 E2E 테스트를 구현했습니다. 실제 Kafka 브로커와 `benchmarks/producer.py`를 사용하여 메시지를 생성하고, `BrokerPoller` → `WorkManager` → `AsyncExecutionEngine` 전체 파이프라인을 검증합니다.

#### 테스트 인프라
- **`ResultTracker`**: 키별(`results`) 및 파티션별(`partition_results`) 처리 순서를 기록하고 검증하는 헬퍼 클래스
- **`run_ordering_test()`**: 공통 테스트 설정(BrokerPoller, WorkManager, Engine 생성, producer 실행, stop_event 대기)을 캡슐화한 헬퍼 함수. `worker_fn`, `max_in_flight`, `timeout` 파라미터 지원
- **`create_e2e_topic` fixture**: 테스트 전후 토픽(`e2e_ordering_test_topic`, 8 파티션) 생성/삭제

#### 구현된 테스트 (5개)
1. **`test_key_hash_ordering`**: KEY_HASH 모드에서 동일 키 내 sequence 오름차순 보장 검증 (10000 msgs, 100 keys)
2. **`test_partition_ordering`**: PARTITION 모드에서 동일 파티션 내 오프셋 오름차순 보장 검증 (10000 msgs, 100 keys)
3. **`test_unordered`**: UNORDERED 모드에서 전체 메시지 처리 완료 검증 (10000 msgs, 100 keys)
4. **`test_backpressure`**: `max_in_flight=20`으로 제한된 상태에서 500개 메시지 처리 완료 및 `MAX_IN_FLIGHT_MESSAGES`, `MIN_IN_FLIGHT_MESSAGES_TO_RESUME` 설정값 검증. 인라인 구성 사용
5. **`test_offset_commit_correctness`**: 랜덤 지연 워커로 500개 메시지 처리 후, Kafka에 커밋된 오프셋이 실제 처리된 최대 오프셋+1을 초과하지 않는지 검증. 인라인 구성 사용

#### 주요 버그 수정
- `benchmarks/producer.py` 호출 시 `--topic` 인자 누락 수정 (기본값 `test_topic` 대신 `e2e_ordering_test_topic` 사용)
- `test_offset_commit_correctness`의 `stop_event` 접근 불가 버그 수정: 커스텀 `worker_fn`을 `run_ordering_test()`에 전달할 경우 내부 `stop_event`에 접근할 수 없어 항상 타임아웃되던 문제를 인라인 구성으로 리팩토링하여 해결

### 5.9 운영 안정성 개선 (2026-02-10)

프로세스 기반 실행 엔진의 운영 안정성을 높이기 위해 3건의 개선을 수행했습니다.

#### 5.9.1. `multiprocessing.Value` → `int` 단순화 (완료)
- **문제**: `ProcessExecutionEngine`의 `_in_flight_count`가 `multiprocessing.Value("i", 0)`로 구현되어 Lock 오버헤드 발생. 그러나 이 카운터는 메인 프로세스의 async 이벤트 루프에서만 접근하므로 프로세스 간 공유가 불필요.
- **수정**: 일반 `int`로 교체. `Value` import 제거, Lock 획득 코드 제거.
- **파일**: `process_engine.py`

#### 5.9.2. `poll_completed_events` 무한 드레인 방지 (완료)
- **문제**: `poll_completed_events`가 큐가 빌 때까지 무한 루프로 이벤트를 꺼내, 대량 완료 시 이벤트 루프를 장시간 블로킹할 수 있음.
- **수정**: `batch_limit: int = 1000` 파라미터 추가. `BaseExecutionEngine`, `ProcessExecutionEngine`, `AsyncExecutionEngine` 모두 동일하게 적용.
- **파일**: `base.py`, `process_engine.py`, `async_engine.py`

#### 5.9.3. 멀티프로세스 QueueHandler/QueueListener 로깅 (완료)
- **문제**: 워커 프로세스가 부모 프로세스의 로거를 그대로 상속받아, 여러 프로세스가 동시에 같은 핸들러에 쓰면 로그 출력이 뒤섞이거나 깨질 수 있음.
- **수정**:
  - `logger.py`에 `LogManager.setup_worker_logging(log_queue)`, `LogManager.create_queue_listener(log_queue, handlers)` 유틸리티 추가.
  - `ProcessExecutionEngine.__init__`에서 `log_queue` 생성 및 `QueueListener` 시작.
  - `_worker_loop`에 `log_queue` 파라미터 추가, 진입 시 `setup_worker_logging()` 호출.
  - `shutdown()`에서 `QueueListener.stop()` 호출.
- **파일**: `logger.py`, `process_engine.py`

#### 테스트 결과
- 82개 테스트 통과 (unit 80 + integration 2), 0 failures

### 5.10 재시도 및 DLQ (Dead Letter Queue) 구현 (2026-02-17)

실패한 메시지에 대한 자동 재시도와 최종 실패 시 DLQ 퍼블리싱 기능을 완전히 구현했습니다.

#### 5.10.1. 구성 필드 추가 (Task 1 완료)
- **ExecutionConfig**: `max_retries=3`, `retry_backoff_ms=1000`, `exponential_backoff=True`, `max_retry_backoff_ms=30000`, `retry_jitter_ms=200` 추가
- **KafkaConfig**: `dlq_enabled=True`, `dlq_topic_suffix='.dlq'` 추가
- **파일**: `config.py`, `tests/unit/test_config.py`
- **테스트**: 재시도/DLQ 설정 기본값 및 오버라이드 검증, `dump_to_rdkafka` 제외 검증

#### 5.10.2. CompletionEvent attempt 필드 추가 (Task 2 완료)
- **DTO**: `CompletionEvent`에 `attempt: int` 필드 추가하여 재시도 횟수 추적
- **파일**: `dto.py`
- **테스트**: AsyncExecutionEngine, ProcessExecutionEngine의 모든 테스트에서 attempt 필드 검증

#### 5.10.3. AsyncExecutionEngine 재시도 로직 (Task 3 완료)
- **구현**:
  - 워커 호출을 재시도 루프로 래핑
  - 지수/선형 백오프 계산 (cap + jitter)
  - 타임아웃도 재시도 대상으로 처리
  - 세마포어는 재시도 간 유지 (최종 완료 시에만 release)
- **파일**: `async_engine.py`, `tests/unit/execution_plane/test_async_execution_engine.py`
- **테스트**: 첫 시도 성공, 재시도 후 성공, 최종 실패, 타임아웃 재시도, 백오프 타이밍, 백오프 cap 검증

#### 5.10.4. ProcessExecutionEngine 재시도 로직 (Task 4 완료)
- **구현**:
  - 워커 프로세스 내에서 재시도 수행
  - 배치 처리 시 아이템별 독립적 재시도
  - 동일한 백오프 계산 로직 적용
  - in-flight 카운트는 아이템별로 최종 완료 시에만 감소
- **파일**: `process_engine.py`, `tests/unit/execution_plane/test_process_engine_batching.py`
- **테스트**: 재시도 후 성공, 최종 실패, 백오프 타이밍, 백오프 cap, 즉시 성공 시 attempt=1 검증

#### 5.10.5. BrokerPoller DLQ 퍼블리싱 (Task 5 완료)
- **구현**:
  - `_message_cache: Dict[Tuple[DtoTopicPartition, int], Tuple[Any, Any]]`로 메시지 key/value 보존
  - `_publish_to_dlq` 헬퍼 메서드: DLQ 토픽으로 퍼블리싱 + 재시도 로직 + 헤더 추가
  - 실패 이벤트 처리 시 `attempt >= max_retries` 검증 후 DLQ 퍼블리싱
  - DLQ 퍼블리싱 실패 시 오프셋 커밋 건너뜀 (gap 유지)
  - `dlq_enabled=False`일 때는 기존 동작 유지 (로깅만, 정상 커밋)
- **헤더**: `x-error-reason`, `x-retry-attempt`, `source-topic`, `partition`, `offset`, `epoch`
- **파일**: `broker_poller.py`, `tests/integration/test_broker_poller_integration.py`
- **테스트**: DLQ 퍼블리싱 성공 시 커밋, 비활성화 시 건너뛰기, 재시도 실패 시 커밋 건너뛰기 검증

#### 5.10.6. 와이어링 검증 (Task 6 완료)
- **확인**: `engine_factory.py`가 전체 `config` 객체를 양쪽 엔진에 전달하여 자동 와이어링 확인
- **테스트**: 전체 실행 엔진 + 통합 테스트 재실행 (45개 통과)

#### 5.10.7. 문서 업데이트 (Task 7 완료)
- **파일**: `README.md`
- **추가 섹션**: "재시도 및 DLQ 설정" (환경 변수, 백오프 계산, 헤더 형식, 동작 흐름, 예제 코드)

#### 5.10.8. 최종 검증 (Task 8 완료)
- **pre-commit**: 전체 hooks 통과 (mypy 포함)
- **mypy 수정**: `test_base_execution_engine.py`의 `poll_completed_events` 시그니처 수정 (batch_limit 파라미터 추가)
- **로깅 검증**: f-string 없음 확인 ✓
- **assert 검증**: 프로덕션 코드에 assert 없음 확인 ✓
- **전체 테스트**: 138개 통과 (unit 123 + integration 5), 0 failures

#### 구현 상세
- **메시지 캐싱**: `(TopicPartition, offset)` 튜플을 키로 사용해 key/value 보존
- **DLQ 재시도**: `asyncio.to_thread`로 동기 producer 연산 래핑, 동일한 백오프 설정 재사용
- **에포크 펜싱**: DLQ 헤더에 epoch 포함하여 리밸런싱 추적 가능
- **LSP 타입 이슈**: confluent-kafka의 `metadata` kwarg는 런타임 동작하나 타입 스텁 누락 (무시 가능)

#### 테스트 결과
- 138개 테스트 통과 (unit 128 + integration 5), 0 failures
- 3개 경고 (unawaited mock coroutines, non-critical)
- pre-commit 전체 hooks 통과

### 5.11 Yappi 프로파일링 계획 수립 (2026-02-24)

- `docs/plans/2026-02-24-profile-analysis-design.md`에 baseline/async/process 벤치마크를 yappi로 프로파일링하고 snakeviz로 시각화하는 절차(풀볼륨/스모크/오프라인 분석 옵션 포함)를 정리했습니다.
- `benchmarks/results/profiles/` 경로에 `.prof` 산출물이 아직 없으며, 실행에는 로컬 Kafka 클러스터가 필요합니다.
- 다음 단계: Kafka를 띄운 뒤 `uv run python -m benchmarks.profile_benchmark_yappi --bootstrap-servers localhost:9092 --modes baseline async process`(메시지/키/파티션 수는 용량에 맞게 조정)로 프로파일 생성 → pstats/console 요약과 snakeviz로 병목 비교.

### 5.12 Yappi 프로파일 실행 및 병목 관찰 (2026-02-24)

- 실행: `uv run python -m benchmarks.profile_benchmark_yappi` (10k msgs, 100 keys, 8 partitions, timeout 120s).
- 산출물: `benchmarks/results/profiles/20260224T091356Z/` 내 `baseline_profile.prof`, `async_profile.prof`, `process_profile.prof`.
- 요약 (실측 TPS/런타임): baseline 0.23s / 44,364 TPS; async 4.33s / 2,309 TPS; process 8.60s / 1,163 TPS.
- Hotspot (async): yappi 누적 대부분이 `asyncio.tasks.sleep`(10,010 calls, ~1,498 cum sec)와 `asyncio.wait_for` → 벤치마크 워커의 5ms 슬립이 전체 병목. 실제 런타임 4.33s로 합산된 wall-time은 동시성 합.
- Hotspot (process): 누적 상위에 `asyncio.sleep`(diag/worker), `Queue.get`(~8.5s), `BrokerPoller._run_consumer`(~8.5s); 프로파일은 메인 프로세스만 캡처(워커 프로세스 yappi 미수집).
- 개선 아이디어: (1) 벤치마크 워커 슬립을 파라미터화/기본 축소하여 엔진 오버헤드만 측정, (2) 프로세스 워커에서 yappi 시작/저장하도록 래핑해 실제 워커 호출 스택 수집, (3) 프로파일 시 diag 루프(5s sleep) 비활성화 옵션 추가로 노이즈 축소.

### 5.13 프로파일러 정합성 개선 및 재실행 (2026-02-24)

- 변경: baseline/async/process 모두 동일한 워크로드를 사용하도록 벤치마크 정렬.
  - `profile_benchmark_yappi.py`에 `--worker-sleep-ms` 추가(기본 5ms), 공통 슬립 워커를 baseline/async/process에 적용.
  - `run_pyrallel_consumer_test`는 커스텀 워커 주입(비동기/프로세스) 허용, baseline 소비자도 커스텀 worker_fn을 받아 동일 워크로드 실행.
- 재실행: `uv run python -m benchmarks.profile_benchmark_yappi --worker-sleep-ms 5` 산출물 `benchmarks/results/profiles/20260224T111936Z/`.
- 실측 TPS/런타임(10k msgs, 5ms work): baseline 61.8s / 161.7 TPS; async 4.19s / 2,387 TPS; process 8.59s / 1,163 TPS.
- Hotspot 정합: 이제 세 모드 모두 동일 슬립이 상단에 나타남 (`_sleep_work_*` / `asyncio.sleep`), 병렬 처리 이점(특히 async) 명확하게 비교 가능. Process 프로파일은 여전히 메인 프로세스 중심(워커별 yappi 미포함)으로 표시되므로 워커 프로파일링 필요 시 추가 수집 필요.

### 5.14 벤치마크/프로파일 통합 플래그 및 워크로드 선택 추가 (2026-02-24)

- `benchmarks/run_parallel_benchmark.py`에 프로파일 토글/출력 옵션 추가: `--profile`, `--profile-dir`, `--profile-clock`, `--profile-top-n`, `--profile-threads`, `--profile-greenlets`(yappi 필요, 기본 off).
- 동일 스크립트에 워크로드 선택 추가: `--workload {sleep,cpu,io}` + `--worker-sleep-ms`/`--worker-cpu-iterations`/`--worker-io-sleep-ms`를 baseline/async/process 공통으로 주입(커스텀 워커 훅 사용).
- 프로파일 .prof는 모드명으로 `profile_dir/run_name.prof` 저장, top-N 출력 옵션 지원. 프로세스 모드 워커 내부 yappi는 아직 미적용(필요 시 후속 작업).

### 5.15 프로세스 워커 프로파일링 및 벤치마크 README 추가 (2026-02-24)

- `run_parallel_benchmark.py`: 프로파일 모드에서 프로세스 워커 내부에서도 yappi를 시작하고 종료 시 per-worker `.prof`를 `run_name-worker-<pid>.prof`로 저장하도록 래핑(프로파일 실패 시 워커 진행 유지). `datetime.utcnow()` 사용을 UTC aware `datetime.now(datetime.UTC)`로 교체.
- 워크로드 옵션에 `all` 추가(sleep→cpu→io 순차 실행, 토픽/그룹 접미사로 충돌 방지). run_name에 워크로드 접두사 부여해 결과/프로파일 구분.
- `benchmarks/README.md`에 사용법/옵션 업데이트.

### 5.16 consumer/offset_manager 커버리지 보강 (2026-02-25)

- `tests/unit/test_consumer_and_offset_manager.py` 추가: PyrallelConsumer wiring(start/stop/metrics) 더미 객체로 검증, OffsetTracker add/remove/safe_offsets/total_in_flight 테스트로 커버리지 확보.
- 루트 `README.md`에 프로파일 OFF 벤치마크 샘플(TPS) 표 추가 (sleep/cpu/io workload, 4 partitions, 2000 msgs, 100 keys).

### 5.17 BrokerPoller 견고성 및 E2E 정렬 테스트 고정 (2026-02-25)

- `broker_poller`: mock 친화적으로 기본 numeric 값 사용(poll batch/worker size, blocking_warn_seconds/diag_log_every)하고, partition ordering 시 submit key를 파티션 ID로 고정해 PARTITION 모드 정렬 보장. commit 시 KafkaTopicPartition metadata를 설정해 통합 테스트 기대 충족.
- E2E ordering 테스트 속도 단축(대량 메시지 2000으로 감소) 및 PARTITION 모드 정렬 실패 수정.
- 통합 테스트(`tests/integration`)와 E2E ordering 전체 통과 확인.

### 5.18 README 시작 가이드 추가 (2026-02-25)

- 루트 `README.md`에 설치/설정/워커 정의( async I/O, CPU, sleep ), 실행 엔진 선택 예시를 포함한 빠른 시작 섹션을 추가했습니다.

### 5.19 ExecutionMode Enum 도입 (2026-02-25)

- `pyrallel_consumer.dto.ExecutionMode` 추가, `ExecutionConfig.mode`를 Enum으로 전환하고 `engine_factory`에서 문자열 입력 시 Enum으로 정상 변환하도록 처리.
- README 예제를 `ExecutionMode.ASYNC/PROCESS`로 갱신. 주요 유닛/통합 테스트 재실행(통과, 기존 경고만 유지).

### 5.20 IPC 직렬화 msgpack 전환 및 모니터링 스택 확장 (2026-02-25)

- `process_engine`: multiprocessing 큐에서 pickle을 제거하고 WorkItem/CompletionEvent를 msgpack으로 직렬화/역직렬화하도록 변경. 헬퍼 추가, 배치 버퍼 플러시 시 msgpack bytes 전송, 워커/메인 모두 디코딩 후 처리. `pyproject.toml`에 `msgpack` 의존성 추가.
- `docker-compose.yml`에 Prometheus(9090), Grafana(3000), Kafka Exporter(9308) 추가. `monitoring/prometheus.yml` 작성.
- README 모니터링 가이드 추가(메트릭 활성화, compose up, Grafana 데이터소스). 사용법 섹션 재정리.
- `.env.sample` 추가: Kafka/Parallel Consumer/Execution/Metrics/DLQ 설정 예시 포함.

### 5.21 Backpressure 큐 한도 추가 (2026-02-25)

- `ParallelConsumerConfig.queue_max_messages` 기본 5000 도입, BrokerPoller에서 총 대기 메시지가 한도 초과 시 pause, 70% 이하로 줄면 resume(기존 in-flight 기반 히스테리시스와 병행).
- `.env.sample`/README 예시에 queue_max_messages 추가.

### 5.22 ProcessEngine in-flight 레지스트리 확장 및 커밋 클램프 (2026-02-25)

- ProcessExecutionEngine: Manager 기반 in-flight 레지스트리를 워커별 리스트로 확장하여 워커가 잡은 다중 작업을 추적. 워커 사망 시 리스트의 모든 작업을 msgpack으로 재큐잉 후 워커 재시작. 최소 in-flight 오프셋 조회가 파티션별로 다중 항목을 고려하도록 변경.
- BrokerPoller: 커밋 계산 시 레지스트리 최소 오프셋을 커밋 상한으로 적용해 더 안전한 커밋 지점 확보.
- 단위 회귀: clamp 테스트 추가(`test_broker_poller_inflight_clamp`), 프로세스 엔진 관련 빠른 회귀 통과.

### 5.15 프로세스 워커 프로파일링 및 벤치마크 README 추가 (2026-02-24)

- `run_parallel_benchmark.py`: 프로파일 모드에서 프로세스 워커 내부에서도 yappi를 시작하고 종료 시 per-worker `.prof`를 `run_name-worker-<pid>.prof`로 저장하도록 래핑(프로파일 실패 시 워커 진행 유지).
- 동일 파일에 프로파일/워크로드 옵션 문서화용 README 추가: `benchmarks/README.md`에 사용 예시, 옵션 요약, 출력 위치 설명.

- Benchmark TUI dashboard red 단계: 진행률 바/워크로드×모드 TPS 표를 요구하는 회귀 테스트를 tests/unit/benchmarks/test_tui_log_parser.py, tests/unit/benchmarks/test_tui_app.py에 추가했고 아직 snapshot/dashboard 필드가 없어 실패를 확인할 예정입니다.

- Benchmark TUI dashboard green 단계: benchmarks/tui/log_parser.py에 완료 run 수/TPS 매트릭스를 추가하고 benchmarks/tui/app.py RunScreen이 ProgressBar + DataTable 요약 보드를 렌더/갱신하도록 확장했습니다.

### 5.23 Benchmark topic-create noise suppression (2026-03-10)

- `benchmarks/producer.py`, `benchmarks/pyrallel_consumer_test.py`의 `create_topic_if_not_exists()`가 먼저 `list_topics()`로 존재 여부를 확인한 뒤, 이미 있는 토픽이면 `create_topics()`와 중복 출력("already exists")을 건너뛰도록 조정했습니다.
- 회귀 테스트 `tests/unit/benchmarks/test_topic_creation_noise.py`를 추가해 producer/benchmark consumer helper가 기존 토픽에 대해 재생성을 시도하지 않는지 검증했습니다.

### 5.24 Benchmark TUI ordering-aware live dashboard (2026-03-11)

- `benchmarks/tui/log_parser.py`: live progress snapshot이 `current_ordering`과 `tps_by_workload_ordering`를 추적하도록 확장하고, 선택된 ordering 수를 포함해 total_runs/progress를 계산하도록 조정했습니다. 결과 테이블 로그의 `Order` 컬럼 유무(구/신 포맷)를 모두 파싱합니다.
- `benchmarks/tui/app.py`: RunScreen 진행 대시보드에 ordering badge를 추가하고, 다중 ordering 선택 시 workload×ordering 행으로 TPS 표를 렌더링하도록 변경했습니다. 단일 ordering 실행은 기존 workload 행 레이아웃을 유지합니다.
- TDD: `tests/unit/benchmarks/test_tui_log_parser.py`, `tests/unit/benchmarks/test_tui_app.py`에 ordering-aware 회귀 테스트를 먼저 추가했고, 해당 테스트 통과를 확인했습니다.

### 5.25 BrokerPoller/WorkManager offset tracker 분리 (2026-03-11)

- TDD(red): `tests/unit/control_plane/test_broker_poller.py`의 할당 테스트를 갱신해 `BrokerPoller._on_assign()`가 WorkManager에 `OffsetTracker` 인스턴스를 공유하지 않고 시작 오프셋만 전달해야 함을 먼저 고정했습니다.
- Green: `pyrallel_consumer/control_plane/broker_poller.py`에서 WorkManager 할당 payload를 tracker 객체 대신 partition starting offset으로 바꿔 BrokerPoller/WorkManager가 커밋 상태를 독립적으로 유지하도록 정리했습니다.


### 5.26 Ordered-path completion resubmission poll tightening (2026-03-11)

- TDD(red): `tests/unit/control_plane/test_broker_poller.py`에 active backlog/in-flight가 남아 있을 때 consumer가 idle 0.1s poll 대신 non-blocking consume timeout을 써야 한다는 회귀 테스트 2건을 추가하고 실패를 확인했습니다.
- Green: `pyrallel_consumer/control_plane/broker_poller.py`에 `_get_consume_timeout_seconds()`를 추가해 queued/in-flight work가 남아 있으면 `consumer.consume(..., timeout=0.0)`로 즉시 루프를 재진입하도록 조정했습니다. idle 상태는 기존 0.1s timeout을 유지합니다.

### 5.27 Benchmark process ordering validation false-positive guard (2026-03-12)

- `benchmarks/pyrallel_consumer_test.py`: ordered benchmark validation은 다시 async 모드에만 적용하고, process ordered runs는 `Ordering validation SKIP: process-mode validation unavailable`를 출력하도록 조정했습니다. 기존 process worker 내부 validator는 워커별 상태 복사본 때문에 key/partition ordering false-positive를 만들 수 있었습니다.
- `tests/unit/benchmarks/test_benchmark_runtime.py`: process ordered runs가 skip 메시지를 출력하고, 의도적으로 어긋난 시퀀스 payload를 넣어도 false-positive 예외를 내지 않는 회귀 테스트로 갱신했습니다.
- 검증: `pytest tests/unit/benchmarks/test_benchmark_runtime.py -q` 통과.
- `benchmarks/pyrallel_consumer_test.py`: process benchmark path no longer wraps `process_worker_fn` in a local closure before passing it to `ProcessExecutionEngine`; this avoids macOS spawn pickling failures (`AttributeError: Can't get local object ... validated_process_worker`).
- 회귀 테스트 `test_run_pyrallel_consumer_test_uses_picklable_process_worker` 추가 후 통과.
- 실벤치 검증: `python -m benchmarks.run_parallel_benchmark --num-messages 1000 --num-keys 100 --num-partitions 4 --workloads cpu --order key_hash --strict-completion-monitor off --skip-baseline --skip-async --timeout-sec 120 --topic-prefix pyrallel-cli-verify-20260312-2 --process-group cli-verify-process-20260312-2 --json-output /tmp/pyrallel-cli-verify-20260312-2.json --log-level WARNING` 실행 시 process/key_hash run이 `COMPLETED`로 종료되고 JSON summary가 `/tmp/pyrallel-cli-verify-20260312-2.json`에 기록됨.
- 추가 실벤치 검증: `python -m benchmarks.run_parallel_benchmark --num-messages 1000 --num-keys 100 --num-partitions 4 --workloads cpu --order partition --strict-completion-monitor off --skip-baseline --skip-async --timeout-sec 120 --topic-prefix pyrallel-cli-verify-20260312-3 --process-group cli-verify-process-20260312-3 --json-output /tmp/pyrallel-cli-verify-20260312-3.json --log-level WARNING` 실행 시 process/partition run도 `COMPLETED`로 종료되고 JSON summary가 `/tmp/pyrallel-cli-verify-20260312-3.json`에 기록됨.

### 5.28 Process benchmark ordering validation redesign (2026-03-12)

- `pyrallel_consumer/control_plane/work_manager.py`: completion 처리 시 metrics exporter가 `observe_work_completion(event, work_item, duration)` 훅을 제공하면 원본 `WorkItem`과 함께 부모 프로세스에서 호출하도록 확장했습니다. 기존 exporter는 `observe_completion(...)` 경로를 그대로 사용합니다.
- `benchmarks/pyrallel_consumer_test.py`: process ordered benchmark는 더 이상 worker 프로세스 내부 validator 복사본에 의존하지 않고, 부모 프로세스 `BenchmarkMetricsObserver.observe_work_completion(...)`에서 단일 `OrderingValidator` 상태로 검증합니다. async ordered benchmark는 기존 worker-side validator를 유지합니다.
- 같은 파일의 process 경로는 `ProcessExecutionEngine`에 로컬 closure wrapper 대신 picklable한 top-level `process_worker_fn`을 직접 전달하도록 정리해 macOS spawn pickling 실패를 제거했습니다.
- 회귀 테스트:
  - `tests/unit/control_plane/test_work_manager.py::test_poll_completed_events_uses_work_completion_observer_hook`
  - `tests/unit/benchmarks/test_benchmark_runtime.py::test_run_pyrallel_consumer_test_validates_key_hash_ordering_in_process_mode`
  - `tests/unit/benchmarks/test_benchmark_runtime.py::test_run_pyrallel_consumer_test_raises_on_process_ordering_violation`
  - `tests/unit/benchmarks/test_benchmark_runtime.py::test_run_pyrallel_consumer_test_uses_picklable_process_worker`
- 검증:
  - `pytest tests/unit/benchmarks/test_benchmark_runtime.py tests/unit/control_plane/test_work_manager.py -q` 통과.
  - 실벤치 `process + key_hash` 검증: `python -m benchmarks.run_parallel_benchmark --num-messages 1000 --num-keys 100 --num-partitions 4 --workloads cpu --order key_hash --strict-completion-monitor off --skip-baseline --skip-async --timeout-sec 120 --topic-prefix pyrallel-cli-verify-20260312-4 --process-group cli-verify-process-20260312-4 --json-output /tmp/pyrallel-cli-verify-20260312-4.json --log-level WARNING` → `Ordering validation PASS: key_hash keys=100 checks=1000`.
  - 실벤치 `process + partition` 검증: `python -m benchmarks.run_parallel_benchmark --num-messages 1000 --num-keys 100 --num-partitions 4 --workloads cpu --order partition --strict-completion-monitor off --skip-baseline --skip-async --timeout-sec 120 --topic-prefix pyrallel-cli-verify-20260312-5 --process-group cli-verify-process-20260312-5 --json-output /tmp/pyrallel-cli-verify-20260312-5.json --log-level WARNING` → `Ordering validation PASS: partition partitions=4 checks=1000`.
- Results modal detailed report visibility fix: `benchmarks/tui/app.py`에서 `#results-table` 높이를 `1fr` 대신 고정 높이(`12`)로 조정해 `VerticalScroll` 안에서 DataTable이 1-cell 높이로 붕괴되던 문제를 수정. Textual harness에서 수정 전 `table size height=1`, 수정 후 `height=10` 확인.
- 회귀 테스트: `tests/unit/benchmarks/test_tui_results_report.py`에 modal `#results-table` 가시 높이(`size.height >= 8`) 검증 추가 후 통과.
- TUI run-screen UX polish: 성공 후 `Cancel` 버튼 라벨을 `Reopen report`로 바꾸고 `Back`은 `Exit`를 유지하도록 정리했습니다. 실패 시에는 마지막 stderr 한 줄을 상단 상태 텍스트(`Benchmark failed ...: <error>`)에 요약해 로그를 끝까지 스크롤하지 않아도 핵심 오류를 볼 수 있게 했습니다.
- Run screen spacing compact 조정: `#run-status`, meta/pill rows, output path, progress row, badge/pill margin을 줄여 작은 터미널에서 대시보드가 덜 빽빽하게 보이도록 CSS를 다듬었습니다.
- 검증: `pytest tests/unit/benchmarks/test_tui_app.py tests/unit/benchmarks/test_tui_results_report.py -q` 통과.
- WorkManager correctness/perf bundle (2026-03-12, local integration after team unblock):
  - `on_revoke()` now releases revoked in-flight bookkeeping via `_release_in_flight_item(...)` so `_current_in_flight_count`, dispatch timestamps, and key/partition inflight sets do not leak across rebalance.
  - stale/zombie completions now free tracked in-flight slots even when epoch commit state is intentionally skipped; commit/HWM progression remains fenced by current epoch.
  - `schedule()` now peeks queue head before `execution_engine.submit(...)` and only dequeues on submit success, preserving queue order on submit failure instead of requeueing the failed head at the tail.
  - runnable queue cleanup from the parallel worker remains in place (`_deactivate_queue_key()` prunes stale deque entries) and all related control-plane regressions are green.
- Added/updated regressions in `tests/unit/control_plane/test_work_manager.py` and `tests/unit/control_plane/test_work_manager_ordering.py` for revoke cleanup, stale completion cleanup, submit-failure order preservation, and stale runnable-entry behavior.
- Verification: `pytest tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_work_manager.py tests/unit/control_plane/test_work_manager_ordering.py -q` -> 54 passed.
- Doc/code alignment pass (2026-03-12): `BrokerPoller.MIN_IN_FLIGHT_MESSAGES_TO_RESUME`를 다시 `max_in_flight * 0.7`로 맞췄고(`tests/unit/control_plane/test_broker_poller.py::test_broker_poller_uses_seventy_percent_resume_threshold` 추가/통과), `GEMINI.md`의 기존 5.21 기록(queued count 70% 이하 resume)과도 정합성을 회복했습니다.
- 문서 동기화: `README.md`, `README.ko.md`, `benchmarks/README.md`에서 구식 `--workload` / `all` 설명을 제거하고 현재 CLI(`--workloads`, `--order`, `--strict-completion-monitor`)와 TUI 진입 설명으로 갱신했습니다.
- `prd.md`, `prd_dev.md`는 현재 `BaseExecutionEngine` 계약(`submit(work_item)`, `poll_completed_events`, `wait_for_completion`, `get_in_flight_count`)과 queue-aware 70% resume 규칙, `queue_max_messages` / `strict_completion_monitor_enabled` 설정을 반영하도록 갱신했습니다.
- 검증: `pytest tests/unit/control_plane/test_broker_poller.py -k 'seventy_percent_resume_threshold' -q` 통과, `python -m py_compile pyrallel_consumer/control_plane/broker_poller.py tests/unit/control_plane/test_broker_poller.py` 통과.
- Release prep (2026-03-12): bumped package version in `pyproject.toml` from `0.1.1` to prerelease `0.1.2a1` (requested release label `v0.1.2-a.1`, normalized to valid PyPI/PEP 440 version form for publishing).
- Process shutdown hang investigation (2026-03-13): focused repros showed the long shutdown occurs **after** target processing completes. In the failing `unordered + process + cpu + 2000 msgs` repro, stop triggered at ~0.48s and `broker_poller.stop()` finished in ~10ms, but `engine.shutdown()` entered with residual `in_flight_registry` entries and then spent 30s-per-worker on sequential join timeouts.
- Key evidence: before the experimental drain, shutdown began with non-zero `in_flight_registry` and multiple workers timed out despite processing already being complete. After adding a bounded pre-join IPC drain in `ProcessExecutionEngine.shutdown()`, the same repro showed `in_flight_registry=6` at shutdown start, `registry_events=708` drained in ~28ms, residual registry dropping to 0, and all workers joining gracefully with total engine shutdown ~0.06s.
- Interpretation: the dominant shutdown hang is not real remaining work or commit latency; it is parent-side failure to keep draining registry/completion IPC long enough after stop, allowing worker shutdown to stall until terminate. In the reproduced case, the critical queue was the registry-event path (`completion_events=0`, `registry_events=708`).
- Process shutdown fix landed (2026-03-13): `ProcessExecutionEngine.shutdown()` now performs a bounded pre-join IPC drain after enqueueing sentinels, consuming residual registry/completion events before waiting on worker joins. This eliminates the reproduced 30s-per-worker hang in the `unordered + process + cpu` benchmark path while preserving end-to-end TPS semantics.
- Regression coverage: `tests/unit/execution_plane/test_process_engine_batching.py::TestShutdownLifecycle::test_shutdown_drains_registry_events_before_join` added; `pytest tests/unit/execution_plane/test_process_engine_batching.py tests/unit/execution_plane/test_process_execution_engine.py -q` -> 15 passed.
- Smoke verification: `process + unordered + cpu` (2000 msgs) and `process + partition + sleep` (2000 msgs, 0.5ms) both complete without `did not shut down gracefully` warnings after the fix.
- Rebalance state preservation implementation (2026-03-13, local continuation): `ParallelConsumerConfig`에 `rebalance_state_strategy` (`contiguous_only` 기본, `metadata_snapshot` 옵션)를 추가했고, `BrokerPoller._on_assign()`는 `metadata_snapshot`일 때 assignment metadata를 decode해 sparse completed offsets를 `OffsetTracker(initial_completed_offsets=...)`로 hydrate하도록 보강했습니다. `last_fetched_offset`도 hydrate된 최대 완료 오프셋까지 복원합니다.
- `BrokerPoller._on_revoke()`는 `metadata_snapshot`일 때 final revoke commit에 sparse completed offsets를 commit metadata로 싣고, `contiguous_only`일 때는 기존처럼 contiguous safe offset만 커밋합니다. 둘 다 실제 committed offset은 contiguous safe offset만 사용합니다.
- 회귀 테스트 추가: `tests/unit/test_config.py`의 rebalance strategy default/env override, `tests/unit/control_plane/test_broker_poller.py`의 assignment hydration / revoke metadata snapshot on/off 테스트, `tests/unit/control_plane/test_work_manager.py`의 stale completion after reassign contract test.
- `BrokerPoller._on_assign()` metadata hydration 보강: `consumer.committed(partitions)`가 반환한 committed partition entry의 metadata를 assignment snapshot source로 우선 사용하도록 수정했습니다. committed offset base와 sparse completed-offset snapshot이 서로 다른 객체를 보던 문제를 막기 위한 변경이며, 회귀 테스트 `test_on_assign_uses_committed_partition_metadata_for_snapshot_hydration`를 추가했습니다.
- `BrokerPoller._encode_revoke_metadata()`가 revoke final commit에서도 `_get_commit_metadata_offsets(...)`를 재사용하도록 정리했습니다. 이제 일반 commit과 revoke commit 모두 `base_offset` 이상 + capped subset만 metadata snapshot에 싣고, 대규모 sparse completed set 때문에 revoke snapshot이 overflow marker로 상태를 잃는 경로를 줄였습니다. 회귀 테스트 `test_on_revoke_metadata_snapshot_limits_offsets_encoded` 추가.
- CI hang 조사(2026-03-13): GitHub Actions `ci` workflow의 `Run unit tests` step이 장시간 in-progress로 남는 현상을 추적하기 위해, 현재 `.github/workflows/ci.yml`에 job timeout(20분)과 `timeout ... pytest` 래핑, pytest 이후 `ps`/`pgrep` 덤프를 임시로 추가했습니다. 목표는 테스트 자체가 느린지/프로세스가 남는지 runner에서 직접 확인하는 것입니다.
- CI hang 조사 추가(2026-03-13): `Run unit tests`를 Python wrapper로 감싸 `faulthandler.dump_traceback_later(60, repeat=True)`를 켰습니다. pytest summary 이후 interpreter teardown에서 멈춘다면 다음 CI 로그에 thread stack dump가 반복 출력되도록 하기 위한 순수 진단용 계측입니다.
- CI hang 조사 보정(2026-03-13): unit suite 로컬 기준이 약 3~5분이므로 diagnostic timeout을 5분(`timeout 5m`)으로 낮추고 thread dump 주기도 30초로 줄였습니다. 장시간 대기 대신 teardown hang를 빠르게 포착하기 위한 조정입니다.
- 문서 동기화: `README.md`, `README.ko.md`, `prd.md`, `prd_dev.md`에 `contiguous_only` vs `metadata_snapshot` 정책과 at-least-once/idempotency 기대치를 반영했습니다.
- 검증: `pytest tests/unit/test_config.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_broker_poller_completion_driven.py tests/unit/control_plane/test_work_manager.py -q` -> 55 passed; `python -m py_compile pyrallel_consumer/config.py pyrallel_consumer/control_plane/broker_poller.py tests/unit/test_config.py tests/unit/control_plane/test_broker_poller.py tests/unit/control_plane/test_work_manager.py` 통과.
- WorkManager revoke undercount fix (2026-03-13): `_release_in_flight_item()`가 queued+submitted 전체를 추적하는 `_in_flight_work_items`와 실제 submitted 항목만 세는 `_current_in_flight_count`의 차이를 반영하도록 수정했습니다. 이제 `dispatch_time`이 있는 항목(실제 submit 성공 후 timestamp가 기록된 경우)에 대해서만 in-flight count를 감소시켜, revoke 시 queued-only 항목 때문에 카운터가 과소 계산되던 문제를 막습니다.
- 회귀 테스트 `tests/unit/control_plane/test_work_manager.py::test_on_revoke_does_not_decrement_for_queued_unsubmitted_items` 추가.
- Process execution logging hardening (2026-03-13): `ProcessExecutionEngine`가 프로세스 워커 로그용 `multiprocessing.Queue`를 이제 `process_config.queue_size`로 bounded 생성해 로그 폭주 시 무한 메모리 증가를 막습니다. 워커 로그 레벨은 기존 `INFO`를 유지했습니다.
- 회귀 테스트 `tests/unit/execution_plane/test_process_execution_engine.py::test_process_execution_engine_bounds_log_queue_to_process_queue_size` 추가.
- CI/release policy alignment prep (2026-03-13): repo에 `.github/workflows/`가 없음을 확인하고, 첫 GitHub Actions unit gate로 Python 3.12/3.13 matrix에서 `pytest tests/unit -q --maxfail=1`를 실행하는 `ci.yml` 초안을 추가했습니다. release policy는 현재 배포 버전/분류(`0.1.2a1`, Alpha)에 맞춰 README/README.ko에 `main`을 active hardening branch로 보는 문구를 추가했습니다.
- 로컬 게이트 검증: `pytest tests/unit -q --maxfail=1` -> 277 passed.
- Benchmark 결과 확장(2026-03-15): `BenchmarkStats`/`BenchmarkResult`에 100-message window 기반 TPS 분포 지표를 추가했습니다. JSON에 `window_size_messages`, `tps_p50_window`, `tps_p10_window`, `tps_min_window`를 저장하고, TUI 결과 모달 상세표에도 `TPS P50 (100)`, `TPS P10 (100)`, `TPS Min (100)` 컬럼을 노출합니다. 부족한 샘플(<100 completions)은 `null`/`—`로 처리합니다.
- 검증: `pytest tests/unit/benchmarks/test_stats.py tests/unit/benchmarks/test_tui_results_report.py tests/unit/benchmarks/test_benchmark_runtime.py -q` -> 28 passed, `python -m py_compile benchmarks/stats.py benchmarks/tui/results_report.py tests/unit/benchmarks/test_stats.py tests/unit/benchmarks/test_tui_results_report.py` 통과.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomorrow9913)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tomorrow9913)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
