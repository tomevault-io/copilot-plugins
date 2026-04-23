## ai-story

> - `backend/` holds Django (`config/settings/*` for env configs, `apps/` per domain, `core/` for shared pipeline and AI clients); supporting notes live in `backend/docs/`.

# Repository Guidelines

## Project Structure & Module Organization
- `backend/` holds Django (`config/settings/*` for env configs, `apps/` per domain, `core/` for shared pipeline and AI clients); supporting notes live in `backend/docs/`.
- `frontend/` is a Vue 2 SPA under `frontend/src/` (components, views, Vuex store) with static assets in `frontend/src/assets/`. Infra artifacts live in `docker/` and `docker-compose.yml`, while generated media belongs in `storage/` outside Git.

## Build, Test, and Development Commands
- Launch everything with `docker-compose up -d`, then run `docker-compose exec backend python manage.py migrate` or `createsuperuser` whenever schema or creds change.
- Backend dev: `cd backend && python manage.py runserver`, plus `celery -A config worker -l info` and `celery -A config beat -l info`; install deps inside a venv from `requirements/development.txt`. Frontend dev: `cd frontend && npm install && npm run dev`, `npm run build` for production, `npm run lint` before commits, and `bash test_sse.sh` for SSE regressions.

## Coding Style & Naming Conventions
Stick to PEP8 with four space indents, auto format via `black .`, and lint with `flake8 .`; keep modules SOLID aligned and name stages after their domain action (`rewrite`, `image_generation`). Vue code follows ESLint defaults, camelCase scripts, kebab case component tags, and the existing atoms or molecules or organisms folder split.

## Testing Guidelines
Use `cd backend && pytest --cov apps --cov core` as the primary suite, storing new `test_*.py` files next to the feature or alongside existing probes such as `backend/test_celery_redis.py`. Reserve `python manage.py test` for smoke checks and rerun `test_sse.sh` after touching Channels or SSE code; include repro notes or screenshots for UI work until automated component tests are added.


## Frontend UI Consistency
- `frontend/src/views/projects/*` 与相关画布组件（如 `frontend/src/components/canvas/*`）的视觉风格必须对齐 `frontend/src/views/prompts/PromptList.vue`。
- 页面级主操作按钮优先复用提示词管理页的胶囊按钮风格：浅色主题下白底细边框，悬浮时出现青蓝色描边、阴影和轻微上浮；深色主题下使用深底浅字版本。
- 列表卡片、分组卡片、阶段卡片、分镜卡片优先复用提示词管理页的卡片语言：顶部 3px 渐变描边、半透明背景、18px 左右圆角、hover 时 `translateY(-4px)` 与阴影增强，并补齐深色主题适配。
- 信息区块优先采用 `card-top / card-meta / card-footer` 的三段式结构，避免随意拼接按钮和说明文字。
- 空状态需与提示词管理页一致，使用标题、说明、副按钮的组合，不要使用过于朴素的占位文案。
- 新增或修改项目管理相关 UI 时，优先保持与提示词管理页一致的动效节奏、边框透明度、阴影层级和颜色语义，除非用户明确要求例外。
- 不要过度使用卡片样式

## Commit & Pull Request Guidelines
Write short imperative commit subjects under roughly 60 characters (module prefix optional) and reference any related ticket or doc. PRs must explain intent, list commands executed, surface schema or env var changes, attach UI screenshots when applicable, and tag a reviewer from the affected layer.

## Security & Configuration Tips
Copy `.env.example` to `.env`, inject keys locally, and avoid committing them; Django reads env vars from that file. Keep generated media in `storage/` or external buckets, prune stale files, rotate provider credentials through the admin UI, and keep logs free of raw secrets.

---
> Source: [xhongc/ai_story](https://github.com/xhongc/ai_story) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
