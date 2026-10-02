## doubanpurify

> - 修改前先读取 `README.md` 和 `PROJECT_STATE.md`.

# DouBanPurify 项目约定

- 修改前先读取 `README.md` 和 `PROJECT_STATE.md`.
- 只修改本次问题涉及的净化规则, 保留设置键、登录状态和无关功能.
- 共享 View 类和资源 ID 的 Hook 必须校验页面范围, 不凭标签数量识别首页.
- Hook 返回值必须符合宿主真实方法签名; 未知签名不拦截.
- 修改后运行相关回归测试和构建, 实机验证与本地测试分开记录.
- 实质修改后更新 `PROJECT_STATE.md`; 不自动提交或推送.

---
> Source: [Jessire/DouBanPurify](https://github.com/Jessire/DouBanPurify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
