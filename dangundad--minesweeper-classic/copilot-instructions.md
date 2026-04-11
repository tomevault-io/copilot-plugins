## minesweeper-classic

> > 최종 업데이트: 2026-03-06

# Minesweeper Classic -- 프로젝트 가이드

> 최종 업데이트: 2026-03-06

## 프로젝트 개요

클래식 지뢰찾기 게임 앱. Beginner(9x9), Intermediate(16x16), Expert(16x30) 난이도와 커스텀 모드를 지원합니다. 첫 클릭 안전, 깃발 표시, 타이머, 최고 기록, 보상형 광고 기반 추가 목숨 기능을 제공합니다.

- **패키지명**: com.dangundad.minesweeper
- **버전**: 1.0.0+1
- **디자인 기준**: 375x812 (ScreenUtil)
- **테마**: FlexScheme.indigo, 시스템 모드 (`ThemeMode.system`)

## 기술 스택

| 분류 | 패키지 | 버전 |
|------|--------|------|
| 상태 관리 | get | ^4.7.3 |
| 로컬 저장 | hive_ce + hive_ce_flutter | ^2.19.3 / ^2.3.4 |
| UI 반응형 | flutter_screenutil | ^5.9.3 |
| 테마 | flex_color_scheme (indigo) | ^8.4.0 |
| 폰트 | google_fonts | ^6.3.2 |
| 아이콘 | lucide_icons_flutter | ^3.1.10 |
| 광고 | google_mobile_ads | ^6.0.0 |
| 광고 미디에이션 | gma_mediation_applovin, pangle, unity | ^2.5.1 / ^3.5.0 / ^1.6.2 |
| 인앱 구매 | in_app_purchase | ^3.2.3 |
| 애니메이션 | flutter_animate | ^4.5.2 |
| Firebase | firebase_core, analytics, crashlytics | ^4.4.0 / ^12.1.2 / ^5.0.7 |
| 유틸 | intl, share_plus, vibration, uuid | - |
| 앱 평가 | in_app_review, rate_my_app | ^2.0.11 / ^2.3.2 |

## 개발 명령어

```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs
flutter analyze
flutter run
```

## 아키텍처 (프로젝트 구조)

```
lib/
├── main.dart                        # AdMob -> Hive -> 광고매니저 -> PurchaseService
├── hive_registrar.g.dart
└── app/
    ├── admob/
    │   ├── ads_helper.dart          # 광고 ID 관리
    │   ├── ads_banner.dart          # 배너 광고 위젯
    │   ├── ads_interstitial.dart    # 전면 광고 매니저
    │   └── ads_rewarded.dart        # 보상형 광고 매니저 (추가 목숨)
    ├── bindings/
    │   └── app_binding.dart         # GetX 앱 바인딩
    ├── controllers/
    │   ├── game_controller.dart     # 핵심 게임 로직 (Cell 모델, 보드 생성, 탐색)
    │   ├── home_controller.dart     # 홈 화면
    │   ├── history_controller.dart  # 게임 히스토리
    │   ├── premium_controller.dart  # 프리미엄 관리
    │   ├── setting_controller.dart  # 설정
    │   └── stats_controller.dart    # 통계 화면
    ├── data/
    │   ├── enums/
    │   │   ├── difficulty.dart      # beginner/intermediate/expert/custom
    │   │   └── game_status.dart     # initial/playing/won/lost
    │   └── models/                  # (Cell은 game_controller 내부 정의)
    ├── pages/
    │   ├── home/                    # 난이도 선택, 게임 시작
    │   ├── game/                    # 게임 보드 화면
    │   │   └── widgets/
    │   │       ├── cell_widget.dart      # 셀 위젯 (지뢰/숫자/깃발)
    │   │       └── result_dialog.dart    # 결과 다이얼로그
    │   ├── history/                 # 게임 히스토리
    │   ├── stats/                   # 통계
    │   ├── settings/                # 설정
    │   └── premium/                 # 프리미엄 구매
    ├── routes/
    │   ├── app_routes.dart          # HOME/GAME/SETTINGS/HISTORY/STATS/PREMIUM
    │   └── app_pages.dart
    ├── services/
    │   ├── hive_service.dart        # settings/app_data 박스
    │   ├── purchase_service.dart    # 인앱 구매
    │   ├── app_rating_service.dart  # 앱 평가
    │   └── activity_log_service.dart
    ├── theme/
    │   └── app_flex_theme.dart      # FlexScheme.indigo
    ├── translate/
    │   └── translate.dart           # GetX 다국어 (ko 기본)
    ├── utils/
    │   └── app_constants.dart       # HiveKeys, AppUrls, PurchaseConstants
    └── widgets/
        └── confetti_overlay.dart    # 승리 컨페티
```

## 데이터 모델

| 모델 | 위치 | 설명 |
|------|------|------|
| Cell | game_controller.dart 내부 | 셀 (지뢰 여부, 공개/깃발/폭발 상태, 인접 지뢰 수) |
| Difficulty | difficulty.dart | 난이도별 rows/cols/mines 설정 |

## 개발 가이드라인

- `Cell` 클래스: `game_controller.dart` 내부에 정의됨 (별도 Hive 모델 없음)
- 첫 클릭 안전: `_firstClick` 플래그로 첫 탭 시 지뢰 재배치
- `Difficulty` enum: rows/cols/mines 속성 + hiveKey/labelKey 편의 getter
- 타이머: `Timer.periodic` 사용, `onClose()`에서 반드시 cancel
- 추가 목숨: 보상형 광고 시청 시 `hasExtraLife` 활성화
- 커스텀 모드: `customRows`, `customCols`, `customMines` RxInt로 관리
- 광고 ID: `ads_helper.dart`에서 테스트 ID를 프로덕션 ID로 교체 필요

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dangundad)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dangundad)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
