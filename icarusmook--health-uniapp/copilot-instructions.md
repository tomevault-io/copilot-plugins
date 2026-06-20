## health-uniapp

> Galaxy Watch 8 Classic(삼성헬스)과 COROS PACE 3의 건강/운동 데이터를

# Health Unify — 프로젝트 컨텍스트

## 프로젝트 개요
Galaxy Watch 8 Classic(삼성헬스)과 COROS PACE 3의 건강/운동 데이터를
**Android Health Connect**를 통해 한 곳에서 통합해서 보여주는 Flutter 앱.

- 대상 기기: Samsung Galaxy S23 (Android)
- 데이터 허브: Android Health Connect (구글 공식 건강 데이터 API)
- 언어/프레임워크: Flutter (Dart)
- 패키지: `health` (pub.dev — Health Connect 지원)

---

## 기기 사용 패턴
| 시간대 | 착용 기기 | 주요 데이터 |
|--------|-----------|------------|
| 00:00 ~ 14:00 | Galaxy Watch 8 Classic | 수면, 심박, 걸음수, SpO2 |
| 14:00 ~ 22:00 | COROS PACE 3 | 운동 기록, 페이스, 거리, VO2Max |
| 22:00 ~ 24:00 | Galaxy Watch 8 Classic | 심박, 취침 전 활동 |

---

## 핵심 정책 결정사항

### 1. 동시 활성 처리 (중요)
두 기기가 같은 시간대에 데이터를 모두 기록한 경우:
- **도넛 차트(24시간 활동 비율)**: Galaxy Watch 우선 표시
- **운동 기록 화면**: 두 기기 데이터 모두 표시 (출처 명시)

```dart
String getActiveDevice(bool gwActive, bool corosActive) {
  if (gwActive && corosActive) return 'galaxy'; // Galaxy 우선
  if (gwActive) return 'galaxy';
  if (corosActive) return 'coros';
  return 'none';
}
```

### 2. 운동 이름 자동 생성 규칙
Health Connect는 운동 이름을 저장하지 않으므로 앱이 직접 생성.

```dart
// 시간대 매핑
String getTimePeriod(int hour) {
  if (hour >= 5 && hour < 7)  return '새벽';
  if (hour >= 7 && hour < 10) return '아침';
  if (hour >= 10 && hour < 11) return '오전';
  if (hour >= 11 && hour < 13) return '점심';
  if (hour >= 13 && hour < 17) return '오후';
  if (hour >= 17 && hour < 20) return '저녁';
  if (hour >= 20 && hour < 24) return '밤';
  return '심야';
}

// 운동 종목 한글 매핑
String getExerciseName(int exerciseType) {
  // Health Connect ExerciseSessionRecord.EXERCISE_TYPE_* 상수 사용
  const map = {
    56: '러닝', 79: '산책', 8: '자전거',
    74: '수영', 80: '근력 운동', 37: '하이킹',
  };
  return map[exerciseType] ?? '운동';
}

// 최종 이름 = 시간대 + 종목 (예: "저녁 러닝", "점심 산책")
String buildExerciseName(DateTime start, int type) =>
    '${getTimePeriod(start.hour)} ${getExerciseName(type)}';
```

---

## 3개 화면 구성

### 화면 1 — 홈 (오늘 요약)
- 상단: 날짜, 알림 아이콘
- 출처 범례: 파란 점(Galaxy Watch) / 초록 점(COROS)
- 총 칼로리 링 차트 (Galaxy + COROS 합산, 도넛형)
- 4개 핵심 지표 그리드: 걸음수 / 운동거리 / 평균심박 / 페이스
- 수면 카드 (Galaxy Watch 전용): 총 수면, 깊은잠, REM
- 건강 지표 그리드: SpO2(Galaxy) / VO2Max(COROS)
- 하단 네비게이션: 홈 / 운동 / 통계 / 설정

### 화면 2 — 운동 기록
- 상단 탭 필터: 전체 / Galaxy / COROS
- 이번 주 운동 시간 막대 그래프 (기기별 색 구분)
- 오늘 운동 목록 카드 (종목 아이콘, 출처 기기, 시간, 거리/페이스)
- 운동 상세: 거리, 시간, 평균페이스, 평균심박, 칼로리, 고도상승

### 화면 3 — 하루 타임라인
- 두 기기 데이터를 시간순 통합 리스트
- 워치 교체 이벤트도 타임라인에 표시
- 하단: 24시간 도넛 차트
  - 24개 호(각 1시간 = 15°), 자정(00:00)이 12시 방향
  - 파란색: Galaxy Watch 활성, 초록색: COROS 활성
  - 세그먼트 탭 시 해당 시간대 상세 표시
  - 하단 요약: Galaxy Xh (X%) / COROS Xh (X%)

---

## 색상 & 디자인 규칙
| 항목 | 값 |
|------|----|
| Galaxy Watch 색 | `#378ADD` (파란색) |
| COROS 색 | `#1D9E75` (초록색) |
| 아이콘 라이브러리 | Material Icons (Flutter 기본) |
| 테마 | Light/Dark 모두 지원 |
| 출처 표시 | 모든 데이터 카드에 기기 출처 점(dot) 표시 |

---

## Health Connect 읽기 대상 데이터 타입
```dart
// health 패키지의 HealthDataType 열거형 사용
final types = [
  HealthDataType.STEPS,              // 걸음수 (Galaxy 주)
  HealthDataType.HEART_RATE,         // 심박수 (양쪽)
  HealthDataType.ACTIVE_ENERGY_BURNED, // 활동 칼로리 (양쪽)
  HealthDataType.TOTAL_CALORIES_BURNED, // 총 칼로리
  HealthDataType.SLEEP_ASLEEP,       // 수면 (Galaxy 주)
  HealthDataType.SLEEP_DEEP,         // 깊은 수면
  HealthDataType.SLEEP_REM,          // REM 수면
  HealthDataType.BLOOD_OXYGEN,       // SpO2 혈중산소 (Galaxy 주)
  HealthDataType.WORKOUT,            // 운동 세션 (양쪽)
  HealthDataType.WEIGHT,             // 체중
  HealthDataType.VO2MAX,             // VO2Max (COROS 주)
];
```

---

## 개발 환경
- IDE: VSCode
- Flutter SDK 설치 필요
- Android SDK 필요 (Android Studio 통해 설치)
- 테스트 기기: Samsung Galaxy S23 (USB 디버깅)
- pubspec.yaml 핵심 패키지:
  - `health: ^10.x.x` (Health Connect 연동)
  - `fl_chart: ^0.x.x` (차트/그래프)
  - `intl: ^0.x.x` (날짜 한국어 포맷)

---

## 개발 단계 로드맵
1. **Phase 1** (2주): Flutter 환경 설정, 기기 연결, Hello World
2. **Phase 2** (3주): Health Connect 권한 요청 + 데이터 읽기
3. **Phase 3** (4주): 3개 화면 UI 구현
4. **Phase 4** (선택): 홈 위젯, 알림, Claude API 운동 분석

---

## Claude Code 작업 시 유의사항
- 코드 각 블록마다 **한국어 주석** 필수
- 복잡한 로직은 함수로 분리하고 이유 설명
- 에러 발생 시 원인과 해결책 함께 제시
- 단계별로 진행하고 각 단계 완료 확인 후 다음 진행

---
> Source: [icarusmook/health_uniapp](https://github.com/icarusmook/health_uniapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
