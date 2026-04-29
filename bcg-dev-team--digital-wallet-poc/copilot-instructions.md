## digital-wallet-poc

> | Tailwind 포맷/유틸리티 | packages/ui/tailwind.config.cjs, packages/theme/tailwind.config.cjs |

# Tailwind CSS 베스트 프랙티스

## 1. 참고 파일 안내

| 상황/패턴 | 참고 파일 |
|---|---|
| Tailwind 포맷/유틸리티 | packages/ui/tailwind.config.cjs, packages/theme/tailwind.config.cjs |
| 디자인 토큰(CSS 변수) | packages/theme/src/styles/__tokens-light.css, __tokens-dark.css |
| SCSS(Sass) | 각 컴포넌트의 .scss 파일, packages/ui/src/style.css |
| 하이브리드/동적 스타일 | 위 모든 파일 + 컴포넌트 내 computed/style 로직 |

> **참고:**
> - Tailwind 커스텀 유틸리티/색상/spacing 등은 반드시 tailwind.config.cjs에서 확인/추가
> - 디자인 토큰(CSS 변수)은 __tokens-light.css, __tokens-dark.css에서 확인
> - SCSS에서 @apply, CSS 변수 사용 시 위 파일 경로 참고

---

## 2. 스타일 적용 기본 원칙
- **Tailwind 클래스로 정의된 것들은 Vue 파일에서 바로 사용한다.**
- **SCSS에서는 @apply를 사용하지 않고 CSS 변수를 직접 사용한다.**
- **미리 정의된 클래스 방식을 우선 사용하여 개발자 도구 가독성을 높인다.**
- **조건부 변경이 필요한 경우만 :style로 동적 처리한다.**
- **SCSS(Sass)를 적극 활용하여 컴포넌트별 스타일을 분리한다.**
- **디자인 토큰은 CSS 변수로 관리하고, 컴포넌트별 토큰만 SCSS에서 활용한다.**

---

## 3. CSS 클래스 방식 가이드

### 3.1 미리 정의된 클래스 방식 (권장)
개발자 도구에서 복잡한 클래스명을 피하고 가독성을 높이기 위해 미리 정의된 클래스를 사용하세요.

```vue
<!-- ❌ 피해야 할 방식 (동적 CSS 변수 클래스) -->
<button :class="`bg-[var(--button-${variant}-background)] text-[var(--button-${variant}-text)]`">
  버튼
</button>

<!-- ✅ 권장 방식 (미리 정의된 클래스) -->
<button :class="`btn-${variant}`">
  버튼
</button>
```

### 3.2 SCSS 파일 분리 패턴
컴포넌트별로 별도의 SCSS 파일을 생성하여 스타일을 관리하세요. SCSS에서는 컴포넌트별 토큰만 사용하고, Tailwind 클래스는 Vue 파일에서 직접 사용합니다.

```vue
<!-- BaseButton.vue -->
<script setup lang="ts">
import './BaseButton.scss'; // SCSS 파일 import
// ... 컴포넌트 로직
</script>

<template>
  <button :class="`btn-${variant}`">
    {{ label }}
  </button>
</template>
```

```scss
/* BaseButton.scss - 컴포넌트별 토큰만 사용 */
.btn-primary {
  background-color: var(--button-primary-background);
  color: var(--button-primary-text);
}

.btn-primary:hover {
  background-color: var(--button-primary-background-deep);
}

.btn-outline {
  background-color: transparent;
  color: var(--button-outline-text);
  border-color: var(--button-outline-border);
}

.btn-outline:hover {
  background-color: var(--button-outline-background);
}
```

### 3.3 클래스 매핑 패턴
Vue 컴포넌트에서 variant별 클래스 매핑을 사용하세요.

```typescript
const variantClasses = {
  primary: 'btn-primary',
  outline: 'btn-outline',
  red: 'btn-red',
  blue: 'btn-blue',
  // ... 기타 variant
};

const staticClasses = computed(() => {
  const classes = [
    'inline-flex items-center justify-center font-sans font-semibold transition-all',
    // ... 기본 클래스들
  ];
  
  classes.push(variantClasses[props.variant] || 'btn-primary');
  return classes.join(' ');
});
```

### 3.4 장점
- **개발자 도구 가독성**: `btn-primary` 같은 깔끔한 클래스명
- **성능**: 동적 클래스 생성보다 정적 클래스가 효율적
- **유지보수성**: 스타일을 별도 파일로 분리하여 관리 용이
- **디버깅**: 특정 variant의 스타일 문제를 쉽게 찾을 수 있음

