## python-route-tests

> Python Route Tests

## Python Route Tests


- Polyfactory is the [factory](app/factories/) library in use. `ModelNameFactory.build()` is how you generate factories.
- Use `assert_status(response)` instead of `assert response.status_code == status.HTTP_200_OK`
- Do not reference routes by raw strings. Instead, use the typed route helpers defined in `app/generated/fastapi_typed_routes.py`.

---
> Source: [iloveitaly/llm-ide-rules](https://github.com/iloveitaly/llm-ide-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
