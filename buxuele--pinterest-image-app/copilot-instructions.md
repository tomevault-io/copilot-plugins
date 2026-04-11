## pinterest-image-app

> 项目：pinterest_image_app

# AGENTS

项目：pinterest_image_app

## 结构速览
1. 后端：`backend_python_api`
2. 前端：`frontend_react_app`
3. 图片库：`imgs_data/<文件夹名>`
4. 上传目录：`backend_python_api/user_uploads`
5. 删除回收站：`backend_python_api/dustbin`

## 启动
1. 后端：`uvicorn main:app --host 127.0.0.1 --port 8000 --reload`（工作目录：`backend_python_api`）
2. 前端：`npm start`（工作目录：`frontend_react_app`）
3. 一键：Windows 可用 `just_run.bat` 或 `python run_all.py`

## 配置
1. 默认前端通过代理访问 `http://localhost:8000`。
2. 若前后端不在同一域名/端口，创建 `frontend_react_app/.env` 并设置 `REACT_APP_API_BASE_URL`。

## 维护
1. 删除图片会移动到 `backend_python_api/dustbin`，需要定期清理以释放空间。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/buxuele)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/buxuele)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
