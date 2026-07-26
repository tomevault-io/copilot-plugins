## ai-novelist

> 3. 重构时不要兼容任何旧的逻辑，除了加速代码腐化以外，没有任何用处

# 这是你必须遵守的开发规范
1. 后端导入，一概使用绝对导入
2. 前端类型一概放在types/
3. 重构时不要兼容任何旧的逻辑，除了加速代码腐化以外，没有任何用处
4. 所有配置读取和修改操作必须通过 [`backend/config.py`](backend/settings/config.py) 中提供的 `settings` 类方法进行。
5. **所有 Git 提交信息（commit message）必须使用中文撰写**，确保用户能直接理解每次更新的具体内容并在本地选择合适的存档点/分支。
其他详见DEVELOPMENT.md

---
> Source: [FlickeringLamp/ai-novelist](https://github.com/FlickeringLamp/ai-novelist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
