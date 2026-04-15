## traffic-algo

> 4지 교차로에서 신호 대기 시간(`duration`) 변화에 따른 효율성을 객체 기반 시뮬레이션으로 측정합니다. `terminal_runner`를 통해 현실적인 교통 부하 상황에서의 통계적 유의성을 확보한 뒤, GUI를 통해 시각적으로 검증하는 워크플로우를 가집니다.

# 📝 [Smart Spec] Traffic Efficiency Simulator: Phase-by-Phase Development

## 🎯 Goal
4지 교차로에서 신호 대기 시간(`duration`) 변화에 따른 효율성을 객체 기반 시뮬레이션으로 측정합니다. `terminal_runner`를 통해 현실적인 교통 부하 상황에서의 통계적 유의성을 확보한 뒤, GUI를 통해 시각적으로 검증하는 워크플로우를 가집니다.

## 📊 Realistic Scenario Standards (현실적 변수 범위)
모든 Phase의 검증은 다음의 현실 기반 범위를 포함해야 합니다.

| 수준 | Spawn Rate (차량 생성 밀도) | 비고 (현실 환산 시) |
| :--- | :--- | :--- |
| **Low** | `0.02` | 한산한 도로 (약 360 VPH) |
| **Medium** | `0.04` | 일반 시내 도로 (약 720 VPH) |
| **High** | `0.06` | 혼잡 구간 (약 1,080 VPH) |
| **Saturated** | `0.08` | 포화 상태 (약 1,440 VPH) |

*   **Duration Range:** `20s ~ 100s` (국내 신호 설계 지침 기준)
*   **Target Count:** 최소 `500 ~ 1,000`대 (통계적 안정성 확보)

## 🔄 Phase Roadmap (Iterative Development)

* **✅ Phase A: Straight Only (Completed)**
    * 북-남 / 동-서 직진 신호 체계 구축.
    * `logic.py` 핵심 엔진 구현 및 대기 시간 산출 버그 수정.
* **✅ Phase B: Left Turn & RHT (Completed)**
    * **Right-Hand Traffic (RHT):** 현실적인 우측통행 좌표계 및 차선(안쪽/바깥쪽) 시스템 도입.
    * **4-Unit Signal:** [적-황-좌-녹] 4구 신호등 체계 및 건너편 배치 시각화.
    * **Split Phase:** 충돌 방지를 위한 진입부별 개별 통행(Protected Approach) 신호 모드 추가.
* **🏃 Phase C-1: Right Turn & Yielding (In Progress)**
    * `Vehicle` 클래스: 우회전(`R`) 타입 및 궤적 로직 추가.
    * **Red-Light Right Turn:** 빨간불에서도 일시 정지 후 우회전 허용(Stop-and-Go).
    * **Collision Avoidance:** 우회전 차량 진입 시 타 방면 직진 차량과의 충돌 방지 로직 기초 설계.
* **🏃 Phase C-2: Pedestrians & Crosswalks (Pending)**
    * 보행자 객체 및 횡단보도 추가. 
    * 차량 우회전 시 보행 신호에 따른 일시정지 및 보행자 보호 로직 반영.
* **Phase D: Scramble (Diagonal)**
    * 모든 차량 정지 후 보행자만 전 방향 이동하는 페이즈 추가.

## 🏃 Current Implementation: Phase B Results
Phase B 분석 결과, 2차선 교차로에서 **'공용 차로 + 진입부별 직좌 동시 신호'** 또는 **'전용 차로 + 직진 후 좌회전 신호'**가 가장 효율적임이 데이터로 입증되었습니다.

1.  **Engine Logic (`logic.py`):**
    * `Vehicle` 클래스: 좌회전 목적지(`turn_type`) 및 차선(`lane`) 기반 주행 로직.
    * **Trajectory:** 좌회전 후 목적지 도로의 우측 차선으로 안착하는 궤적 보정 로직 적용.
2.  **Visualization (`app.py`):**
    * **Intuitive UI:** 신호등 박스 내 화살표(←) 점등 및 차량 몸체 방향 표시.
    * **Real-time Config:** 실행 중 차선 구성 및 신호 모드 스위칭 지원.

## 🎨 UI/UX Principles (app.py)
시뮬레이션의 시각적 일관성과 직관성을 위해 다음 원칙을 준수합니다.

*   **Color System:**
    *   **Roads:** `#34495e` (Dark Gray), **Background:** `#2c3e50`.
    *   **Lane Dividers:** 중앙선 `#f1c40f` (Yellow), 차선 구분선 `white` (Dashed).
    *   **Vehicles:** 직진(S) `#3498db` (Blue), 좌회전(L) `#9b59b6` (Purple), 정지 상태 `#e74c3c` (Red).
*   **Signal Visualization (4-Unit):**
    *   구성: `[Red | Yellow | Left(←) | Green]` 순서의 가로형 박스.
    *   배치: 운전자 시야를 고려하여 교차로 건너편(Far-side)에 배치.
    *   상태: 활성 상태는 밝은 색, 비활성 상태는 어두운 색(Dimmed)으로 표현.
*   **Information Overlay:**
    *   차량 객체 내부에 목적지 화살표(←)를 표시하여 의도 시각화.
    *   우측 컨트롤 패널을 통해 `Avg Wait`, `Passed Count`, `Current Phase` 실시간 모니터링.

## ⚠️ 3-Level Boundaries (경계 시스템)

✅ **Always do:**
- 모든 파일 최상단에 `# filename.py` 주석 작성.
- `logic.py`는 UI 라이브러리에 의존하지 않는 순순 연산 모듈로 유지.
- 새로운 페이즈 구현 후 반드시 **현실적 변수 범위**에서 `terminal_runner`를 실행하여 효율 변화를 리포트.
- 차량의 '평균 대기 시간' 산출 시, 교차로 진입 대기부터 통과 완료 시점까지의 시간을 정확히 계산.

⚠️ **Ask first:**
- 특정 페이즈에서 대기 시간이 급격히 증가하여 시뮬레이션이 종료되지 않을 경우.
- 새로운 외부 라이브러리 설치가 필요할 경우.

🚫 **Never do:**
- 사용자의 지시 없이 다음 페이즈 기능을 미리 구현하지 말 것.
- `time_scale` 조절 시 물리 연산(충돌/정지선 체크)을 건너뛰지 말 것.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anoBand)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/anoBand)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
