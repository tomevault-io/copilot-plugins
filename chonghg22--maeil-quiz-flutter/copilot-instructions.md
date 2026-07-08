## maeil-quiz-flutter

> - **앱 이름**: 매일퀴즈 (MaeilQuiz)

# 매일퀴즈 (MaeilQuiz) Flutter 프로젝트

## 프로젝트 개요
- **앱 이름**: 매일퀴즈 (MaeilQuiz)
- **설명**: 매일 새로운 퀴즈로 지식을 키우세요
- **패키지명**: com.maeilquiz.app
- **플랫폼**: Android
- **Flutter SDK**: >=3.3.0

## 기술 스택
- **프레임워크**: Flutter
- **상태관리**: Riverpod (flutter_riverpod)
- **라우팅**: GoRouter (go_router)
- **백엔드**: Supabase (supabase_flutter) - RPC 함수 및 DB 직접 호출
- **광고**: Google Mobile Ads (google_mobile_ads)
- **로컬저장소**: shared_preferences, flutter_secure_storage

## Supabase 설정
- **URL**: REDACTED_SUPABASE_URL
- **Anon Key**: 
- **스키마**: maeil_quiz
- **연결방식**: Supabase Flutter SDK 직접 호출 (Spring Boot 백엔드 없음)

### RPC 함수
| 함수명 | 파라미터 | 설명 |
|--------|----------|------|
| get_quiz_feed | p_android_id, p_category, p_cursor, p_size | 퀴즈 피드 조회 (daily_count < 20, 푼 문제 제외) |
| upsert_user | p_android_id | 사용자 등록/조회 |
| increment_daily_count | p_android_id | 답변 제출 시 일일 카운트 +1 (날짜 바뀌면 자동 리셋) |
| update_categories | p_android_id, p_categories | 카테고리 업데이트 |

### DB 테이블 (maeil_quiz 스키마)
- **users**: id, android_id(UNIQUE), categories(jsonb), is_premium, daily_count, daily_reset_at
- **questions**: id, content, option_1~4, answer(1~4 int), explanation, category, is_active
- **user_history**: id, user_id, question_id, is_correct, answered_at (30일 보관)
- **payments**: id, user_id, payment_key, order_id, amount, status

## 프로젝트 구조
lib/
core/
constants/    # app_constants.dart (supabaseUrl, supabaseAnonKey, categories)
router/       # app_router.dart (GoRouter 라우트 정의)
features/
auth/         # splash_screen.dart, user_repository.dart
quiz/         # quiz_feed_screen.dart, quiz_provider.dart, quiz_repository.dart, models/question.dart
settings/     # settings_screen.dart, category_screen.dart
payment/      # payment_screen.dart (미구현)
shared/
widgets/      # ad_banner_widget.dart, ad_interstitial_helper.dart
theme/        # app_theme.dart
## 라우트 구조
| 경로 | 화면 | 설명 |
|------|------|------|
| /splash | SplashScreen | androidId 생성/등록 후 /quiz로 이동 |
| /quiz | QuizFeedScreen | 메인 퀴즈 피드 (세로 PageView) |
| /settings | SettingsScreen | 구독 상태, 카테고리 변경 |
| /categories | CategoryScreen | 단일 카테고리 선택 |
| /payment | PaymentScreen | 프리미엄 결제 (미구현) |

## 핵심 로직
- **사용자 식별**: UUID v4 생성 후 SharedPreferences에 android_id로 저장 (익명)
- **카테고리**: 단일 선택, users.categories(jsonb)에 저장 ex) ["IT"]
- **일일 한도**: 무료 20문제, daily_count >= 20이면 피드 빈 리스트 반환
- **광고**: 하단 배너 고정 + 15문제마다 전면 광고
- **정답 처리**: answer 컬럼은 1~4 (1-based), Flutter 선택값도 1~4로 통일
- **피드 초기화**: 카테고리 변경 시에만 ref.invalidate(quizFeedProvider)

## AdMob 설정
| 항목 | ID |
|------|-----|
| 앱 ID (AndroidManifest) | REDACTED_ADMOB_APP_ID |
| 배너 광고 단위 ID | REDACTED_ADMOB_BANNER_ID |
| 전면 광고 단위 ID | REDACTED_ADMOB_INTERSTITIAL_ID |

> ⚠️ 현재 AdMob 앱 스토어 미연결 상태 (403 에러). 구글 플레이 정식 출시 후 연결 필요.

## 빌드 및 실행
```bash
# 의존성 설치
flutter pub get

# Chrome 실행 (개발용, AdMob 미동작)
flutter run -d chrome

# 폰 실행 (USB 연결 후)
flutter run

# AAB 빌드 (구글 플레이 업로드용)
flutter build appbundle

# 런처 아이콘 생성
dart run flutter_launcher_icons
```

## 버전 관리
- **versionCode**: 매 배포마다 +1 (정수, 구글 플레이 내부용)
- **versionName**: 사용자 표시용 (예: 1.0.1)
- **위치**: android/app/build.gradle

## 개발 컨벤션
- **상태관리**: 비즈니스 로직은 반드시 Riverpod Provider로 관리
- **라우팅**: 모든 네비게이션은 GoRouter 사용, 라우트 정의는 app_router.dart에서
- **Supabase**: 복잡한 쿼리는 RPC 함수로, 단순 조회는 .from().select() 직접 호출
- **테마**: 전역 테마는 app_theme.dart에서 관리, primaryColor: #6B21A8 (다크 퍼플)
- **커밋 메시지**: 한국어로 작성

## 구글 플레이 현황
- **개발자 계정**: 승인 완료
- **비공개 테스트**: 진행 중 (12명 + 14일 조건 충족 필요)
- **정식 출시**: 미완료
- **개인정보처리방침**: https://chonghg22.github.io/maeil-quiz-flutter/docs/privacy-policy.html

## 주의사항
- AdMob은 Chrome에서 동작 안 함 → 반드시 실제 Android 기기에서 테스트
- flutter run 시 폰이 연결 안 되면 adb kill-server && adb start-server 실행
- 구글 플레이 AAB 업로드 시 versionCode 반드시 이전보다 높아야 함
- Supabase answer 컬럼은 1-based (1~4), Flutter에서도 동일하게 처리

---
> Source: [chonghg22/maeil_quiz_flutter](https://github.com/chonghg22/maeil_quiz_flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
