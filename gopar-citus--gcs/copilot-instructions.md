## gcs

> **유형:** 안드로이드 애플리케이션 (Kotlin)

# 프로젝트 컨텍스트 (Project Context)

## 개요 (Overview)
**프로젝트명:** gcs
**유형:** 안드로이드 애플리케이션 (Kotlin)
**위치:** `C:\Dev\AndroidStudioProjects\gcs`

이 프로젝트는 Kotlin과 주요 Jetpack 라이브러리를 사용하는 기본적인 안드로이드 프로젝트 구조입니다.
메인 페이지에 여러 데모를 연결합니다.
첫번째로 오프라인 퍼스트(Offline-first) 아키텍처를 이용한 데모를 만듭니다.


## 아키텍처 (Architecture)
- **언어:** Kotlin
- **빌드 시스템:** Gradle (Kotlin DSL)
- **최소 SDK (Min SDK):** 24 (Android 7.0)
- **타겟 SDK (Target SDK):** 34 (Android 14)
- **UI 툴킷:** XML 레이아웃 (View 시스템) 및 Material Design 3
- **MVVM 패턴**
- **Jetpack 라이브러리:**
- **Local Storage/Database: Room**
- **오프라인 퍼스트(Offline-first) 아키텍처**

## 주요 파일 (Key Files)
- `settings.gradle.kts`: 프로젝트 레벨의 저장소 및 모듈 설정.
- `build.gradle.kts`: 최상위 빌드 설정.
- `app/build.gradle.kts`: 앱 모듈 설정 (의존성, SDK 버전 등).
- `app/src/main/AndroidManifest.xml`: 앱 매니페스트 (권한, 액티비티 설정).
- `app/src/main/java/com/example/gcs/MainActivity.kt`: 메인 진입점 액티비티.
- `app/src/main/res/layout/activity_main.xml`: 메인 화면 레이아웃.

### 오프라인 우선 데모 관련 파일 (Offline-First Demo Related Files)
**수정된 파일:**
- `build.gradle.kts`: KSP 플러그인 추가
- `app/build.gradle.kts`: 종속성, ViewBinding, JVM 대상 추가/수정
- `app/src/main/AndroidManifest.xml`: 인터넷 권한, `DemoActivity` 등록 추가
- `app/src/main/java/com/example/gcs/MainActivity.kt`: `DemoActivity` 실행 로직 추가, ViewBinding 활성화
- `app/src/main/res/layout/activity_main.xml`: 데모 화면으로 이동하는 버튼 추가

**새로 추가된 파일 (데이터 계층):**
- `app/src/main/java/com/example/gcs/data/model/DemoItem.kt`: 데이터 모델 정의
- `app/src/main/java/com/example/gcs/data/local/DemoItemDao.kt`: Room DAO 인터페이스
- `app/src/main/java/com/example/gcs/data/local/AppDatabase.kt`: Room 데이터베이스 클래스
- `app/src/main/java/com/example/gcs/data/remote/ApiService.kt`: Retrofit API 서비스 인터페이스
- `app/src/main/java/com/example/gcs/data/repository/DemoRepository.kt`: 데이터 저장소 (로컬/원격)

**새로 추가된 파일 (UI 계층):**
- `app/src/main/java/com/example/gcs/ui/demo/DemoActivity.kt`: 데모 화면 Activity
- `app/src/main/java/com/example/gcs/ui/demo/DemoViewModel.kt`: 데모 화면 ViewModel
- `app/src/main/java/com/example/gcs/ui/demo/DemoAdapter.kt`: RecyclerView 어댑터
- `app/src/main/res/layout/activity_demo.xml`: 데모 화면 레이아웃
- `app/src/main/res/layout/item_demo.xml`: 데모 목록 항목 레이아웃

## 주요 명령어 (Commands)
이 프로젝트는 Gradle을 사용하므로 Gradle Wrapper(`gradlew`)를 통해 명령을 실행합니다.

- **디버그 APK 빌드:**
  ```bash
  ./gradlew assembleDebug
  ```
- **단위 테스트 실행:**
  ```bash
  ./gradlew test
  ```
- **계측 테스트(Instrumented Test) 실행:**
  ```bash
  ./gradlew connectedAndroidTest
  ```
- **린트(Lint) 체크:**
  ```bash
  ./gradlew lint
  ```

## 개발 규칙 (Development Conventions)
- **Kotlin:** 표준 Kotlin 코딩 컨벤션을 따릅니다.
- **리소스 명명:** 레이아웃 및 드로어블 파일은 `snake_case`를 사용합니다 (예: `activity_main.xml`).
- **패키지 명:** `com.example.gcs`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gopar-citus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gopar-citus)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
