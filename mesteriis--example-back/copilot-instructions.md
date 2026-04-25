## example-back

> always_use_venv: true


python:
  always_use_venv: true
  venv_path: .venv
  run_command: uv run
  install_command: uv add
  sync_command: uv pip sync
  python_path: src

paths:
  relative: true

git:
  servers: ["gitea"]
  workspace_git_paths: ["./"]

docker:
  vm_path: ./docker/data

ai:
  default_repo_server: gitea
  allow_gitea: true

project:
  name: fastapi-app
  language: python
  python_path: src
  description: >
    Fullstack проект на FastAPI + Vue 3 + uv + PostgreSQL + Redis + WebSocket.
    Код разделён по уровням ответственности, без dataclass, с строгим Pydantic v2 и SQLAlchemy 2.0 async.
    Основные принципы: KISS, DRY, чистая архитектура, читаемость, минимализм.

rules:
  # === Общие код-гайдлайны ===
  - name: Code style
    description: Соблюдай чистоту и простоту. Никаких dataclass, только Pydantic v2.
    applies_to: ["*.py"]
    match: true
    directives:
      - enforce_no_dataclass: true
      - prefer_pydantic_v2: true
      - prefer_async_sqlalchemy_v2: true
      - prefer_absolute_imports: true
      - function_per_action: true
      - no_business_logic_in_routes: true
      - use_type_hints_everywhere: true
      - max_function_length: 40
      - no_inline_comments: true
      - no_unused_imports: true

  # === Архитектура ===
  - name: Folder structure
    description: Архитектура должна соответствовать шаблону src/{core,apps}/{module}.
    applies_to: ["src/**"]
    match: true
    directives:
      - enforce_dirs:
          - src/core/config
          - src/core/database
          - src/core/middleware
          - src/apps
      - each_app_has_subdirs:
          - api
          - models
          - schemas
          - repo
          - services
      - disallow_shared_state_between_apps: true

  # === Асинхронность и производительность ===
  - name: Async best practices
    applies_to: ["src/**/*.py"]
    directives:
      - enforce_async_functions: true
      - forbid_blocking_calls: ["time.sleep", "requests", "subprocess"]
      - prefer_httpx_async: true
      - prefer_redis_asyncio: true
      - use_uvloop: true

  # === Конфигурация ===
  - name: Config rules
    applies_to: ["src/core/config/*.py"]
    directives:
      - use_pydantic_settings: true
      - config_files_in_env: true
      - secrets_in_dotenv: true
      - forbid_hardcoded_passwords: true
      - forbid_env_in_code: true

  # === Логирование ===
  - name: Logging and errors
    applies_to: ["src/**/*.py"]
    directives:
      - use_loguru: true
      - log_errors_with_context: true
      - prefer_structured_logging: true
      - no_print_statements: true

  # === Тесты ===
  - name: Tests and validation
    applies_to: ["tests/**/*.py"]
    directives:
      - pytest_style: true
      - prefer_async_tests: true
      - cover_minimum: 80
      - no_db_side_effects: true


  # === Документация (только по запросу) ===
  - name: Documentation
    applies_to: ["src/**/*.py"]
    directives:
      - no_docstrings_by_default: true
      - docstrings_only_on_request: true
      - concise_readme_per_app: true

  # === Оптимизация и сборка ===
  - name: Build & performance
    applies_to: ["Dockerfile", "docker-compose.yml"]
    directives:
      - multi_stage_build: true
      - use_official_python_slim: true
      - prefer_uv_over_pip: true
      - mount_src_readonly: true
      - container_healthcheck: true

  # === Git и CI/CD ===
  - name: Git & CI
    applies_to: [".gitignore", ".woodpecker.yml", ".gitea/workflows/*.yml"]
    directives:
      - no_secrets_in_ci: true
      - prefer_cache_layers: true
      - test_before_build: true
      - lint_on_commit: true
      - format_on_commit: true

ai_assistant:
  behavior:
    - concise
    - never_add_docstrings
    - prefer_type_safety
    - avoid_magic
    - always_refactor_to_functions
    - no_class_god_objects
    - prefer_async/await
    - prefer_fastapi_dependencies_over_global_state
    - respect_architecture

  # === Project Progress Tracking ===
  - name: Progress Tracking
    description: Отслеживание прогресса проекта
    applies_to: [".cursor/project_progress.md"]
    directives:
      - update_progress_on_completion: true
      - move_tasks_from_in_progress_to_complete: true
      - add_new_tasks_to_planned: true
      - use_month_year_date_format: true
      - group_by_project_phases: true
      - track_technical_metrics: true

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mesteriis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
