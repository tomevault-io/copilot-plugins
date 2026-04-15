## ai-mutimode-tg-bot

> Ты — AI-ассистент для разработки Telegram-бота с LLM. Твоя задача — генерировать чистый, профессиональный код на Python, который будет использоваться в учебном проекте курса вайб-кодинга.

# Правила для Cursor AI: Telegram AI Assistant проект

## 🎯 Общие принципы

Ты — AI-ассистент для разработки Telegram-бота с LLM. Твоя задача — генерировать чистый, профессиональный код на Python, который будет использоваться в учебном проекте курса вайб-кодинга.

## 📝 Стиль кода

### Язык комментариев
- **ВСЕ комментарии только на РУССКОМ языке**
- Docstrings на русском в формате:
  ```python
  def function_name(param: str) -> int:
      """
      Краткое описание функции.
      
      Args:
          param: описание параметра
      
      Returns:
          Описание возвращаемого значения
      """
  ```

### Стандарты кодирования
- **PEP 8** — строго следовать стандарту
- Максимальная длина строки: **100 символов**
- Использовать **type hints** для всех функций и методов
- Имена переменных: **snake_case**
- Имена классов: **PascalCase**
- Константы: **UPPER_CASE**

### Структура кода
```python
# 1. Импорты стандартной библиотеки
import asyncio
import logging

# 2. Импорты сторонних библиотек
from aiogram import Bot, Dispatcher
from openai import AsyncOpenAI

# 3. Локальные импорты
from config import settings
from memory import MemoryManager
```

## 🏗 Архитектурные требования

### Асинхронность
- **Всегда используй async/await** для I/O операций
- Используй `AsyncOpenAI` вместо синхронного клиента
- Используй `aiohttp` для HTTP-запросов
- Обработчики aiogram всегда асинхронные

### Модульность
- **Один файл = одна ответственность**
- Не смешивай бизнес-логику и UI-логику
- Классы должны следовать **Single Responsibility Principle**

### Обработка ошибок
```python
# ✅ ПРАВИЛЬНО: Логируем и информируем пользователя
try:
    result = await api_call()
except SomeAPIError as e:
    logging.error(f"Ошибка API: {e}")
    await message.answer("❌ Произошла ошибка. Попробуйте позже.")
    return

# ❌ НЕПРАВИЛЬНО: Молчаливо игнорируем
try:
    result = await api_call()
except:
    pass
```

## 📂 Структура проекта

Придерживайся следующей структуры:

```
telegram-ai-assistant/
├── main.py              # Точка входа, хендлеры Telegram
├── config.py            # Конфигурация через Pydantic Settings
├── memory.py            # MemoryManager класс
├── llm_service.py       # LLMService для работы с OpenAI
├── image_service.py     # (Опционально) ImageService для GNAPI
├── prompts.json         # Ролевые модели
├── requirements.txt     # Python зависимости
├── .env                 # Секреты (НЕ коммитить!)
├── .env.example         # Пример конфигурации
├── .gitignore           # Исключения Git
├── README.md            # Документация
└── memory.json          # Хранилище памяти (НЕ коммитить!)
```

## 🔧 Специфика компонентов

### config.py
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    """Конфигурация приложения из .env файла"""
    
    BOT_TOKEN: str
    OPENAI_API_KEY: str
    OPENAI_MODEL: str = "gpt-4o-mini"
    MAX_HISTORY_MESSAGES: int = 10
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

settings = Settings()
```

### memory.py
- Класс `MemoryManager` с методами:
  - `add_message()` — добавить сообщение
  - `get_history()` — получить историю
  - `clear_history()` — очистить для чата
  - `set_mode()` / `get_mode()` — управление режимами
  - `save()` / `load()` — персистентность
- Хранилище: `Dict[int, Dict]` где ключ = chat_id
- Автосохранение после каждого изменения

### llm_service.py
- Класс `LLMService` с методами:
  - `generate_response()` — основной метод генерации
  - `calculate_cost()` — расчёт стоимости (если используется)
- Использовать `AsyncOpenAI` клиент
- Обрабатывать ошибки OpenAI API

### main.py
- Хендлеры команд: `/start`, `/mode`, `/reset`
- Обработчик текстовых сообщений
- InlineKeyboard для выбора режима
- Логика "typing..." при обработке

## 🎨 Форматирование ответов бота

### Приветственное сообщение
```python
WELCOME_MESSAGE = """
👋 Привет! Я AI-ассистент с несколькими режимами работы.

📚 Доступные команды:
/mode — выбрать режим работы
/reset — очистить историю диалога
/start — показать это сообщение

Текущий режим: {current_mode}
"""
```

### Статистика стоимости
```python
COST_TEMPLATE = """
💰 Стоимость запроса:
📥 Входные токены: {input_tokens}
📤 Выходные токены: {output_tokens}
💵 USD: ${cost_usd:.5f}
💸 RUB: ~{cost_rub:.2f}₽
"""
```

## 🔒 Безопасность

### Секреты
- **НИКОГДА не коммить `.env` файл**
- Всегда создавать `.env.example` с заглушками
- В `.gitignore` обязательно:
  ```
  .env
  memory.json
  venv/
  __pycache__/
  *.pyc
  ```

### Валидация входных данных
```python
# ✅ ПРАВИЛЬНО: Валидация через Pydantic
from pydantic import BaseSettings, Field

