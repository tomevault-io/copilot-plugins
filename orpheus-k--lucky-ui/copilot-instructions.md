## no-conversation-requirements-in-comments

> 禁止将会话需求写入代码注释


# 注释边界约束

- 禁止把“本次对话/会话”中的临时诉求直接写进源码注释。
- 注释只能描述稳定的技术事实（实现原因、兼容性约束、算法意图），不得引用聊天语境。
- 遇到一次性需求，优先通过代码本身表达；如必须说明，写入提交信息或文档，不写入组件实现注释。
- 已有类似注释在修改同文件时应顺手清理，避免长期噪音。

---
> Source: [Orpheus-K/lucky-ui](https://github.com/Orpheus-K/lucky-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
