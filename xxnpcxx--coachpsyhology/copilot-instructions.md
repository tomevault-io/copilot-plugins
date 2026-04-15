## coachpsyhology

> **Telegram боты - это НЕ веб-приложения!** Не применяй стандартные веб-практики безопасности.


# 🤖 Правила Cursor AI - Telegram Bot Development

## 🚨 КРИТИЧЕСКИ ВАЖНО
**Telegram боты - это НЕ веб-приложения!** Не применяй стандартные веб-практики безопасности.

## ✅ Telegram сам обеспечивает:
- Аутентификацию пользователей
- CSRF защиту
- XSS защиту
- Rate limiting
- Управление сессиями

## 🚫 НЕ используй для Telegram ботов:
- `SESSION_SECRET` - не нужен
- `JWT токены` - Telegram ID уже идентификатор
- `CSRF токены` - нет веб-форм
- `Пароли пользователей` - Telegram управляет аутентификацией
- `Веб-сессии` - Telegram управляет сессиями

## 🔧 Обязательные правила:

### 1. Webhook
```javascript
// ✅ Правильно
await bot.setWebhook(`${process.env.WEBHOOK_URL}/webhook`);

// ❌ Неправильно
bot.setWebhookInfo(); // НЕ СУЩЕСТВУЕТ
```

### 2. Клавиатуры - КРИТИЧЕСКИ ВАЖНО!

#### ⚠️ ЧАСТАЯ ОШИБКА: Двойная вложенность reply_markup
```javascript
// ❌ НЕПРАВИЛЬНО - Функции Markup создают объект с reply_markup:
const keyboard = Markup.keyboard([['Кнопка']]);
// keyboard = { reply_markup: { keyboard: [...] } }

await ctx.reply('Текст', { reply_markup: keyboard });
// Результат: { reply_markup: { reply_markup: { keyboard: [...] } } } ← ДВОЙНАЯ ВЛОЖЕННОСТЬ!
// КНОПКИ НЕ ОТОБРАЖАЮТСЯ!

// ✅ ПРАВИЛЬНО - Способ 1: Извлекай .reply_markup
await ctx.reply('Текст', { reply_markup: Markup.keyboard([['Кнопка']]).reply_markup });

// ✅ ПРАВИЛЬНО - Способ 2: Прямое создание объектов (рекомендуется)
await ctx.reply('Текст', {
  reply_markup: {
    keyboard: [['Кнопка']],
    resize_keyboard: true
  }
});

// ✅ ПРАВИЛЬНО - Способ 3: Прямое использование Markup
await ctx.reply('Текст', Markup.keyboard([['Кнопка']]));
```

#### 🚫 Общие правила:
```javascript
// ❌ НЕ смешивай inline и reply клавиатуры в одном сообщении
// ✅ Отправляй отдельными сообщениями
await ctx.reply('Действия:', { reply_markup: inlineKeyboard });
await ctx.reply('Меню:', { reply_markup: replyKeyboard });
```

### 3. Безопасность
```javascript
// Проверка администратора
const isAdmin = msg.from.id.toString() === process.env.ADMIN_USER_ID;

// Параметризованные SQL запросы
await db.query('SELECT * FROM users WHERE telegram_id = $1', [msg.from.id]);

// Обработка ошибок
try {
    await processRequest(msg);
} catch (error) {
    logger.error('Ошибка:', error);
    await bot.sendMessage(chatId, '❌ Произошла ошибка');
}
```

### 4. Переменные окружения
```env
TELEGRAM_BOT_TOKEN=your_token
ADMIN_USER_ID=your_telegram_id
DATABASE_URL=postgresql://...
WEBHOOK_URL=https://your-domain.com
```

## 🔥 КРИТИЧЕСКИ ВАЖНО: Порядок middleware и FSM

### ⚠️ Правило №1: Порядок middleware имеет решающее значение!
```javascript
// ✅ ПРАВИЛЬНЫЙ порядок в bot.js:
const bot = new Telegraf(BOT_TOKEN);
const stage = new Scenes.Stage([...scenes]);

// 1. Session middleware ПЕРВЫМ
bot.use(session({ defaultSession: () => ({}) }));

// 2. Глобальный middleware ДО stage.middleware()
bot.use(async (ctx, next) => {
  // Логирование, проверки, НО НЕ обработка команд!
  if (ctx.scene && ctx.scene.current) {
    return await next(); // Передаём управление сценам
  }
  // Только специальная логика, НЕ команды
  await next();
});

// 3. Stage middleware для FSM сцен
bot.use(stage.middleware());

// 4. Глобальные обработчики ПОСЛЕ stage.middleware()
registerGlobalReplyHandlers(bot);
registerMenuHandlers(bot, stage);
```

