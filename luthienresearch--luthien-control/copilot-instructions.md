## pytest-interpretation

> PYTEST EXIT CODE=0 JUST MEANS THE TESTS RAN. IT DOES NOT MEAN THEY WERE SUCCESSFUL. Always look for the `pytest` summary line at the end of the output (e.g., `==== X passed, Y failed, Z warnings in S.ss ====`).

PYTEST EXIT CODE=0 JUST MEANS THE TESTS RAN. IT DOES NOT MEAN THEY WERE SUCCESSFUL. Always look for the `pytest` summary line at the end of the output (e.g., `==== X passed, Y failed, Z warnings in S.ss ====`).

Always pipe results to cat so you can see them.
`poetry run pytest | cat`

For reasons unknown, you sometimes don't see the output of pytest the first time you run it. If the first run of pytest seemingly produces no output, just run it again. This seems to work.

---
> Source: [LuthienResearch/luthien_control](https://github.com/LuthienResearch/luthien_control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
