## vocahang

> 이 문서는 VocaHang 프로젝트의 코드베이스 일관성과 유지보수성을 위한 규칙을 정의합니다.

# VocaHang 프로젝트 규칙

이 문서는 VocaHang 프로젝트의 코드베이스 일관성과 유지보수성을 위한 규칙을 정의합니다.

## 1. 프로젝트 구조

- 소스 코드는 `src` 디렉토리에 위치합니다.
- `src` 하위 디렉토리는 다음과 같이 구성됩니다:
    - `screens/`: 화면 단위 컴포넌트
    - `components/`: 재사용 가능한 UI 컴포넌트
    - `constants/`: 앱 전역 상수 (색상, 테마 등)
    - `types/`: TypeScript 타입 정의
    - `hooks/`: (필요시) 커스텀 React Hooks
    - `utils/`: (필요시) 유틸리티 함수
- 메인 애플리케이션 진입점은 [App.tsx](mdc:App.tsx) 입니다.

## 2. 코드 스타일

- **컴포넌트**: 함수형 컴포넌트를 사용합니다.
- **스타일링**: 컴포넌트 파일 하단에 `StyleSheet.create`를 사용하여 스타일을 정의합니다.
- **Props**: 컴포넌트 Props는 항상 TypeScript 인터페이스로 정의합니다.
- **Linter/Formatter**: ESLint와 Prettier 설정을 준수합니다. 관련 설정은 [.eslintrc.js](mdc:.eslintrc.js)를 참고하세요.

## 3. 상태 관리

- **로컬 상태**: 각 컴포넌트의 개별 상태는 `useState`를 사용합니다.
- **전역 상태**: 여러 컴포넌트에서 공유되는 상태는 React Context API를 활용합니다.
    - 핵심 게임 로직 및 상태는 [src/components/GameProvider.tsx](mdc:src/components/GameProvider.tsx) (존재한다면)에 중앙화되어 관리됩니다.
    - `GameProvider`는 React Context API (`createContext`, `useContext`)를 사용하여 게임 상태와 액션을 컴포넌트에 제공합니다.
    - 주요 관리 상태 예시: `currentWord`, `solvedWordIds`, 게임 통계 (`wins`, `losses`, `streak`), `keyboardLayout`, `gameStatus`, `currentGrade`, `wordList`.
    - 핵심 함수 예시: `loadWordList`, `pickNewWord`, `handlePressLetter`, `saveGameState`, `loadGameState`.
    - 데이터 영속성: `AsyncStorage` (예: `@react-native-async-storage/async-storage`)를 사용하여 게임 상태 (통계, 설정)를 저장하고 불러올 수 있습니다.
    - 사운드 효과: `expo-av` 등을 사용하여 사운드 로딩 및 재생을 관리할 수 있습니다.
    - 게임 규칙, 플레이어 통계 또는 단어 처리 수정 시 `GameProvider`를 주로 확인합니다.
- **복잡한 상태 로직**: 복잡한 상태 업데이트 로직은 `useReducer`를 고려합니다.

## 4. 네이밍 컨벤션

- **컴포넌트 파일명 및 이름**: `PascalCase` (예: `IntroScreen.tsx`, `HowToPlayScreen`)
- **함수 및 변수**: `camelCase` (예: `handlePress`, `userData`)
- **상수**: `UPPER_SNAKE_CASE` (예: `MAX_USERS`, `API_URL`)
- **타입/인터페이스**: `PascalCase` (접두사 `I` 또는 `T` 없이, 예: `User`, `GameSettings`)

## 5. 임포트 순서

가독성을 위해 다음 순서로 임포트합니다:
1. 외부 라이브러리 (예: `react`, `@react-navigation/native`)
2. 내부 컴포넌트 (예: `../components/Button`)
3. 내부 타입 정의 (예: `../types/navigation`)
4. 내부 유틸리티/상수 (예: `../constants/theme`)

## 6. 성능 최적화

- 불필요한 리렌더링을 방지하기 위해 `React.memo`를 적절히 사용합니다.
- 함수와 객체의 재생성을 막기 위해 `useCallback`과 `useMemo`를 활용합니다.
- 큰 컴포넌트는 논리적으로 더 작은 단위로 분리합니다.

## 7. 테스트

- 단위 테스트는 `Jest`를 사용하여 작성합니다. 테스트 설정은 [jest.config.js](mdc:jest.config.js)를 참고하세요.
- 주요 비즈니스 로직과 핵심 기능은 반드시 테스트 케이스를 작성합니다.

## 8. 문서화

- 복잡하거나 이해하기 어려운 로직에는 주석을 작성하여 설명을 추가합니다.
- 프로젝트 전반에 대한 정보는 [README.md](mdc:README.md)에 최신 상태로 유지합니다.

## 9. Firebase 사용

- Firebase는 웹 애플리케이션 배포 및 Expo 앱 빌드/배포를 위해 사용됩니다.
- Firebase 관련 설정 파일은 [firebase.json](mdc:firebase.json) 입니다.
- 관련 의존성은 [package.json](mdc:package.json)에 명시되어 있습니다. (예: `@react-native-firebase/app`, `@react-native-firebase/analytics`)
- Firebase 서비스 계정 키와 같은 민감한 정보는 `config/` 디렉토리에 위치할 수 있으나, 버전 관리 시스템에는 포함되지 않도록 주의합니다. ([config/google-service-account-key.json](mdc:config/google-service-account-key.json) - 현재는 삭제되었을 수 있음)