### 🚨 ОПАСНО: Неправильный порядок блокирует FSM!
```javascript
// ❌ НЕПРАВИЛЬНО - блокирует сцены:
bot.use(session());
bot.use(stage.middleware());
bot.use(globalMiddleware); // ← middleware ПОСЛЕ stage блокирует сцены!
```

### ⚠️ Правило №2: НЕ дублируй команды и обработчики!
```javascript
// ❌ ЗАПРЕЩЕНО - дублирование команд:
// В commands/start.js:
bot.start(handleStart);
bot.command('start', handleStart);
// В bot.js middleware:
if (text.startsWith('/start')) { handleStart(); } // ← ДУБЛИРОВАНИЕ!

// ✅ ПРАВИЛЬНО - команды только в одном месте:
// В commands/start.js:
export function registerStartCommand(bot, handleStart) {
  bot.start(handleStart);
  bot.command('start', handleStart); // Достаточно одного места
}

// ❌ ЗАПРЕЩЕНО - дублирование bot.hears:
bot.hears("🏠 Главное меню", ...);  // в bot.js
bot.hears("🏠 Главное меню", ...);  // в другом месте - БЛОКИРУЕТ FSM!

// ✅ ПРАВИЛЬНО - только один обработчик:
// В globalReplyHandlers.js:
bot.hears(["🏠 Главное меню", "Главное меню"], async (ctx) => {
  if (ctx.scene && ctx.scene.current) {
    await ctx.scene.leave();
  }
  await sendMainMenu(ctx);
});
```

### ⚠️ Правило №3: Синтаксис для Telegraf 4.x
```javascript
// ✅ ПРАВИЛЬНО для сцен:
scene.on("message:text", async (ctx) => {
  // обработка текста в сцене
});

// ✅ ПРАВИЛЬНО для глобальных:
bot.on("text", async (ctx) => {
  // глобальная обработка текста
});
```

## 🚨 Частые ошибки:
1. **Middleware ПОСЛЕ stage.middleware()** - КРИТИЧНО! Блокирует все сцены
2. **Дублирование команд (/start, /help)** - конфликты и непредсказуемое поведение  
3. **Дублирование bot.hears** - блокирует FSM сцены
4. **Глобальный middleware без await next()** - обрывает цепочку
5. **Смешивание клавиатур** - отправляй отдельно
6. **Игнорирование ошибок** - всегда используй try-catch
7. **Хардкод токенов** - используй переменные окружения

## 📋 Чек-лист:
- [ ] Webhook настроен в Telegram API  
- [ ] **session() middleware ПЕРВЫЙ**
- [ ] **Глобальный middleware ДО stage.middleware()**
- [ ] **stage.middleware() ПОСЛЕ глобального middleware**
- [ ] **НЕТ middleware ПОСЛЕ stage.middleware()**
- [ ] НЕТ дублированных команд (/start, /help)
- [ ] НЕТ дублированных bot.hears обработчиков
- [ ] Все middleware вызывают await next()
- [ ] Клавиатуры отправляются отдельно
- [ ] Все ошибки обрабатываются
- [ ] SQL запросы параметризованы
- [ ] Токены в переменных окружения

## 🛡️ ОТЛАДКА КЛАВИАТУР

### Как проверить клавиатуры:
```javascript
// ✅ Тестирование простых клавиатур:
bot.command('test_keyboards', async (ctx) => {
  // Тест 1: Простая inline кнопка
  await ctx.reply('Тест inline:', {
    reply_markup: {
      inline_keyboard: [[
        { text: '✅ Тест', callback_data: 'test' }
      ]]
    }
  });
  
  // Тест 2: Простая reply клавиатура
  await ctx.reply('Тест reply:', {
    reply_markup: {
      keyboard: [['✅ Тест']],
      resize_keyboard: true
    }
  });
});

// ✅ Логирование структуры клавиатур:
const keyboard = getMyKeyboard();
console.log('Клавиатура:', JSON.stringify(keyboard, null, 2));

// Проверь что НЕТ двойной вложенности reply_markup!
```

### Проверочные вопросы:
1. Есть ли в объекте `{ reply_markup: { reply_markup: {...} } }`? → ОШИБКА!
2. Используешь функции `Markup.*()` в `ctx.reply({ reply_markup: ... })`? → Добавь `.reply_markup`
3. Кнопки не отображаются? → Проверь структуру JSON в логах

---

**Помни: FSM сцены блокируются дублированными обработчиками!**
**Помни: Кнопки не отображаются из-за двойной вложенности reply_markup!**

- В чек-листе к PR добавить пункт: [ ] Все обработчики событий зарегистрированы в точке входа (bot.js).
- Не допускайте регистрации нескольких обработчиков /start в разных модулях — это приведёт к конфликту и неработающему боту!
- Проверяйте, что /start всегда работает после любых изменений FSM, middleware и сцен.
- **ВСЕГДА тестируйте клавиатуры простыми объектами перед использованием функций Markup!**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xxNpCxx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/xxNpCxx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
