## agentic-task-manager

> Branch: `dev` (от `f660737`)

# Agentomaton — Progress

## Текущий статус
Branch: `dev` (от `f660737`)

## Выполнено (2026-02-09)

### Communication Style (промпт)
- BRAIN_PROMPT переписан под стиль AK: короткий, технический, без эмоджи-декора
- Запрещены фразы-паразиты, самопредставления, похвала без запроса
- Эмоджи только как статус-маркеры: ✅ ⏳ 🔥 ⚠️
- Примеры хороших/плохих ответов в промпте

### Система напоминаний
- `src/core/reminder.py` — модель (Reminder, ReminderType, ReminderStatus)
- `src/storage/mongo_reminder_store.py` — MongoDB store (CRUD, TTL 30d)
- `src/tools/reminder_tools.py` — 3 tool definitions + парсер времени (ISO, "через X часов", "завтра в 10:00")
- `src/core/scheduler.py` — polling loop (30s), send_fn callback, recurring support
- `src/tools/brain_tools.py` — подключены BRAIN_REMINDER_TOOLS
- `src/core/orchestrator.py` — передает reminder_store, channel_id, creator_id
- `main.py` — init/run/shutdown для reminder_store + scheduler

### Фикс инструментов Brain (2026-02-11)
- `brain_tools.py` — добавлен `list_files` tool (обёртка над `gitea_adapter.list_files()`)
- `brain_tools.py` — убраны: `BRAIN_AGENT_TOOLS` заглушки, `get_my_tasks` (дубль list_tasks), `get_project_goals` (статика)
- `brain.py` — `tool_arguments` schema: string → object (убран double-encoding)
- `brain.py` — early exit: `same_error_repeats` + `last_error_sig`, hard stop после 3 идентичных ошибок
- `brain.py` — iteration budget: `iteration` и `budget_left` в каждом tool result
- `orchestrator.py` — `max_iterations`: 100 → 20
- итого 25 tools (было 29)

### Фикс scheduler + тесты (2026-02-12)
- `brain.py` — `use_llm_debug_explainer` flag (отключает LLM-debug, выводит raw tool_name+args)
- `adapter.py` — status filter: нормализация обеих сторон сравнения через `_normalize_status_name()`
- `orchestrator.py` — scheduler events теперь получают `platform_user_id` (добавлен `EventSource.SCHEDULER`)
- `settings.py` — load_dotenv() до создания Settings, nested BaseSettings теперь подхватывают .env
- `tests/integration/test_readonly_tools.py` — 47 интеграционных тестов (46 pass, 1 skip)
  - Platform: list_statuses, list_users, list_tasks (8 вариантов фильтров), get_task, get_user_workload
  - Gitea: list_repos, list_user_repos, list_files, read_file, list_commits (5 вариантов), list_branches, compare_branches, list_issues (4 вариантов), get_commit_status
  - Web: fetch_page, fetch_invalid_url
  - Unit: normalize_status_name (6 вариантов)

### Prompt refactor + tone fix (2026-02-12)
- `src/core/prompts.py` — **NEW** — все brain prompt templates вынесены из brain.py
- identity: senior engineer → lead developer (убрана резкость)
- communication: разрешены минимальные вежливые реакции ("принял", "рад помочь" в ответ на благодарность)
- proactive: только blockers/overdue/conflicts, запрет комментить WIP
- tagging: только когда нужен action, запрет тегать "для информации"

### Фикс cookie auth (2026-02-13)
- `client.py` — убран `get_platform_user_id()` из `_get_request_cookies()`, всегда `self.user_id` (bot creds)

### Multitenancy (2026-02-13)
- 1 бот = 1 тенант, множество чатов в одном тенанте
- `src/storage/tenant_config.py` — TenantConfigStore (MongoDB: tenant_config collection)
- `src/context/tenant_context.py` — TenantContextSource (goals, chats description, chat name resolution)
- `src/services/chat_session.py` — `get_tenant_session()`: merge history из всех чатов тенанта
- `src/core/prompts.py` — `CHATS_SECTION_TEMPLATE` с описанием доступных чатов
- `src/core/brain.py` — `BrainContext.chats_description`, `source_chat_name`; `[chat_title]` prefix в history
- `src/tools/brain_tools.py` — `send_message_to_chat` tool (cross-chat messaging)
- `src/core/orchestrator.py` — tenant lookup → merged history → tenant goals/context → tools
- `src/api/admin.py` — 6 endpoints: GET/PUT tenant, POST/DELETE channels, PUT goals, PUT settings
- `main.py` — init/shutdown TenantConfigStore, pass to orchestrator + admin API
- 118 тестов pass (15 новых tenant tests)

