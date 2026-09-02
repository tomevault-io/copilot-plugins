## fnos-home-assistant

> - 执行任何 `git commit` 前，必须读取 `.github/workflows/validate.yaml` 中

# 项目约束

## 提交前验证

- 执行任何 `git commit` 前，必须读取 `.github/workflows/validate.yaml` 中
  `validate-lint` job 的当前定义，并在本地执行其全部等效步骤。
- 只有 `validate-lint` 的全部等效步骤均成功后，才允许创建提交。
- 任一步骤无法执行或返回失败时，必须先修复问题并重新验证；禁止跳过或绕过验证。

---
> Source: [Timandes/fnos-home-assistant](https://github.com/Timandes/fnos-home-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
