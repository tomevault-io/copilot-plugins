## yente

> - In new test code, write bare asserts (`assert x == y`) and rely on pytest's assertion rewriting for failure output. Don't append the tested variable as a failure message (`assert x == y, x`) — some existing tests do this, but it's not the convention for new code.

## Coding hints

- In new test code, write bare asserts (`assert x == y`) and rely on pytest's assertion rewriting for failure output. Don't append the tested variable as a failure message (`assert x == y, x`) — some existing tests do this, but it's not the convention for new code.

---
> Source: [opensanctions/yente](https://github.com/opensanctions/yente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