---

## 4. 상황별 권장 방식

| 상황 | 권장 방식 | 참고 파일 |
|------|-----------|-----------|
| 단순 색상/값 적용 | 미리 정의된 클래스 + SCSS | 컴포넌트.scss |
| 조건부 색상 변경 | 동적 CSS 변수 (:style) | __tokens-light.css, 컴포넌트 |
| 반응형/상태 클래스 | Tailwind 클래스 | tailwind.config.cjs |
| 복잡한 조건부 스타일 | 하이브리드 | tailwind.config.cjs, __tokens-light.css, 컴포넌트 |
| 정적 컴포넌트 스타일 | 미리 정의된 클래스 + SCSS | 컴포넌트.scss |

---

## 5. 상황별 스타일 적용 예시

### 5.1 미리 정의된 클래스 + SCSS (권장)
```vue
<!-- Tailwind 클래스는 Vue 파일에서 직접 사용 -->
<!-- 컴포넌트별 토큰은 SCSS에서 정의 -->
<button :class="`btn-${variant} inline-flex items-center justify-center px-4 py-2 rounded-lg font-semibold transition-all`">
  {{ label }}
</button>
```

```scss
/* BaseButton.scss - 컴포넌트별 토큰만 사용 */
.btn-primary {
  background-color: var(--button-primary-background);
  color: var(--button-primary-text);
  border-color: var(--button-primary-border);
}

.btn-primary:hover {
  background-color: var(--button-primary-background-deep);
}

.btn-primary:disabled {
  background-color: var(--button-disabled-background);
}
```

### 5.2 동적 CSS 변수 (:style) - 조건부 변경 시만
```vue
<!-- 조건부 색상 변경이 필요한 경우만 -->
<!-- 참고: __tokens-light.css, 컴포넌트 내 computed -->
<script setup>
const buttonStyle = computed(() => {
  if (isDisabled.value) {
    return {
      backgroundColor: 'var(--button-disabled-background)',
      color: 'var(--button-disabled-text)'
    };
  }
  return {
    backgroundColor: `var(--button-${props.variant}-background)`,
    color: `var(--button-${props.variant}-text)`
  };
});
</script>
<template>
  <button :style="buttonStyle">
    {{ label }}
  </button>
</template>
```

### 5.3 하이브리드 접근법 (최적)
```vue
<!-- 미리 정의된 클래스 + 동적 조건부 스타일 -->
<!-- 참고: __tokens-light.css, 컴포넌트 내 computed -->
<script setup>
// 미리 정의된 클래스 사용
const variantClasses = {
  primary: 'btn-primary',
  outline: 'btn-outline',
  // ... 기타 variant
};

const staticClasses = computed(() => {
  const classes = [
    'inline-flex items-center justify-center font-sans font-semibold transition-all',
    // ... 기본 클래스들
  ];
  
  classes.push(variantClasses[props.variant] || 'btn-primary');
  return classes.join(' ');
});

const dynamicStyle = computed(() => {
  // 조건부 변경이 필요한 경우만
  if (borderVariants.includes(props.variant)) {
    return {
      borderWidth: '1px',
      borderStyle: 'solid',
    };
  }
  return {};
});
</script>
<template>
  <button
    :class="staticClasses"
    :style="dynamicStyle"
  >
    {{ label }}
  </button>
</template>
```

### 5.4 :class 사용 가이드
```vue
<!-- Tailwind 클래스는 Vue 파일에서 직접 사용 -->
<!-- 조건/상태/반응형에 따라 Tailwind 클래스 조합 -->
<!-- 참고: tailwind.config.cjs, __tokens-light.css -->
<template>
  <button
    :class="[
      'inline-flex items-center px-4 py-2 rounded-lg font-semibold transition-all',
      // 미리 정의된 클래스 사용 (컴포넌트별 토큰)
      `btn-${variant}`,
      // Tailwind 클래스 직접 사용
      isActive && 'bg-primary-primary100',
      isError && 'bg-red-red100',
      isDisabled && 'opacity-50 cursor-not-allowed',
      'md:px-6'
    ]"
    :disabled="isDisabled"
  >
    {{ label }}
  </button>
</template>
```