### Unified Channels: Task 1-2 — TenantChannel extended + migration (2026-02-13)
- `src/storage/tenant_config.py` — TenantChannel: +telegram_username, platform_username, platform_user_id, gitea_username, gitea_user_id, enabled, can_admin
- `scripts/migrate_identities_to_channels.py` — миграция identity store → channels

### Unified Channels: Task 3 — Auto-discovery (2026-02-13)
- `src/core/entities.py` — `EventContext.author_username` field
- `src/adapters/telegram/adapter.py` — передача username во всех normalize-методах
- `src/core/orchestrator.py` — авторегистрация новых чатов как disabled channels + обновление telegram_username для известных каналов

### Unified Channels: Task 4 — Replace identity resolver with channel lookup (2026-02-13)
- `src/core/orchestrator.py` — убраны imports/params identity_store + IdentityResolver
- `src/core/orchestrator.py` — identity resolution заменён на `tenant_config_store.find_channel(author_id)` → platform_user_id из channel
- `src/core/orchestrator.py` — user_mapping_store теперь fallback если channel lookup не нашёл
- `src/core/orchestrator.py` — BrainContext.user_identity=None (будет заменено в Task 5)
- `src/adapters/telegram/listener.py` — убран identity_store, author_identity_id=None
- `main.py` — убран identity_store из listener и orchestrator
- `tests/test_message_listener.py` — обновлены (5 pass, убран тест identity resolution)

### Unified Channels: Task 5 — System prompt with channel profiles (2026-02-13)
- `src/context/tenant_context.py` — `get_chats_description()`: enriched format (private = identity links, group = type), фильтрация по `enabled`
- `src/context/tenant_context.py` — `_chat_id_to_name`, `get_all_chat_ids()`, `get_chat_id_by_name()` — только enabled каналы
- `src/core/brain.py` — удалён `user_identity` из BrainContext, убран import Identity, убран dead code user_identity в `_build_initial_messages`
- `src/core/brain.py` — log line в `think()`: `user_identity.display_name` → `source_chat_name`
- `src/core/orchestrator.py` — убран `user_identity=None` из BrainContext init
- 170 тестов pass (13 новых tenant_context тестов)

### Unified Channels: Task 6 — Secret admin tools (2026-02-13)
- `src/tools/brain_tools.py` — `BRAIN_ADMIN_TOOLS`: `show_system_prompt`, `update_tenant_config` (update_channel_description, update_goals)
- `src/tools/brain_tools.py` — `get_brain_tools()`: +can_admin, +tenant_config_store, +system_prompt_holder; admin tool implementations; tool filtering includes BRAIN_ADMIN_TOOLS when can_admin
- `src/core/brain.py` — `BrainContext.can_admin` field; `Brain.__init__` +system_prompt_holder; `_build_initial_messages` fills holder
- `src/core/orchestrator.py` — can_admin resolve from tenant_config channel; system_prompt_holder shared dict; passed to get_brain_tools + Brain
- 96 тестов pass

### Unified Channels: Task 7 — Admin UI unified tenant tab (2026-02-13)
- `src/api/admin.py` — GET /tenant: возвращает все TenantChannel fields (telegram_username, platform_username, platform_user_id, gitea_username, gitea_user_id, enabled, can_admin)
- `src/api/admin.py` — GET /chats: merge message-based chat discovery с stored channels из tenant_config (stored-only каналы тоже возвращаются)
- `src/webapp/index.html` — убран таб "Пользователи", "Тенант" стал default active tab
- `src/webapp/app.js` — убран весь users-related код (loadUsers, renderUsers, editUser, unlinkPlatform, unlinkGitea, setupSearch); обновлён renderTenant с двумя чекбоксами (Тенант + Админ), identity fields для private чатов; saveTenant отправляет все каналы со всеми полями
- `src/webapp/style.css` — добавлены .chat-checkboxes, .chat-tg стили

