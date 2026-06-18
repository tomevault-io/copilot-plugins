## csslearning

> 주어진 내용을 바탕으로 CSS 시험을 준비한다면, 다음의 핵심 영역들을 깊이 있게 학습해야 합니다.

CSS 시험 대비 핵심 학습 내용
주어진 내용을 바탕으로 CSS 시험을 준비한다면, 다음의 핵심 영역들을 깊이 있게 학습해야 합니다.

1. 레이아웃 디자인 (Layout Design)
   웹 페이지의 구조를 잡는 가장 중요한 부분입니다.

- Flexbox: 1차원 레이아웃(행 또는 열)을 다루는 데 필수적입니다. 아이템 정렬, 간격 배분 등 다양한 속성을 숙지해야 합니다.
- Grid: 2차원 레이아웃(행과 열)을 다루는 데 강력합니다. 복잡한 그리드 시스템을 만들고 아이템을 배치하는 방법을 이해해야 합니다.
- Positioning (static, relative, absolute, fixed, sticky): 요소의 위치를 제어하는 다양한 방법을 알고 각 상황에 맞는 적절한 사용법을 익혀야 합니다.
- Float & Clear: 과거 레이아웃에 많이 사용되었던 방식이므로, 개념과 사용법, 그리고 clear 속성을 통한 문제 해결 방법을 알아두는 것이 좋습니다. (현대에는 Flexbox나 Grid를 더 선호하지만, 기존 코드 이해를 위해 필요합니다.)
- Display 속성 (block, inline, inline-block, none 등): 요소의 렌더링 방식을 이해하고 적절히 활용하는 방법을 알아야 합니다.

2. 스타일링 구현 (Styling Implementation)
   시각적인 요소를 꾸미는 데 필요한 다양한 CSS 속성들입니다.

- 선택자 (Selectors): HTML 요소를 정확하게 선택하기 위한 다양한 선택자(클래스, ID, 태그, 자식, 후손, 속성, 가상 클래스, 가상 요소 등)와 선택자 우선순위(Specificity)를 완벽하게 이해해야 합니다.
- 텍스트 및 글꼴: font-family, font-size, font-weight, color, text-align, line-height, text-decoration 등 텍스트 관련 속성들을 알아야 합니다.
- 배경: background-color, background-image, background-repeat, background-position, background-size 등 배경 관련 속성들을 이해해야 합니다.
- 박스 모델 (Box Model): margin, border, padding, content의 개념과 각 속성이 요소의 크기와 공간에 미치는 영향을 정확히 이해해야 합니다. box-sizing 속성(content-box, border-box)도 중요합니다.
- 색상 및 단위: HEX, RGB, RGBA, HSL, HSLA와 같은 색상 표기법 및 px, em, rem, %, vw, vh 등 다양한 단위를 이해하고 적절히 활용할 수 있어야 합니다.
- 트랜지션(Transitions) 및 애니메이션(Animations): 부드러운 UI 변화를 위한 기본적인 트랜지션과 애니메이션 효과 구현 방법을 알아두면 좋습니다.

3. 반응형 프레임워크 (Responsive Frameworks) 이해
   직접 프레임워크를 개발하는 것보다는, 주요 개념과 작동 방식을 이해하는 것이 중요합니다.

- 미디어 쿼리 (Media Queries): 기기의 특성(화면 너비, 높이 등)에 따라 다른 스타일을 적용하는 핵심 기술입니다. min-width, max-width 등을 활용하여 반응형 디자인을 구현하는 방법을 숙지해야 합니다.
- 모바일 우선 (Mobile-First) 디자인 원칙: 작은 화면에서 큰 화면으로 확장하는 방식으로 디자인하는 개념을 이해하고 적용할 수 있어야 합니다.
- 유동적인 이미지/비디오: max-width: 100%; 등을 활용하여 다양한 화면 크기에 맞춰 이미지나 비디오가 유동적으로 조절되도록 하는 방법을 알아야 합니다.

4. 현대 CSS 기능 및 크로스 브라우징 호환성
   최신 CSS 트렌드와 웹 표준에 대한 이해가 필요합니다.

- CSS 변수 (Custom Properties / CSS Variables): 재사용 가능한 값을 정의하여 CSS를 더욱 효율적으로 관리하는 방법입니다.
- calc() 함수: CSS 내에서 계산을 수행하여 동적인 값을 설정하는 방법입니다.
- 크로스 브라우징 (Cross-Browser Compatibility): 다른 웹 브라우저(Chrome, Firefox, Safari, Edge 등)에서 웹 페이지가 일관되게 보이도록 하는 기술적 접근 방법입니다.
- 벤더 프리픽스 (Vendor Prefixes): 특정 브라우저에서만 지원되는 실험적인 CSS 속성을 사용하기 위한 접두사(-webkit-, -moz-, -ms-)의 개념을 이해해야 합니다. (최근에는 사용 빈도가 줄었지만, 기존 코드나 특정 상황에서 필요할 수 있습니다.)
- CSS 전처리기 (Preprocessors) 개념: SASS/SCSS, LESS 등의 CSS 전처리기가 무엇이며 어떤 장점이 있는지(변수, 믹스인, 네스팅 등) 개념적으로 이해하고 있다면 좋습니다. (시험 범위에 따라 직접 사용법까지 요구될 수도 있습니다.)

---
> Source: [saramjh/csslearning](https://github.com/saramjh/csslearning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