### 5.5 :style 사용 가이드
```vue
<!-- 동적으로 계산된 수치, 실시간 애니메이션 등 -->
<!-- 참고: __tokens-light.css, 컴포넌트 내 computed -->
<template>
  <div
    class="absolute shadow-lg"
    :style="{
      width: `${dynamicWidth}px`,
      top: `${positionTop}px`,
      left: `${positionLeft}px`,
      zIndex: zIndex,
      transform: `scale(${scale})`,
      backgroundColor: customColor // 사용자 입력값
    }"
  >
    {{ content }}
  </div>
</template>
```

### 5.6 SCSS(Sass) 활용 가이드
```scss
// 컴포넌트별 토큰만 사용, @apply 사용하지 않음
// 참고: style.css, __tokens-light.css
.button {
  // @apply 사용하지 않고 컴포넌트별 토큰만 사용
  border-radius: var(--button-radius, 8px);
  background-color: var(--button-primary-background);
  color: var(--button-primary-text);
  &:hover:not(:disabled) {
    background-color: var(--button-primary-background-deep);
  }
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
  @media (min-width: 768px) {
    padding-left: 24px;
    padding-right: 24px;
  }
}
```

---

## 6. 성능 최적화 가이드

### 6.1 미리 정의된 클래스 vs 동적 CSS 변수

#### ✅ 미리 정의된 클래스 (권장)
```vue
<!-- 성능 우수: CSS 클래스로 처리, 가독성 우수 -->
<button :class="`btn-${variant}`">
  버튼
</button>
```

#### ⚠️ 동적 CSS 변수 (조건부 변경 시만)
```vue
<!-- 성능 오버헤드: JavaScript 실행 필요 -->
<button :style="buttonStyle">
  버튼
</button>
```

### 6.2 성능 비교표

| 방식 | 성능 | 번들 크기 | 캐싱 | 가독성 | 사용 시기 |
|------|------|-----------|------|--------|-----------|
| 미리 정의된 클래스 | ✅ 우수 | ✅ 최적화 | ✅ 브라우저 캐싱 | ✅ 직관적 | 단순 색상/값 |
| 동적 CSS 변수 | ⚠️ 오버헤드 | ⚠️ JavaScript | ⚠️ 런타임 | ⚠️ 복잡 | 조건부 변경 |

---

## 7. 컴포넌트/유틸리티/반응형 패턴 예시 (디자인 토큰 활용)

### 7.1 컴포넌트 스타일 패턴
```vue
<!-- 참고: tailwind.config.cjs, __tokens-light.css -->
<script setup lang="ts">
import { computed } from 'vue'
import './BaseCard.scss';

const variantClasses = {
  default: 'card-default',
  elevated: 'card-elevated',
  outlined: 'card-outlined',
};
</script>
<template>
  <div :class="`card-${variant}`">
    <slot />
  </div>
</template>
```

### 7.2 유틸리티 클래스 컴포넌트
```scss
/* 참고: tailwind.config.cjs, __tokens-light.css */
@layer components {
  .btn {
    @apply inline-flex items-center justify-center px-4 py-2 text-sm font-medium rounded-lg transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed;
  }
  .btn-primary {
    @apply btn;
    background-color: var(--button-primary-background);
    color: var(--button-primary-text);
  }
  .btn-primary:hover {
    background-color: var(--button-primary-background-deep);
  }
  .btn-outline {
    @apply btn border-2 bg-transparent;
    border-color: var(--button-outline-border);
    color: var(--button-outline-text);
  }
  .btn-outline:hover {
    background-color: var(--button-outline-background);
  }
  /* ... 기타 버튼, 인풋, 카드, 배지 등도 동일하게 디자인 토큰 활용 ... */
}
```

### 7.3 반응형 디자인 패턴
```vue
<!-- 참고: tailwind.config.cjs, __tokens-light.css -->
<script setup lang="ts">
import { computed } from 'vue'
// ...생략 (ResponsiveLayout.vue 예시)...
</script>
<template>
  <div :class="layoutClasses">
    <!-- ... -->
  </div>
</template>
```

---

## 8. 실무 주의점 및 결론

### 8.1 성능 최적화 원칙
- **단순 색상/값 → 미리 정의된 클래스 + SCSS**
- **조건부 변경 → 동적 CSS 변수 (:style)**
- **:style 남용 금지**: 정말 필요한 경우에만 사용