## 10. API 연동 (향후 확장 가능성)

- 현재는 프론트엔드 앱으로만 구성되지만, 향후 백엔드 API 연동 시 다음 규칙을 고려합니다:
    - API 호출 로직은 별도의 서비스 레이어(예: `src/services/api.ts`)로 분리합니다.
    - API 요청/응답에 대한 타입 정의를 철저히 합니다.
    - 네트워크 상태(로딩, 성공, 실패)에 따른 UI 피드백을 제공합니다.
    - API 오류 처리를 일관된 방식으로 구현합니다.

## 11. 라이브러리 및 기능 상세 가이드

### 11.1. 네비게이션 (`@react-navigation/native-stack`)

앱 네비게이션은 `@react-navigation/native-stack`을 사용하여 관리됩니다.

- 주요 **Stack Navigator**는 [App.tsx](mdc:App.tsx)에 설정되어 있습니다.
- 애플리케이션의 모든 주요 화면을 정의합니다: `Intro`, `VocaMan`, `HowToPlay`, `Dashboard`, `Settings`, `AppInfo`, `Support`.
- `initialRouteName`은 `Intro`로 설정되어 있습니다.
- 전역 네비게이션 바 (사이드 메뉴) 컴포넌트 [src/components/GNB.tsx](mdc:src/components/GNB.tsx) (존재한다면)는 `App.tsx`의 상태를 통해 제어되며, [src/components/Header.tsx](mdc:src/components/Header.tsx) (존재한다면)에 의해 트리거될 수 있습니다.
- 화면 전환은 일반적으로 `useNavigation` 훅에서 얻은 `navigation.navigate()` 메소드를 사용하여 처리됩니다.

### 11.2. `expo-av` Video 컴포넌트 스타일링 (웹 및 `IntroScreen.tsx` 중심)

`expo-av`의 `Video` 컴포넌트를 스타일링할 때, 특히 웹 환경 및 `[src/screens/IntroScreen.tsx](mdc:src/screens/IntroScreen.tsx)` 파일에서의 사용 사례를 중심으로 중요한 점들은 다음과 같습니다.

#### 주요 스타일링 Props

1.  **`style` Prop**:
    *   React Native의 `<Video>` 컴포넌트에 적용된 `style` prop은 웹으로 렌더링될 때, 실제 HTML `<video>` 태그를 감싸는 부모 `div` 컨테이너에 주로 적용됩니다.
    *   이 prop을 사용하여 비디오 플레이어 전체의 레이아웃, 크기 (예: `width`, `aspectRatio`), 여백 등을 설정할 수 있습니다.

2.  **`videoStyle` Prop**:
    *   이 prop은 내부 HTML `<video>` 요소 자체에 직접 스타일을 전달할 때 사용합니다.
    *   웹 환경에서 비디오 *내용물*의 크기를 부모 컨테이너에 정확히 맞추고자 할 때 (예: `width: '100%', height: '100%'`) 유용합니다. `[src/screens/IntroScreen.tsx](mdc:src/screens/IntroScreen.tsx)`에서 이 방식을 사용했습니다.

3.  **`resizeMode` Prop**:
    *   `ResizeMode.CONTAIN`, `ResizeMode.COVER` 등의 값을 가집니다.
    *   CSS의 `object-fit` 속성과 유사하게, 비디오 내용물이 `videoStyle` 또는 `style`로 정의된 영역 내에서 원본 비율을 유지하면서 어떻게 크기 조절되고 배치될지를 결정합니다.
    *   `ResizeMode.CONTAIN`: 비디오 전체가 보이도록 내용물을 축소/확대합니다. 비율이 다르면 레터박스가 생길 수 있습니다.
    *   `ResizeMode.COVER`: 영역을 완전히 덮도록 내용물을 축소/확대하며, 비율이 다르면 일부가 잘릴 수 있습니다.

#### `IntroScreen.tsx` 적용 사례

`[src/screens/IntroScreen.tsx](mdc:src/screens/IntroScreen.tsx)` 파일에서는 다음 전략을 사용했습니다:

*   외부 컨테이너(`div`)의 크기를 `style` prop (예: `width: '50%', aspectRatio: 16/9`)으로 지정했습니다.
*   내부 비디오 내용물(HTML `<video>`)이 이 컨테이너를 완전히 채우도록 `videoStyle={{ width: '100%', height: '100%' }}`을 적용했습니다.
*   비디오의 원본 비율을 유지하면서 컨테이너에 맞게 내용물이 표시되도록 `resizeMode={ResizeMode.CONTAIN}`을 사용했습니다.

이 조합을 통해 개발자 도구에서 직접 내부 `<video>` 태그에 `width: 100%`를 적용했을 때와 동일한 시각적 결과를 코드 레벨에서 달성할 수 있었습니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hiwony7933) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
