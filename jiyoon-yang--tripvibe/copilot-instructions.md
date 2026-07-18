## 02-wireframe

> 아래의 조건 및 주의사항을 모두 적용하여 코드를 작성할 것.

아래의 조건 및 주의사항을 모두 적용하여 코드를 작성할 것.

==============================================

* [공통경로]
    - [글로벌CSS]: src/app/globals.css

==============================================

1. CSS조건
    - css는 cssModule만 사용할 것.
    - css에서 예약어(키워드) ":global"는 사용하지 말 것.
    - css에서 예약어(키워드) ":root"는 사용하지 말 것.
    - css에서 예약어(키워드) "important"는 사용하지 말 것.
    - globals.css는 * [공통경로]에 제공된 [글로벌CSS]를 사용할 것.
    - globals.css는 개별 독립적인 파일을 위해서는 변경하지 말 것.
    - globals.css는 변경 전, 다시 한 번 전역을 위해 필요한 작업인지 확인하고, 그러한 경우에만 변경할 것.
    - 추후 수정이 쉽도록, 부모-자식 관계를 형성하여 only flexbox 방식으로 구현할 것. (position-absolute 금지)
    - 추가적인 애니메이션 등은 넣지 말고, 있는 그대로만 완벽히 구현할 것.

---
> Source: [Jiyoon-Yang/tripvibe](https://github.com/Jiyoon-Yang/tripvibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