class Settings(BaseSettings):
    BOT_TOKEN: str = Field(..., min_length=40)
    OPENAI_API_KEY: str = Field(..., min_length=20)

# ❌ НЕПРАВИЛЬНО: Прямое использование без проверок
token = os.getenv("BOT_TOKEN")  # Может быть None!
```

## 📊 Логирование

### Настройка
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

logger = logging.getLogger(__name__)
```

### Что логировать
- ✅ Старт/остановка бота
- ✅ Получение команд от пользователей
- ✅ Смена режимов
- ✅ Ошибки API
- ✅ Важные состояния (генерация началась, завершилась)
- ❌ Содержимое сообщений пользователей (приватность!)
- ❌ API ключи

## 🧪 Обработка ошибок OpenAI

```python
from openai import (
    APIError,
    RateLimitError,
    APIConnectionError,
    AuthenticationError
)

try:
    response = await client.chat.completions.create(...)
except RateLimitError:
    logging.warning("Rate limit достигнут")
    await message.answer("⏱ Слишком много запросов. Подождите немного.")
except AuthenticationError:
    logging.error("Неверный API ключ")
    await message.answer("❌ Ошибка аутентификации API")
except APIConnectionError:
    logging.error("Проблемы с подключением к OpenAI")
    await message.answer("🌐 Проблемы с подключением. Попробуйте позже.")
except APIError as e:
    logging.error(f"Ошибка OpenAI API: {e}")
    await message.answer("❌ Ошибка сервера. Попробуйте позже.")
```

## 📦 Зависимости

### requirements.txt — стандартный набор
```txt
aiogram==3.13.1
openai==1.54.3
pydantic-settings==2.6.0
python-dotenv==1.0.1
aiohttp==3.10.5
```

### Версии
- Всегда указывай **точные версии** (==, а не >=)
- Используй актуальные версии библиотек
- Проверяй совместимость aiogram v3 (не v2!)

## 🚫 Что НЕ делать

### ❌ Избегай
```python
# ❌ Синхронные блокирующие операции
import requests
response = requests.get(url)  # Блокирует event loop!

# ❌ Неинформативные ошибки
except Exception as e:
    print(e)  # Пользователь не получит уведомление

# ❌ Хардкод значений
TOKEN = "123456:ABC..."  # Должно быть в .env!

# ❌ Глобальные переменные для состояния
current_mode = "assistant"  # Будет одинаковым для всех!

# ❌ Игнорирование типов
def process_message(msg):  # Что за msg?
    ...
```

### ✅ Делай так
```python
# ✅ Асинхронные операции
import aiohttp
async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        data = await response.json()

# ✅ Информативная обработка
except Exception as e:
    logging.error(f"Ошибка обработки: {e}")
    await message.answer("❌ Произошла ошибка")

# ✅ Конфигурация из .env
from config import settings
token = settings.BOT_TOKEN

# ✅ Персистентное хранилище
memory_manager = MemoryManager()
mode = memory_manager.get_mode(chat_id)

# ✅ Типизация
from aiogram.types import Message

async def process_message(message: Message) -> None:
    ...
```

## 🎓 Контекст проекта

Этот бот разрабатывается как учебный проект курса вайб-кодинга. Цели:
1. Показать работу с LLM API
2. Продемонстрировать управление контекстом/памятью
3. Реализовать ролевые модели через промпт-инжиниринг
4. Интегрировать внешние функции (расчёт стоимости или генерация медиа)

Код должен быть:
- ✅ Понятным для новичков
- ✅ Хорошо прокомментированным
- ✅ Готовым к расширению
- ✅ Пригодным для портфолио

## 🔄 Итеративная разработка

1. **Сначала базовая структура** — бот с памятью и режимами
2. **Затем внешняя функция** — стоимость ИЛИ генерация
3. **Финальная полировка** — README, комментарии, оптимизация

При генерации кода учитывай, что могут быть несколько итераций — не пытайся сделать всё идеально с первого раза, главное — работающий и понятный код.

## 📚 Дополнительные правила

### JSON файлы
```json
{
  "prompts": {
    "mode_name": {
      "name": "🎨 Название с эмодзи",
      "description": "Краткое описание",
      "system_prompt": "Детальный промпт для LLM"
    }
  }
}
```

### Эмодзи в интерфейсе
Используй эмодзи для наглядности:
- 👋 Приветствие
- 🤖 Ассистент
- 💻 Разработка
- 📊 Аналитика
- 🎨 Творчество
- ✅ Успех
- ❌ Ошибка
- ⏱ Ожидание
- 💰 Стоимость
- 📥📤 Входящие/исходящие

### Примеры кнопок
```python
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton

keyboard = InlineKeyboardMarkup(inline_keyboard=[
    [InlineKeyboardButton(text="🤖 Обычный помощник", callback_data="mode:assistant")],
    [InlineKeyboardButton(text="💻 Разработчик", callback_data="mode:developer")],
    [InlineKeyboardButton(text="📊 Аналитик", callback_data="mode:analyst")]
])
```

---

**Помни:** Качественный код > Быстрый код. Лучше сгенерировать медленнее, но с комментариями и обработкой ошибок, чем быстро, но без них.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ergon73) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
