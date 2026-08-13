## backend

> 编辑后端代码/脚本时参考——日志、加解密、运行方式


# Backend（按需参考）

- **技术栈**：Python 3.11+ / FastAPI / SQLAlchemy 2.0 异步 / PostgreSQL + Redis / Pydantic。
- **日志**：`get_logger(__name__)`；关键步骤与异常打日志，异常时记录堆栈；不记敏感数据与主键。参考 `app.core.logging`。
- **加解密**：凭据用 `decrypt_data`/`encrypt_data`（`app.core.security`）；接收加密数据后解密再使用；测试连接时可 try 解密、失败则当明文。
- **ID**：见 api-id-hashid.mdc；`encode_id` 返回、`decode_id` 接收，不返回数字主键。
- **运行 Python/脚本**：先 `cd backend` 并 `source .venv/bin/activate`，再执行；命令形式：`cd backend && source .venv/bin/activate && python ...`。
- 常见 HTTP 错误：404 路由、405 方法、400 参数、401 鉴权、500 看异常与日志。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
