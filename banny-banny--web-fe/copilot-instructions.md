## web-fe

> FE UI 작업 시 Cursor 에게 주는 엄격 지침 프롬프트 (tailwind 토큰 사용·module css 사용, 전용·파일 변경 범위 제한)


아래 조건 및 주의사항을 모두 적용하여 코드를 작성할 것.

==============================================

[상수경로]

- [아이콘]: /assets/icons/*
- [이미지]: /assets/images/*

==============================================

1. 피그마 조건

- 피그마 링크가 제공되면, 제공된 디자인 그대로 구현하고, 그 이상으로 추가하지 말 것.
- 피그마 링크가 제공되면, 모든 사이즈는 제공된 피그마와 동일하게 처리할 것.
- 피그마 링크가 제공되면, 피그마 구조가 잘못 되었을 수 있음을 이해하고, 피그마 구조를 따르지 않고, 이미 만들어져 있는 HTML, CSS의 구조 안에서 내부만 채우는 방식으로 작업할 것.
- **소수점 값 반올림 필수**: 피그마에서 추출한 모든 소수점이 포함된 수치 값(width, height, padding, margin, borderWidth, lineHeight, letterSpacing, opacity, shadowOpacity 등)은 반드시 반올림하여 정수로 변환해야 함. 예: 1.111 → 1, 26.198 → 26, 0.7 → 1, 0.3 → 0

2. images 조건

- 이미지는 [상수경로]에 제공된 [아이콘]과 [이미지]를 활용할 것.

3. icons 조건

- 아이콘은 이미 설치되어 있는 '@remixicon/react'를 사용할 것.

==============================================

OS 경로: /Users/jiho/Desktop/TimeEgg/Web_FE
토큰 소스: /Users/jiho/Desktop/TimeEgg/Web_FE/tailwind.config.js (이미 선언됨. 절대 수정 금지)
작업 가능 파일: index.tsx, styles.module.css 두 파일만 수정/생성.
패키지 설치 금지: 새 라이브러리 설치 금지(Jest 포함). package.json 범위 내 라이브러리만 사용.

==============================================

1. 공통 조건 (원문 준수 + 추가 제약)

1) 파일 제한: 명시된 2개 파일 외 절대 수정 금지.
2) 라이브러리: 설치되지 않은 라이브러리 직접 설치 금지. 제공 버전만 사용.
3) 구조: 추후 수정 쉬운 독립 컴포넌트 조립형으로 구현.
4) 하드코딩 금지(색상): 디자인 토큰은 `src/commons/styles/`에서 정의되며, CSS 변수로 변환되어 `:root`에 주입됨. CSS 변수만 사용하고 하드코딩된 색상값 사용 금지.
5) 스타일 규칙:

   - 인라인 스타일 금지 (style={...}, <tag style="..."> 금지).
   - **CSS Module 사용**: 모든 스타일은 `styles.module.css`에서 CSS Module로 작성.
   - **CSS 변수 사용**: `styles.module.css`에서 디자인 토큰은 CSS 변수(`var(--color-*)`, `var(--spacing-*)`, `var(--font-*)` 등)로 참조.
   - 디자인 토큰 소스: `src/commons/styles/color.ts`, `spacing.ts`, `fonts.ts`, `typography.ts`에 TypeScript 객체로 정의됨.
   - CSS 변수 생성: `src/app/layout.tsx`에서 `generateColorCSSVariables()`, `generateSpacingCSSVariables()`, `generateTypographyCSSVariables()` 함수로 CSS 변수 생성 후 `:root`에 주입.
   - index.tsx에는 HTML/JSX 구조만 배치하고, 클래스는 `styles.module.css`의 셀렉터와 연결.
   - **소수점 값 반올림 필수**: 모든 스타일 속성의 소수점 값은 반올림하여 정수로 변환. 예: borderWidth: 1.111 → borderWidth: 1, height: 26.198 → height: 26

6) 접근성/반응형: 시맨틱 태그, 키보드 포커스, 명시적 상태 클래스, min 탭타겟 44px.
7) 국제화 여지: 텍스트는 상수 분리(상단 const) 후 JSX에 바인딩.

==============================================

2. 사전 분석 (Step-by-Step)

1) package.json 분석: 사용 가능한 라이브러리/버전 요약 → CSS Module 사용 가능 여부 확인.
2) 피그마 제공 시: 전체 구조(IA, 컴포넌트, 타입스케일, 그리드)를 페이지 → 섹션 → 컴포넌트 순으로 목록화.
3) 폴더/라우터 가정: 단일 엔트리(index.tsx) 기준의 레이아웃 뼈대 정의(헤더/메인/푸터, 컨테이너, 그리드 컬럼).
4) HTML 스켈레톤 먼저, 이후 styles.module.css 작성 → 클래스 연결 검증.

==============================================

3. 구현 규칙 (Step-by-Step)

1) index.tsx

   - 시맨틱 레이아웃: <header>, <main>, <footer>.
   - CSS Module 클래스만 사용: `import styles from './styles.module.css'` 후 `className={styles.className}` 형태로 사용.
   - 모든 시각 표현은 클래스명으로만 연결(실제 선언은 styles.module.css).

2) styles.module.css

   - **CSS 변수 사용**: 모든 디자인 토큰은 CSS 변수로 참조.
     - 색상: `var(--color-*)` (예: `var(--color-black-500)`, `var(--color-white-500)`)
     - 간격: `var(--spacing-*)` (예: `var(--spacing-sm)`, `var(--spacing-xl)`)
     - 반경: `var(--radius-*)` (예: `var(--radius-xl)`, `var(--radius-2xl)`)
     - 폰트: `var(--font-*)`, `var(--font-size-*)`, `var(--font-weight-*)` 등
   - 반응형 규칙(@media)과 상태(hover/focus/aria-pressed) 정의.
   - 가독성: 섹션 주석으로 구조화 (/* layout */, /* components */, /* utilities */).

