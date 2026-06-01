## composetoglance

> 이 프로젝트는 Android 위젯 빌더 시스템으로, Compose DSL을 사용하여 Glance App Widget을 생성하는 것을 목표로 합니다.

# ComposeToGlance (WidgetKit) - Cursor Rules

## 프로젝트 개요

이 프로젝트는 Android 위젯 빌더 시스템으로, Compose DSL을 사용하여 Glance App Widget을 생성하는 것을 목표로 합니다.

**핵심 개념:**
- **DSL → Proto → (Glance or RemoteViews)**: Compose DSL로 작성한 위젯을 Proto Buffer로 직렬화하고, Glance 또는 RemoteViews로 렌더링
- **컴포넌트 기반 아키텍처**: 재사용 가능한 위젯 컴포넌트 시스템
- **실시간/주기적 업데이트**: BroadcastReceiver와 WorkManager를 통한 위젯 업데이트
- **드래그 앤 드롭 에디터**: 시각적 위젯 편집 UI

## 모듈 구조

### 1. app 모듈
**책임**: 메인 애플리케이션 및 위젯 에디터 UI

**주요 파일:**
- `MainActivity.kt`: 앱 진입점, Renderer 초기화, 위젯 컴포넌트 등록
- `MainContent.kt`: 메인 화면 구성
- `editor/`: 위젯 에디터 관련 코드
  - `WidgetEditorViewModel.kt`: 에디터 상태 관리
  - `canvas/WidgetCanvas.kt`: 드래그 앤 드롭 캔버스
  - `draganddrop/`: 드래그 앤 드롭 로직
- `service/WidgetForegroundService.kt`: BroadcastReceiver 관리 (배터리 등)

**주의사항:**
- `MainActivity.onCreate()`에서 `RendererInitializer.initialize()` 필수
- `initializeWidgetComponents()`로 모든 위젯 컴포넌트 등록

### 2. core 모듈
**책임**: DSL → Glance(or Remoteviews) 매핑 시스템의 핵심

**주요 패키지:**
- `dsl/proto/`: Proto 기반 DSL 빌더
  - `component/`: 컴포넌트 DSL (TextDsl, ImageDsl, ButtonDsl 등)
  - `layout/`: 레이아웃 DSL (BoxDsl, ColumnDsl, RowDsl 등)
  - `modifier/`: Modifier DSL
  - `property/`: 속성 DSL (ViewPropertyDsl, TextPropertyDsl 등)
- `dsl/frontend/`: 최상위 DSL 함수 (Text, Image, Box 등)
- `dsl/widget/`: 렌더링 시스템
  - `node/`: NodeRenderer 구현 및 Registry
  - `render/glance/`: Glance 렌더러
  - `render/remoteviews/`: RemoteViews 렌더러 (대체 구현)
  - `render/glance/converter/`: Proto → Glance 변환기

**핵심 클래스:**
- `WidgetScope`: DSL 실행 컨텍스트, children 관리, WidgetLocal 관리
- `RenderNodeRegistry`: NodeRenderer 등록 및 조회
- `RendererInitializer`: 기본 Renderer 등록

**Proto 파일:**
- `widget_dsl.proto`: 위젯 레이아웃 직렬화 스키마

### 3. widget 모듈
**책임**: 실제 위젯 컴포넌트 구현

**주요 패키지:**
- `component/`: 위젯 컴포넌트들
  - `WidgetComponent.kt`: 모든 컴포넌트의 베이스 클래스
  - `battery/`: 배터리 위젯 (실시간 업데이트 예제)
  - `devicecare/`: 디바이스 케어 위젯 (주기적 업데이트 예제)
  - `reminder/today/`: 할 일 위젯
  - `datastore/ComponentDataStore.kt`: 컴포넌트 상태 저장 추상 클래스
  - `update/ComponentUpdateManager.kt`: 업데이트 로직 인터페이스
  - `viewid/`: View ID 관리
- `database/`: Room Database (위젯 레이아웃 저장)
- `provider/`: Glance AppWidgetProvider 구현
- `repository/`: 데이터 저장소

**핵심 인터페이스:**
- `WidgetComponent`: 모든 위젯 컴포넌트의 베이스
- `ComponentUpdateManager<T>`: 업데이트 로직
- `ComponentDataStore<T>`: 상태 저장
- `ViewIdProvider`: View ID 생성

## 아키텍처 패턴

### 1. Component-Based Architecture
- `WidgetComponent` 추상 클래스를 상속하여 컴포넌트 구현
- 각 컴포넌트는 독립적인 패키지에 위치
- `WidgetScope.Content()`에서 UI 정의