### 8.2 코드 품질 원칙
- **SCSS와 Tailwind 혼용 시 네이밍/우선순위 관리**: SCSS에서 @apply 사용 시 Tailwind 버전과 호환성 체크
- **디자인 토큰 변경 시 빌드/적용 프로세스 명확화**: CSS 변수 자동화(Style Dictionary 등)와 연동
- **팀 내 코드리뷰/컨벤션 공유**: 예시 코드와 함께 팀 내 가이드로 문서화
- **컴포넌트별 SCSS 파일 분리**: 스타일 관리 용이성과 가독성 향상

### 8.3 최종 권장사항
- **Tailwind 클래스로 정의된 것들 → Vue 파일에서 직접 사용**
- **컴포넌트별 토큰 → SCSS에서 CSS 변수로 정의**
- **SCSS에서 @apply 사용하지 않음**
- **조건부 색상 변경 → computed로 :style 처리**
- **복잡한 구조/반복/미디어쿼리 → SCSS 적극 활용**
- **디자인 토큰 → CSS 변수로 관리, 컴포넌트별 토큰만 SCSS에서 활용**
- **개발자 도구 가독성 → 미리 정의된 클래스 방식 우선 사용**

---

## 9. 피그마 MCP 디자인 토큰 변환 가이드

### 9.1 피그마 → 디자인 토큰 변환 원칙
피그마 MCP를 통해 디자인을 가져올 때 **반드시 하드코딩된 HEX 값을 실제 존재하는 디자인 토큰으로 변환**해야 합니다.

#### **변환 전 (피그마에서 가져온 코드)**
```scss
// ❌ 하드코딩된 HEX 값 사용 금지
.btn-primary {
  background-color: #ffc300;
  color: #131313;
  border: 1px solid #b4b6bb;
}
```

#### **변환 후 (디자인 토큰 적용)**
```scss
// ✅ 실제 존재하는 디자인 토큰 사용
.btn-primary {
  background-color: var(--button-primary-background);
  color: var(--button-primary-text);
  border: 1px solid var(--button-primary-border);
}
```

### 9.2 실제 사용 가능한 토큰 매핑

#### **Base Colors (기본 색상)**
```scss
// 피그마 HEX → 실제 토큰 변환
#ffc300 → var(--base-colors-primary-primary800)
#131313 → var(--base-colors-neutral-neutral800)
#b4b6bb → var(--base-colors-neutral-neutral400)
#ffffff → var(--base-colors-neutral-neutral000)
#f63338 → var(--base-colors-red-red800)
#8f9299 → var(--base-colors-neutral-neutral500)
#caccce → var(--base-colors-neutral-neutral300)
```

#### **Button 컴포넌트 토큰**
```scss
// 피그마 HEX → Button 토큰 변환
#ffc300 → var(--button-primary-background)     // Button.Primary.background
#131313 → var(--button-primary-text)           // Button.Primary.text
#b4b6bb → var(--button-disabled-border)        // Button.Disabled.border
#f63338 → var(--button-red-solid-background)   // Button.Red-solid.background
#ffffff → var(--button-red-solid-text)         // Button.Red-solid.text
```

#### **Input 컴포넌트 토큰**
```scss
// 피그마 HEX → Input 토큰 변환
#ffffff → var(--input-color-surface)           // Input.Color.surface
#b4b6bb → var(--input-color-border-static)     // Input.Color.border-static
#ffc300 → var(--input-color-border-focus)      // Input.Color.border-focus
#f63338 → var(--input-color-border-error)      // Input.Color.border-error
#8f9299 → var(--input-color-text-placeholder)  // Input.Color.text-placeholder
#131313 → var(--input-color-text-static)       // Input.Color.text-static
```

#### **Checkbox/Radio 컴포넌트 토큰**
```scss
// 피그마 HEX → Checkbox/Radio 토큰 변환
#ffc300 → var(--input-check-radio-selected-bg)  // Input.Check & Radio.selected-bg
#ffffff → var(--input-check-radio-active-bg)   // Input.Check & Radio.active-bg
#b4b6bb → var(--input-check-radio-active-border) // Input.Check & Radio.active-border
#caccce → var(--input-check-radio-disable-bg)   // Input.Check & Radio.disable-bg
```

### 9.3 토큰 존재 확인 방법

#### **1. __tokens.json 확인**
```bash
# packages/theme/src/tokens/__tokens.json에서 토큰 정의 확인
grep -r "Button\|Input" packages/theme/src/tokens/__tokens.json
```

#### **2. CSS 변수 확인**
```bash
# packages/theme/src/styles/__tokens-light.css에서 CSS 변수 확인
grep -r "button\|input" packages/theme/src/styles/__tokens-light.css
```