### Unified Channels: Task 8 — Remove identity system (2026-02-13)
- Удалены source files: `src/core/identity.py`, `src/ports/identity_store.py`, `src/storage/mongo_identity_store.py`, `src/services/identity_resolver.py`, `src/services/account_linking.py`, `src/services/migration.py`, `src/context/static_context.py`
- Удалены test files: 8 файлов (test_identity*, test_account_linking*, test_user_mapping_migration*, integration/test_*_integration)
- `main.py` — убраны MongoIdentityStore, IdentityResolver; identity_store init/shutdown/pass
- `src/adapters/telegram/bot.py` — убраны /link_platform, /link_gitea, /unlink_platform, /unlink_gitea, /me; identity_store + identity_resolver params; /start упрощён
- `src/api/admin.py` — убраны identity_store, UserResponse, UserUpdateRequest, /users/* endpoints
- `src/core/orchestrator.py` — убран StaticContextSource import и self.context_source; fallback goals=[]
- `src/tools/brain_tools.py` — убраны IContextSource import, context_source param, get_team_info tool definition + impl
- `src/services/__init__.py` — убраны identity-related exports
- `src/context/__init__.py` — заменён StaticContextSource на TenantContextSource
- `tests/integration/conftest.py` — убрана identity_store fixture
- 59 тестов pass (было 118, удалено ~60 identity-related)

### Admin Auth + Unified Channels complete (2026-02-13)
- ADMIN_DEV_MODE заменён на ADMIN_SECRET (Bearer token auth для браузера)
- Admin UI: login screen + tenant tab с identity fields + can_admin checkbox
- 10 коммитов, 105 тестов pass
- Identity system полностью удалена, заменена unified channels

### Brain resilience + chat naming fix (2026-02-16)
- `brain.py` — при пустом ответе LLM (no tool call): nudge message + continue вместо hard fail
- `brain.py` — thoughts отправляются в debug callback (💭 understanding, ➡️ next_step)
- `brain_tools.py` — fix `find_free_slots`: `find_slots()` → `find_free_slots()`, параметры `duration`→`duration_minutes`, `date`→`time_min/time_max`
- `orchestrator.py` — авто-регистрация групп: `chat_title` вместо `author_name`
- `orchestrator.py` — обновление `telegram_username` только для private каналов, сохранение `channel.name`
- `tenant_context.py` — секции "Private chats (DMs)" / "Group chats (visible to everyone)"
- `tenant_context.py` — дедупликация имён каналов (`Artist #1`, `Artist #2`)
- `tenant_context.py` — case-insensitive fallback в `get_chat_id_by_name()`
- `prompts.py` — запрет отправки личных сообщений в групповые чаты
- MongoDB fix: переименованы 3 группы с неправильными именами (author_name → real chat title)

### Prompt management admin panel (2026-02-16)
- `src/storage/tenant_config.py` — `system_prompt_override: Optional[str]` → `prompt_overrides: dict[str, str]`
- `src/core/prompt_registry.py` — **NEW** — PromptRegistry: 6 промптов (brain_prompt, response_mode_smart/always, chats_section, compress_prompt, debug_explainer) с defaults из кода + overrides из tenant_config
- `src/core/brain.py` — принимает `PromptRegistry`, использует `registry.get()` вместо прямых импортов констант
- `src/services/chat_session.py` — `compress_prompt` override через параметр compress_if_needed()
- `src/services/debug_explainer.py` — `system_prompt` override через конструктор
- `src/core/orchestrator.py` — создаёт `PromptRegistry(tenant_config.prompt_overrides)` per-request, передаёт в Brain
- `src/api/admin.py` — 3 endpoint'а: GET/PUT/DELETE `/api/admin/prompts/{key}`
- `src/webapp/` — новый таб "Промпты": карточки с textarea, сохранение/сброс/показ дефолта
- 105 тестов pass

### Bot Memory — self-modifying prompt (2026-02-19)
- `src/storage/tenant_config.py` — `memory: list[str]` field + atomic `add_memory()`/`remove_memory()` ($expr size check + $push/$pull)
- `src/tools/brain_tools.py` — `BRAIN_MEMORY_TOOLS`: `update_memory` tool (add/remove/list), доступен всем (не только admin)
- `src/core/prompts.py` — `{memory_section}` placeholder в BRAIN_PROMPT + инструкция "call update_memory when user asks to remember"
- `src/core/brain.py` — `BrainContext.memory` field, `memory_section` в `_build_initial_messages()`
- `src/core/orchestrator.py` — `tenant_id` + `memory` прокинуты в `get_brain_tools()` и `BrainContext`
- `src/api/admin.py` — GET `/memory`, POST `/memory/delete`
- `src/webapp/` — секция "Память бота" в табе Тенант: список + удаление
- `tests/test_memory.py` — 7 тестов (add, remove, limit, nonexistent tenant)

### TODO
- `docker compose up -d --build agentomaton` — деплой
- Проверить admin UI → таб "Промпты" и "Память бота"

### Что НЕ коммитить
- `demo_langgraph_architecture.py`, `docs/`, `plan.md`, `rag.md` — черновики

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whereismyguts)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/whereismyguts)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
