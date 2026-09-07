## neyra-business-agent-network

> Этот репозиторий хранит переносимую продуктовую основу сети бизнес-агентов Нэйры. Он не является копией клиентского runtime или его данных.

# AGENTS.md

## Назначение
Этот репозиторий хранит переносимую продуктовую основу сети бизнес-агентов Нэйры. Он не является копией клиентского runtime или его данных.

## Владение слоями
| Путь | Владелец |
|---|---|
| `agents/orchestrator/` | маршрутизация и приёмка |
| `agents/legal/` | юридический профиль |
| `agents/finance/` | финансовый профиль |
| `shared/schemas/` | межагентные контракты |
| `demo/` | синтетические примеры |

Нельзя добавлять в Git клиентские знания, память, сессии, логи, `.env`, ключи, OAuth, Telegram-данные, реальные договоры, выписки, IP, домены, бэкапы или production-конфигурации.

Перед коммитом: `make check`. Перед внешним push: пройти `docs/publishing-gate.md`.

---
> Source: [Maksim87de/neyra-business-agent-network](https://github.com/Maksim87de/neyra-business-agent-network) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