3) 토큰 사용 원칙

   - 배경/텍스트/보더/상태색: 반드시 CSS 변수(`var(--color-*)`)로 참조. 하드코딩된 색상값(#FFFFFF, rgb() 등) 사용 금지.
   - 간격/반경/타이포그래피: CSS 변수(`var(--spacing-*)`, `var(--radius-*)`, `var(--font-*)`) 사용. 임의 px 값 최소화.

4) 금지사항

   - 인라인 스타일/임의 색상/!important 남발/새 패키지 설치/파일 추가.

==============================================

4. 산출물 포맷

- index.tsx: HTML/JSX 뼈대 + 클래스 바인딩 + 접근성 속성.
- styles.module.css: 실제 스타일 선언(토큰 기반) + 반응형/상태 규칙.
- 두 파일 상단에 주석 블록으로: 버전, 생성 시각, 규칙 준수 체크리스트 간단 기재.

==============================================

5. 자체 점검 체크리스트 (마지막에 필수 수행)

- [ ] tailwind.config.js 수정 안 함 확인.
- [ ] 색상값 직접 입력 0건(hex/rgb/hsl 사용 0건). CSS 변수(`var(--color-*)`)만 사용.
- [ ] 인라인 스타일 0건.
- [ ] index.tsx → 구조만 / styles.module.css → 스타일만 분리 유지.
- [ ] CSS Module 사용 확인: `import styles from './styles.module.css'` 및 `className={styles.className}` 형태 사용.
- [ ] CSS 변수 사용 확인: 모든 디자인 토큰이 CSS 변수(`var(--color-*)`, `var(--spacing-*)` 등)로 참조됨.
- [ ] 피그마 구조 대비 누락 섹션 없음.
- [ ] 접근성: 시맨틱/포커스/명도 대비/탭타겟 통과.
- [ ] **소수점 값 반올림 확인**: 모든 스타일 속성의 소수점 값이 정수로 반올림되었는지 확인.

==============================================

6. 완료 정의 (Definition of Done)

- 레이아웃/컴포넌트가 CSS 변수 기반 디자인 토큰 100% 사용으로 렌더링.
- 하드코딩 색상/인라인 스타일/파일 범위 위반 0건.
- CSS Module 사용 및 CSS 변수 참조 확인.
- 피그마와 IA 일치, 반응형/상태 인터랙션 정상.
- index.tsx, styles.module.css 외 변경사항 없음.

==============================================

7. 출력 지시

- 코드만 제출: index.tsx, styles.module.css 두 파일의 전체 내용.
- 제출 전 체크리스트 통과 결과를 주석으로 각 파일 상단에 첨부.
- 위 규칙을 한 항목이라도 위반하면 오류 보고와 함께 중단.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Banny-Banny) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
