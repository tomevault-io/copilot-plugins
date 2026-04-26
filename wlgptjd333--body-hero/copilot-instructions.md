## body-hero

> Body Hero의 Python ML/UDP 도구와 Godot 사이 연동에 대한 규칙. Godot 쪽에서는 UDP 액션/좌표 인터페이스를 유지하고, ML 로직은 tools/*.py에서만 변경하도록 한다.


# ML & Python Tools - 연동 규칙

## 역할 분리

- Python/ML:
  - `collect_pose_data.py`, `train_pose_classifier.py`, `pose_server.py`, `udp_send_webcam_ml.py` 등에서
  - 포즈 데이터 수집, 학습, 추론, 액션 문자열 생성까지 처리한다.
- Godot/GDScript:
  - UDP로 전달된 액션 문자열/좌표를 해석하고,
  - `play_action` 호출, 글러브 위치 보정 등 게임 연출에만 집중한다.

## 인터페이스 유지

- Godot 쪽은 다음 인터페이스를 **변경하지 않는 것**을 기본으로 한다:
  - 좌표: `"left_x,left_y,right_x,right_y"`
  - 액션: `"jab_l"`, `"jab_r"`, `"upper_l"`, `"upper_r"`, `"hook_l"`, `"hook_r"`, `"guard"`, `"guard_end"`
- 새 모델/옵션/threshold 추가 제안은 Python/ML 쪽 코드와 `tools/README_ML.md`에 한정해 설명한다.

## Python/환경

- ML 관련 커맨드는 `tools/README_ML.md`의 워크플로를 우선 따른다.
- TensorFlow 등 의존성 때문에 **Python 3.9~3.12 가상환경** 사용을 권장하며, Python 3.14는 피하도록 안내한다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wlgptjd333) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
