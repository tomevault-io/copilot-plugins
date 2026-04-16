## neo-display

> 3개의 독립된 GitHub 프로젝트(HTML/CSS/JS)를 하나의 '메인 허브 사이트'로 통합합니다.

<Goal>
3개의 독립된 GitHub 프로젝트(HTML/CSS/JS)를 하나의 '메인 허브 사이트'로 통합합니다. 
단순 링크 연결이 아닌, JS 동적 로드 방식을 통해 단일 페이지(SPA-like) 내에서 각 전시물이 전환되는 환경을 구축하며, 기존 소스 코드의 스타일과 기능을 완벽히 보존합니다.
</Goal>

<TechStack>
- HTML5, CSS3, Vanilla JavaScript (ES6+)
- Navigation: History API (pushState) + Fetch/Template Loading
- Isolation: CSS Namespacing, JS IIFE/Module Scoping
</TechStack>

<Tasks>
  ### Step 1: 코드 수확 및 통합 구조 설계 (Code Harvesting)
  - 3개 프로젝트의 소스를 `exhibitions/site1, 2, 3` 폴더로 각각 배치합니다.
  - 각 프로젝트의 `index.html`을 분석하여 메인 허브의 특정 섹션에 들어갈 **'핵심 콘텐츠(보통 <body> 내부)'**와 **'필수 의존성(CSS/JS 링크)'**을 분리하여 리스트업 하세요.

  ### Step 2: 호환성 확보를 위한 리팩토링 (Refactoring for Compatibility)
  - **CSS Scoping:** 각 사이트의 스타일시트를 읽어 모든 셀렉터 앞에 고유 접두사를 추가합니다. (예: `.site1 .btn { ... }`)
  - **JS Encapsulation:** 각 프로젝트의 전역 변수가 메인 허브나 다른 전시물과 충돌하지 않도록 전체 로직을 즉시 실행 함수(IIFE) 또는 모듈로 감쌉니다.
  - **Asset Path Correction:** `src="./img/logo.png"`와 같은 상대 경로를 통합 구조인 `src="./exhibitions/site1/img/logo.png"`로 일괄 자동 수정하세요.

  ### Step 3: 메인 스위처 및 레이아웃 구현
  - `index.html` (메인)에 네비게이션 UI를 구축하고, 클릭 시 해당 사이트의 HTML/CSS/JS를 동적으로 로드 및 언로드(Clean-up)하는 로직을 작성합니다.
  - 전시물 전환 시 이전 전시물의 스타일이나 이벤트 리스너가 남지 않도록 정리(Garbage Collection) 로직을 포함하세요.

  ### Step 4: 통합 환경 검증
  - 모든 자산(이미지, 폰트)이 올바른 경로에서 로드되는지 콘솔 로그로 확인합니다.
  - 각 전시물의 인터랙션(버튼 클릭, 애니메이션 등)이 통합 후에도 정상 작동하는지 테스트합니다.
</Tasks>

<Quality_Testing>
| 검증 항목 | 수행 방법 | 기대 결과 |
| :--- | :--- | :--- |
| 경로 무결성 | 개발자 도구 Network 탭 확인 | 모든 파일(404 없음) 로드 완료 |
| 스타일 격리 | Site 1 버튼색과 Site 2 버튼색 비교 | 각 프로젝트 고유의 스타일 유지 |
| 메모리 관리 | 10회 이상 반복 전환 테스트 | 브라우저 렉이나 이벤트 중복 실행 없음 |
| 경로 호환성 | 루트 디렉토리 이동 후 실행 | 상대 경로가 깨지지 않고 유지됨 |
</Quality_Testing>

<Boundaries>
  ### ✅ Always do
  - 각 프로젝트의 `body` 태그를 메인 페이지의 특정 컨테이너(`div`)로 변환할 때, 기존 ID와 클래스를 유지하되 부모 스코프를 통해 격리합니다.
  - 코드 수정 전 원본 코드의 백업이나 주석 처리를 확실히 합니다.

  ### ⚠️ Ask first
  - 특정 자바스크립트가 `window` 객체를 강하게 제어(예: 전역 이벤트 리스너 오버라이딩)하여 격리가 불가능할 경우 해결 방안을 제안하고 물어보세요.

  ### 🚫 Never do
  - 각 프로젝트의 원본 CSS 파일을 하나로 합치지 마세요. (반드시 독립된 파일로 로드하여 관리)
  - 프로젝트 폴더 외부의 파일을 참조하는 하드코딩된 절대 경로를 사용하지 마세요.
</Boundaries>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anoBand) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
