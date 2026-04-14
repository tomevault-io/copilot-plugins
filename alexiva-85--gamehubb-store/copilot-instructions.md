## gamehubb-store

> Перед выполнением любой задачи связанной с Altyn, кошельком или платежами — **сначала проверь документацию проекта**.

# Documentation First Rule

## Принцип

Перед выполнением любой задачи связанной с Altyn, кошельком или платежами — **сначала проверь документацию проекта**.

## Обязательный порядок

1. **Проверить документацию** в `docs/`:
   - `docs/altyn-api-reference.md` — API Altyn, комиссии, лимиты, ответы от поддержки
   - `docs/altyn-openapi.json` — полная OpenAPI спецификация Altyn
   - `docs/akuna-wallet-spec.md` — спецификация кошелька Akuna
   - `docs/development-rules.md` — правила разработки

2. **Если ответа нет** — задать вопрос пользователю и дождаться ответа

3. **Только после получения ответа** — приступать к выполнению

4. **Новые подтверждённые ответы** — закреплять в документации

## Ключевые факты (Altyn)

- **Base URL:** `https://api.merchants.altyn.one/gate/`
- **Балансы пользователей:** Altyn НЕ ведёт балансы юзеров — только баланс мерчанта. Балансы ведём в своей БД.
- **external_id:** должен быть уникальным, при повторе — ошибка 400
- **Комиссия конвертации:** 2.5% (включена в курс)
- **Комиссия вывода USDT:** 3 USDT фиксированная
- **Комиссия СБП/карта:** 0%

## Не делай

- Не пиши код пока не проверил документацию
- Не угадывай параметры API — смотри `altyn-openapi.json`
- Не меняй формат авторизации без проверки документации

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexiva-85)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alexiva-85)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