### 2. Registry Pattern
- `WidgetComponentRegistry`: 위젯 컴포넌트 등록/조회
- `RenderNodeRegistry`: NodeRenderer 등록/조회
- `ViewIdAllocator`: View ID 자동 할당

### 3. Template Method Pattern
- `WidgetComponent.renderContent()`: Content 실행 템플릿
- `WidgetComponent.generateViewId()`: View ID 생성 템플릿

### 4. Strategy Pattern
- `ComponentUpdateManager<T>`: 업데이트 전략 인터페이스
- `ComponentDataStore<T>`: 저장 전략 추상 클래스

### 5. Repository Pattern
- `WidgetLayoutRepository`: 위젯 레이아웃 저장/조회
- Room Database를 통한 영속성

## 코딩 컨벤션

### 패키지 구조
```
com.widgetkit.{module}.{feature}
```

예시:
- `com.widgetkit.dsl.widget.node.component` (core 모듈)
- `com.widgetkit.core.component.battery` (widget 모듈)
- `com.widgetkit.widget.editor.canvas` (app 모듈)

### 클래스 네이밍
- **WidgetComponent**: `{Name}Widget` (예: `BatteryWidget`)
- **UpdateManager**: `{Name}UpdateManager` (예: `BatteryUpdateManager`)
- **DataStore**: `{Name}ComponentDataStore` (예: `BatteryComponentDataStore`)
- **ViewIdType**: `{Name}ViewIdType` (예: `BatteryViewIdType`)
- **Renderer**: `{Name}Node` (예: `TextNode`, `BoxNode`)

### 파일 구조 규칙
위젯 컴포넌트는 다음 구조를 따릅니다:
```
component/{componentName}/
├── {ComponentName}Widget.kt          # 필수: GUI 정의
├── {ComponentName}ViewIdType.kt      # 선택: Partially update 필요시
├── {ComponentName}UpdateManager.kt   # 선택: 업데이트 필요시
├── {ComponentName}DataStore.kt      # 선택: 상태 저장 필요시
└── {ComponentName}Data.kt            # 선택: 데이터 모델
```

### Kotlin 컨벤션
- **Extension Functions**: DSL 함수는 `WidgetScope` extension으로 정의
- **Sealed Classes**: ViewIdType은 sealed class로 정의
- **Objects**: UpdateManager, DataStore는 object로 구현 (싱글톤)
- **Data Classes**: 데이터 모델은 data class 사용

### Proto 필드 네이밍
- **snake_case**: Proto 필드는 snake_case 사용
- **필수 필드**: `view_property`는 모든 컴포넌트에 필수
- **oneof**: WidgetNode의 payload는 oneof로 정의

## 주요 클래스 및 인터페이스

### WidgetComponent
```kotlin
abstract class WidgetComponent : ViewIdProvider {
    abstract fun getName(): String
    abstract fun getDescription(): String
    abstract fun getWidgetCategory(): WidgetCategory
    abstract fun getSizeType(): SizeType
    abstract fun getWidgetTag(): String  // 고유 식별자
    abstract fun WidgetScope.Content()   // UI 정의
    
    // 선택적 오버라이드
    override fun getViewIdTypes(): List<ViewIdType> = emptyList()
    abstract fun getUpdateManager(): ComponentUpdateManager<*>?
    open fun getDataStore(): ComponentDataStore<*>? = null
}
```

### WidgetScope
- DSL 실행 컨텍스트
- `children`: WidgetNode 리스트 관리
- `locals`: WidgetLocal 값 저장
- `nextViewId()`: View ID 생성

### ComponentUpdateManager
```kotlin
interface ComponentUpdateManager<T> {
    val widget: WidgetComponent
    suspend fun syncState(context: Context, data: T)
    suspend fun updateByState(context: Context, data: T)
    suspend fun updateByPartially(context: Context, data: T)
}
```

### ComponentDataStore
```kotlin
abstract class ComponentDataStore<T> {
    abstract val datastoreName: String
    abstract suspend fun saveData(context: Context, data: T)
    abstract suspend fun loadData(context: Context): T
    abstract fun getDefaultData(): T
}
```

## 컴포넌트 확장 가이드

### 새로운 위젯 컴포넌트 추가

1. **WidgetComponent 상속 클래스 생성**
   - `{Name}Widget.kt` 파일 생성
   - 필수 메서드 구현: `getName()`, `getDescription()`, `getWidgetCategory()`, `getSizeType()`, `getWidgetTag()`, `Content()`

