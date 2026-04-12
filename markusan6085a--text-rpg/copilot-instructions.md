## text-rpg

> Джерела істини героя/валюти, Merge Policy heroLoadAPI, Register flow, запис l2_accounts_v2, правило мінімальних патчів


# Герой, валюта, merge, storage — не ламати

## 1) Мапінг валюти coinOfLuck (hero) ↔ coinLuck (API)

**Де робиться мапінг:**

| Напрямок | Файл | Функція / місце |
|----------|------|------------------|
| API → Hero | `src/state/heroStore/heroLoadAPI.ts` | При зборі `fixedHero` з `character`: `coinOfLuck: character.coinLuck` (рядки 236, 272). Після GET: `updateServerState({ coinLuck: char?.coinLuck ?? hydratedHero.coinOfLuck })` (рядок 538). Блок «local premium новіший» (488–501): перезапис `hydratedHero.coinOfLuck` з локального. |
| Hero → API | `src/state/heroStore/heroPersistence.ts` | `buildBackupHeroJson`: `coinOfLuck: hero.coinOfLuck`. Перед PUT: `localCoinLuck = hero.coinOfLuck`, `sendCoinLuck` тільки якщо `localCoinLuck >= serverCoinLuck`; `(updatePayload as any).coinLuck = localCoinLuck` (396–410). Після PUT: `serverCoinLuck = (updatedCharacter as any).coinLuck`, передається в `applyServerSync` (432–435). |
| localStorage → Hero | `src/state/heroStore/heroLoad.ts` | Заповнення з heroJson: `(fixedHero as any).coinOfLuck = Number((heroJson as any).coinOfLuck ?? 0)` (рядок 91). |

**Канонічне поле в Hero:**  
`hero.coinOfLuck` (тип у `src/types/Hero.ts`: `coinOfLuck?: number`).

**Що очікує сервер у PATCH/PUT `/characters/:id`:**  
Тіло запиту: поле **`coinLuck`** (number). Код: `server/src/characters.ts` — `body.coinLuck`, валідація «тільки збільшувати» (рядки 706–719, 838).

**Розбіжність (не баг):**  
Різні назви на клієнті (`coinOfLuck`) і на API (`coinLuck`) — це навмисно; мапінг виконується у heroLoadAPI (при читанні) та heroPersistence (при записі). Плутати поля (наприклад слати `coinOfLuck` у body або читати `character.coinOfLuck`) — баг.

---

## 2) Merge Policy Table (heroLoadAPI)

**Змінювати правила merge заборонено без явного «OK» від користувача.**

| Поле | Правило | Причина |
|------|---------|--------|
| exp | **max** (local vs server, як критерій «local preferred»; при server path — з character/heroJson) | Накопичувальний прогрес, не втрачати. |
| level | **max** (так само; при server path — finalLevel = max(heroJson.level, character.level)) | Похідний від exp, не відкатувати. |
| sp | **max** (критерій local preferred; при server path — character.sp) | Накопичувальний. |
| adena | **max** (критерій local preferred; при server path — character.adena) | Валюта, не втрачати. |
| aa | **serverWins** (у merge не входить у localHasMoreProgress; fixedHero з character.aa) | Серверний лічильник; зменшення заборонене на API. |
| coinOfLuck / coinLuck | **serverWins** за замовчуванням; **localWins** лише якщо `localPremiumUntil > serverPremiumUntil` (тоді також перезаписуємо coinOfLuck з локального) | Зменшення тільки через POST /premium/buy; після покупки преміуму не відкатувати. |
| inventory | **localWins** якщо `localInvLen >= serverInvLen`, інакше **serverWins**; окремо при «local preferred» не змінюється (лишаємо локального героя) | Не губити покупки/дроп після F5. |
| equipment | **mergeById** (об'єднання слотів): `{ ...serverEquip, ...localEquip }` — локаль має пріоритет на конфлікт | Плащ/пояс/тату не губити. |
| skills | **mergeById** (по skill id), рівень = **max** (локаль vs сервер) | Доп. скіли та підвищені рівні не губити. |
| buffs | **dedupe** по ключу `id_stackType_name`, залишати баф з **більшим expiresAt**; джерела: heroJson.heroBuffs + loadBattle().heroBuffs; потім cleanupBuffs | Бафи статуї/бою не пропадати після GET. |
| mobsKilled | **max** (критерій local preferred; при server path: `localMobsKilled > serverMobsKilled ? local : server`) | Накопичувальний лічильник. |
| premiumUntil | **max** (локаль vs сервер); якщо локаль новіший — також перезаписуємо coinOfLuck з локального | Після покупки преміуму не відкатувати. |
| hp, mp, cp | **replace** з логікою: hpFull/mpFull/cpFull → finalMax; інакше fill при збільшенні max, інакше clamp поточного значення в [0, finalMax] | Не зменшувати HP/MP/CP при F5 при зростанні max (бафи/екіп). |

---

## 3) Register flow — після реєстрації та F5

**По коду (актуально):**

1. **`Register.tsx`** після успіху: `setCharacterId`, `syncCurrentUserAndAccountHero(trimmedUsername)` перед завантаженням, `loadHeroFromAPI()` → `setHero` + `syncCurrentUserAndAccountHero(trimmedUsername, hero)` (або fallback).
2. **Після F5:** `initializeCharacter()` читає `current_character_id`; `loadHero()` знаходить `l2_current_user` і героя в `l2_accounts_v2` — локальний герой піднімається миттєво; у фоні `loadHeroFromAPI()` може оновити з сервера за merge-правилами.
3. Якщо **немає** ні токена, ні `characterId`, ні локального запису — сценарій як у Landing (логін через API).

**Висновок:** Після реєстрації локальний бекап у `l2_accounts_v2` має бути; використовувати **`syncCurrentUserAndAccountHero(username, hero?)`** у `heroPersistence.ts`.

---

## 4) Записи в l2_accounts_v2 — дозволені місця

**Виклики `setJSON("l2_accounts_v2", ...)`:**

| Файл | Контекст |
|------|----------|
| `src/state/heroStore/heroPersistence.ts` | Збереження героя (save / fallback / backup) **і** публічний helper для `l2_current_user` + рядка в `l2_accounts_v2` при логіні/реєстрації. |
| `src/state/heroStore/heroLoad.ts` | Міграції при читанні (fixProfession, Angel Slayer тощо). |
| `src/utils/fixProfession.ts` | `fixAllHeroProfessions()` — опційно, наразі може не викликатись. |

**Bootstrap (логін / після auth на `/`):** `App.tsx` і **`Register.tsx`** викликають лише **`syncCurrentUserAndAccountHero`**.

Правило: **не додавати нові прямі `setJSON("l2_accounts_v2", ...)`** поза `heroPersistence`, `heroLoad` (міграції) та `fixProfession`.

---

## 5) Правило роботи далі

- Усі правки — **тільки мінімальний патч**. Жодних рефакторів, перейменувань полів/файлів, «покращень архітектури» без явного запиту.
- **Перед патчем:** коротко: план → список файлів → ризики.
- **Після патчу:** чек-ліст «що перевірити» і перелік змінених файлів.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markusan6085A) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
