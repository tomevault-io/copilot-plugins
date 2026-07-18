## 04-func

> 아래의 조건 및 주의사항을 모두 적용하여 코드를 작성할 것.

아래의 조건 및 주의사항을 모두 적용하여 코드를 작성할 것.

==============================================

* [상수경로]
    - [ENUM]: src/commons/constants/enum.ts
    - [URL]: src/commons/constants/url.ts

==============================================

1. JS, HOOKS 조건
    - 모든 기능 및 데이터는 해당 파일 안에서 처리하여, 다른 파일에 의존하지 않도록 구현할 것.
    - 의미를 담고 있는 구조화된 타입은 * [상수경로]에 제공된 [ENUM]을 활용할 것.
    - 최소한의 useState, useEffect를 사용할 것.

2. 페이지 링크(이동) 조건
    - 페이지 이동은 직접 하드코딩 하지 않고, * [상수경로]에 제공된 [URL]을 통해서만 이동할 것.

3. 모달 조건
    - 이미 commons에 셋팅된 react-portal을 사용할 것.

4. 폼, 검증 조건
    - 이미 설치된 react-hook-form을 사용하여 폼을 구현할 것.
    - 이미 설치된 zod를 사용하여 검증로직을 구현할 것.

5. API 조건
    - 이미 설치된 @tanstack/react-query를 사용할 것.
    - 이미 commons에 셋팅된 @tanstack/react-query를 사용할 것.

6. TEST 조건
    - TDD기반으로 playwright 테스트를 먼저 작성할 것.
    - playwright.config.ts 설정은 변경하지 말 것.
    - playwright 테스트는 package.json의 scripts에 등록된 명령으로만 테스트 할 것.
    - playwright 테스트에 mock 데이터 사용하지 말고, 실제 데이터를 테스트로 사용할 것.
    - playwright 테스트에 API 테스트가 필요한 경우, 응답 결과를 하드코딩하지 말 것.
    - 테스트에 timeout 방식의 테스트말고, 다른 방식의 테스트가 가능하면, timeout 테스트는 사용하지 말 것.
    - 테스트에 timeout 방식의 테스트가 사용되어야만 하는 경우에는, timeout 은 2000ms 미만으로 설정할 것.
    - 테스트시 사용되는 페이지이동(page.goto)은 baseUrl(호스트와 포트)을 포함하지 않고, 경로만 추가할 것.
    - 테스트시 사용되는 html,css(page.locator)는 cssModule과의 테스트 충돌을 피하기 위해 data-testid를 지정하여 테스트 할 것.

---
> Source: [Jiyoon-Yang/tripvibe](https://github.com/Jiyoon-Yang/tripvibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