2. **View ID 타입 정의** (partially update 필요시)
   - `{Name}ViewIdType.kt`: sealed class로 정의
   - `getViewIdTypes()` 오버라이드

3. **UpdateManager 구현** (업데이트 필요시)
   - `{Name}UpdateManager.kt`: object로 구현
   - `ComponentUpdateManager<T>` 인터페이스 구현

4. **DataStore 구현** (상태 저장 필요시)
   - `{Name}DataStore.kt`: object로 구현
   - `ComponentDataStore<T>` 추상 클래스 상속
   - `datastoreName`은 고유한 값 사용 (예: `"{name}_pf"`)

5. **등록**
   - `WidgetComponentRegistry.kt`의 `initializeWidgetComponents()`에 추가

### 새로운 DSL 컴포넌트 추가 (core 모듈)

1. **Proto 정의** (`widget_dsl.proto`)
   - `WidgetNode`의 `oneof payload`에 새 필드 추가
   - Property 메시지 정의 (예: `NewComponentProperty`)
   - `view_property` 필드는 필수

2. **DSL 빌더 생성**
   - `dsl/proto/component/{Name}Dsl.kt`: Property DSL 빌더
   - `dsl/frontend/{Name}.kt`: 최상위 DSL 함수

3. **Renderer 구현**
   - `dsl/widget/node/component/{Name}Node.kt`: RenderNode 구현
   - `dsl/widget/render/glance/render/Glance{Name}.kt`: Glance 렌더링

4. **등록**
   - `RendererInitializer.initialize()`에 추가

## 주의사항 및 제약사항

### View ID 관리
- `ViewIdAllocator`가 자동으로 ID 범위 할당
- `getWidgetTag()`는 반드시 고유한 값 반환
- `generateViewId()`는 `WidgetComponentRegistry`를 통해 Base ID 조회

### BroadcastReceiver 관리
- **중요**: BroadcastReceiver는 `WidgetForegroundService`에서 관리
- 컴포넌트에서 직접 등록하지 않음
- `getLifecycle()`은 null 반환, `requiresAutoLifecycle()`은 false

### WorkManager 사용
- 주기적 업데이트가 필요한 경우 `ComponentLifecycle` 구현
- `{Name}Lifecycle.kt`에서 WorkManager 등록
- `requiresAutoLifecycle()`은 true 반환

### DataStore 네이밍
- `datastoreName`은 고유해야 함
- 형식: `"{component_name}_pf"` (preferences)
- Context extension property로 정의: `private val Context.dataStore by preferencesDataStore(name = datastoreName)`

### Proto 필드 번호
- `oneof` 내의 필드 번호는 고유해야 함
- 기존 필드 번호와 충돌하지 않도록 확인

### WidgetLocal 사용
- `WidgetScope`에서 `getLocal()`로 값 조회
- `setLocal()`로 값 설정
- 주요 WidgetLocal:
  - `WidgetLocalState`: 위젯 상태 (GlanceAppWidgetState)
  - `WidgetLocalGridIndex`: 그리드 인덱스
  - `WidgetLocalSize`: 위젯 크기
  - `WidgetLocalTheme`: 테마 정보
  - `WidgetLocalPreview`: 프리뷰 모드 여부

### Partially Update
- `.viewId()`와 `.partiallyUpdate(true)` 모디파이어 사용
- `UpdateManager.updateByPartially()`에서 RemoteViews로 업데이트
- Glance State와 RemoteViews 동기화 필요

## 테스트 및 디버깅

### 초기화 확인
- `RendererInitializer.isInitialized()`: Renderer 등록 확인
- `WidgetComponentRegistry.getAllComponents()`: 컴포넌트 등록 확인

### Proto 재생성
- Gradle 빌드 시 자동 생성
- `build/generated/source/proto/` 확인

### 로그 태그
- 각 컴포넌트는 고유한 TAG 사용
- 예: `"BatteryWidget"`, `"WidgetComponentRegistry"`

## 코드 작성 시 체크리스트

새 컴포넌트/기능 추가 시:
- [ ] 패키지 구조가 올바른가?
- [ ] 네이밍 컨벤션을 따르는가?
- [ ] 필요한 인터페이스/추상 클래스를 구현했는가?
- [ ] Registry에 등록했는가?
- [ ] View ID가 필요한 경우 정의했는가?
- [ ] DataStore 이름이 고유한가?
- [ ] Proto 필드 번호가 고유한가?
- [ ] BroadcastReceiver는 WidgetForegroundService에 등록했는가?

---
> Source: [chc0331/ComposeToGlance](https://github.com/chc0331/ComposeToGlance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
