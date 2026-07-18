## 05-func-role

> 아래의 조건 및 주의사항을 모두 적용하여 코드를 작성할 것.

아래의 조건 및 주의사항을 모두 적용하여 코드를 작성할 것.

==============================================

1. 권한분기 페이지GUARD TEST 조건
    - "로그인 유저"를 기본값으로 하여, 모든 페이지 로그인 여부에 상관 없이 접속 가능하도록 할 것.

2. 권한분기 액션GUARD TEST 조건
    - 로그인유저 시나리오: "로그인 유저"가 기본값이므로, 모든 기능의 로그인검사가드를 무시할 것.
    - 비로그인유저 시나리오: "로그인 유저"가 기본값이므로, 모든 기능에 로그인검사가드를 활성화할 것.(window.__TEST_BYPASS__ = false)

---
> Source: [Jiyoon-Yang/tripvibe](https://github.com/Jiyoon-Yang/tripvibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
