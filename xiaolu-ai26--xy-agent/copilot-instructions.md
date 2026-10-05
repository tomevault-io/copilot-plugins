## xy-agent

> XY-Agent —— 闲鱼电商运营 Agent（自动选品 / 自动上架 / RAG 智能客服）。

# AGENTS.md

XY-Agent —— 闲鱼电商运营 Agent（自动选品 / 自动上架 / RAG 智能客服）。

## 结构
- `src/services/` 核心服务：
  - `product_service.py` 选品 + 上架（分类/计价/价格/大学城定位/违禁词/去重）
  - `message_handler.py` / `message_service.py` / `reply_service.py` 客服（RAG + LLM 兜底，两步回复）
  - `browser_service.py` DrissionPage 浏览器自动化（扫码登录 + Cookie 复用）
  - `knowledge_base.py` ChromaDB + BGE-large-zh-v1.5 本地向量检索
- `src/gui/app.py` customtkinter 桌面控制台（入口 `src/main.py`）
- `build_kb.py` 从 CSV 构建本地知识库

## 运行
```bash
cd src && python main.py
```
扫码登录，Cookie 自动复用。

## 维护注意
- 闲鱼网页会不定期改版导致 RPA 选择器失效；选择器集中在 `product_service.py` / `message_service.py`，优先用**文字/属性定位**而非绝对 XPath。
- 真实密钥、Cookie、话术 CSV、个人 prompt 均已 `.gitignore`，请勿提交。

---
> Source: [xiaolu-ai26/XY-Agent](https://github.com/xiaolu-ai26/XY-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
