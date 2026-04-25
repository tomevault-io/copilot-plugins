## tetrisd

> Правила для операций на VPS (SSH, Docker, бэкапы, диагностика)


# Правила для операций на сервере (VPS)

- **Документация по серверу**: `docs/SERVER_OPERATIONS.md` (деплой, бэкап, восстановление, диагностика диска).
- **Основная директория сервиса на VPS**: `~/apps/tetris-server` (там лежит `docker-compose.yml`), реальные `.env` **не коммитить** и не выводить в ответах.

## SSH-доступ для агентов

- **Хост**: `83.217.221.35`
- **Пользователь**: `gameadmin`
- **Аутентификация**: SSH-ключ `C:\Users\BeQuiet\.ssh\id_ed25519` (использовать только для операций, явно связанных с проектом).
- **Команды по умолчанию**:
  - Диагностика и чтение состояния: `df`, `du`, `top/htop`, `journalctl`, `docker ps`, `docker system df -v`, `docker logs`.
  - Работа с проектом в `~/apps/tetris-server`: `docker compose up -d`, `docker compose down`, `docker compose restart`, просмотр логов сервисов, проверка бэкапов/cron-скриптов.

## Диагностика диска

- При вопросах «чем занят диск»:
  - Сначала проверять Docker: `docker system df -v`.
  - Затем `du -sh /var/lib/docker ~/apps ~/backups ~/*.tgz /var/log 2>/dev/null`.
  - Подробные команды и интерпретацию см. в разделе «5) Диагностика диска» в `SERVER_OPERATIONS.md`.

## Разрешённые операции (без дополнительного подтверждения)

- Запуск диагностических команд и чтение логов (без изменения системных конфигов).
- Безопасная очистка места:
  - Docker: `docker system prune -af` и/или `docker builder prune -a -f` (не трогают используемые контейнеры/volumes).
  - Удаление старых архивов и бэкапов ТОЛЬКО в `~/backups` и `~/*.tgz`, если это явно описано в `SERVER_OPERATIONS.md`.
- Перезапуск сервисов проекта через `docker compose` из `~/apps/tetris-server`.

## Запрещённые операции (требуют явного разрешения пользователя)

- Любые команды с `sudo`, влияющие на систему целиком (`/etc`, пользователи, firewall, systemd-юниты и т.п.).
- Модификация или удаление данных Postgres вне сценариев бэкапа/восстановления, описанных в `SERVER_OPERATIONS.md`.
- Изменение SSH-конфигурации сервера, управляемых ключей и параметров входа.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DenisElspb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