#### **3. Tailwind 클래스 확인**
```bash
# packages/theme/tailwind.config.cjs에서 Tailwind 클래스 확인
grep -r "primary\|neutral\|input" packages/theme/tailwind.config.cjs
```

### 9.4 변환 프로세스

#### **Step 1: 피그마 코드 분석**
```scss
// 피그마에서 가져온 코드 분석
.btn-primary {
  background-color: #ffc300;  // HEX 값 식별
  color: #131313;             // HEX 값 식별
  border: 1px solid #b4b6bb;  // HEX 값 식별
}
```

#### **Step 2: 컨텍스트 기반 토큰 선택**
```scss
// 컴포넌트 컨텍스트에 따라 적절한 토큰 선택
.btn-primary {
  background-color: var(--button-primary-background);  // Button.Primary.background
  color: var(--button-primary-text);                  // Button.Primary.text
  border: 1px solid var(--button-primary-border);     // Button.Primary.border
}
```

#### **Step 3: SCSS 클래스 생성**
```scss
// 미리 정의된 클래스 방식으로 SCSS 생성
.btn-primary {
  background-color: var(--button-primary-background);
  color: var(--button-primary-text);
  border: 1px solid var(--button-primary-border);
  
  &:hover {
    background-color: var(--button-primary-background-deep);
  }
  
  &:disabled {
    background-color: var(--button-disabled-background);
    color: var(--button-disabled-text);
  }
}
```

### 9.5 예외 처리

#### **새로운 HEX 값 발견 시**
```scss
// 매핑이 불분명한 경우 주석으로 TODO 표시
.btn-new-variant {
  background-color: #newcolor; // TODO: 새로운 색상 - 적절한 토큰 매핑 필요
  color: var(--button-primary-text);
}
```

#### **토큰이 존재하지 않는 경우**
```scss
// 토큰이 존재하지 않는 경우 기본 색상 사용
.btn-fallback {
  background-color: var(--base-colors-neutral-neutral000); // 기본 흰색
  color: var(--base-colors-neutral-neutral800);           // 기본 검정
}
```

### 9.6 검증 체크리스트

피그마에서 디자인을 가져온 후 다음 사항을 확인하세요:

- [ ] 모든 하드코딩된 HEX 값이 디자인 토큰으로 변환되었는가?
- [ ] 컴포넌트 컨텍스트에 맞는 적절한 토큰을 선택했는가?
- [ ] `__tokens.json`에서 해당 토큰이 정의되어 있는가?
- [ ] `__tokens-light.css`에서 해당 CSS 변수가 생성되었는가?
- [ ] SCSS에서 미리 정의된 클래스 방식을 사용했는가?
- [ ] Vue 파일에서는 Tailwind 클래스를 사용했는가?

### 9.7 자주 사용되는 매핑 참조

```scss
// 자주 사용되는 피그마 HEX → 디자인 토큰 매핑
const COMMON_MAPPINGS = {
  // Primary Colors
  '#ffc300': 'var(--base-colors-primary-primary800)',
  '#de8100': 'var(--base-colors-primary-primary-deep)',
  
  // Neutral Colors
  '#131313': 'var(--base-colors-neutral-neutral800)',
  '#333740': 'var(--base-colors-neutral-neutral700)',
  '#5a5c5e': 'var(--base-colors-neutral-neutral600)',
  '#8f9299': 'var(--base-colors-neutral-neutral500)',
  '#b4b6bb': 'var(--base-colors-neutral-neutral400)',
  '#caccce': 'var(--base-colors-neutral-neutral300)',
  '#ecedee': 'var(--base-colors-neutral-neutral200)',
  '#f5f6f6': 'var(--base-colors-neutral-neutral100)',
  '#ffffff': 'var(--base-colors-neutral-neutral000)',
  
  // Red Colors
  '#f63338': 'var(--base-colors-red-red800)',
  '#ff595c': 'var(--base-colors-red-red700)',
  
  // Green Colors
  '#00a22f': 'var(--base-colors-green-green800)',
  '#28c053': 'var(--base-colors-green-green700)',
  
  // Blue Colors
  '#0067ef': 'var(--base-colors-blue-blue800-deep)',
  '#1775f0': 'var(--base-colors-blue-blue800)',
};
```

**이 가이드를 따라 피그마에서 디자인을 가져올 때마다 디자인 토큰을 올바르게 적용하세요.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bcg-dev-team) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
