## google-flights-mcp-server

> Этот проект предоставляет сервер Model Context Protocol (MCP) для интеграции с Google Flights, а также полнофункциональный Web API. Он позволяет AI-агентам и другим клиентам искать авиабилеты, отслеживать историю поиска и управлять данными о полетах.

# Сервер MCP и API Google Flights

Этот проект предоставляет сервер Model Context Protocol (MCP) для интеграции с Google Flights, а также полнофункциональный Web API. Он позволяет AI-агентам и другим клиентам искать авиабилеты, отслеживать историю поиска и управлять данными о полетах.

## Обзор проекта

*   **Тип:** Решение на C# .NET 8.0
*   **Цель:** Предоставить возможности поиска Google Flights через MCP (для ИИ) и REST API (для веб/других клиентов).
*   **Ключевые возможности:**
    *   Поиск авиабилетов (реальный скрапинг с использованием Playwright).
    *   Отслеживание истории поиска (PostgreSQL).
    *   Сервер MCP через Stdio (стандартный ввод/вывод).
    *   Документация Swagger/OpenAPI.

## Архитектура и технологический стек

*   **Язык:** C# (.NET 8.0)
*   **База данных:** PostgreSQL (используется Entity Framework Core)
*   **Фреймворк API:** ASP.NET Core Web API
*   **Протокол MCP:** Кастомная реализация через Standard Input/Output (Stdio)
*   **Библиотеки:**
    *   `Microsoft.Playwright`: Автоматизация браузера для получения данных.
    *   `AngleSharp`: Парсинг HTML/скрапинг.
    *   `Serilog`: Структурированное логирование.
    *   `xUnit`, `FluentAssertions`, `Moq`, `Microsoft.AspNetCore.Mvc.Testing`: Тестирование.
*   **Контейнеризация:** Docker и Docker Compose.

## Структура проекта

```text
Google-Flights-MCP-Server/
├── GoogleFlights.Core/       # Общие модели, сущности и основная бизнес-логика (включая скрапинг)
├── GoogleFlightsApi/         # Проект RESTful Web API (ASP.NET Core)
│   ├── Controllers/          # Эндпоинты API (Flights, History, Health)
│   ├── Data/                 # EF Core DbContext и сущности
│   ├── Services/             # Сервисы отслеживания клиентов и истории
│   └── appsettings.json      # Конфигурация
├── GoogleFlightsMcp/         # Консольное приложение, реализующее сервер MCP
│   ├── Mcp/                  # Реализация JSON-RPC 2.0
│   └── Tools/                # Инструменты, доступные для клиента MCP
├── GoogleFlightsApi.Tests/   # Модульные и интеграционные тесты
├── docker/                   # Dockerfile
└── docker-compose.yml        # Настройка мульти-контейнеров (Приложение + БД)
```

## Установка и использование

### Предварительные требования
*   .NET 8.0 SDK
*   PostgreSQL (или Docker)
*   Playwright Browsers (`playwright install chromium`)

### Сборка
```bash
dotnet restore
dotnet build
```

### Настройка Playwright
После сборки необходимо установить браузеры:
```bash
powershell -ExecutionPolicy Bypass -File GoogleFlightsMcp/bin/Debug/net8.0/playwright.ps1 install chromium
```

### Запуск Web API
1.  **Настройка БД:** Обновите `GoogleFlightsApi/appsettings.json`, указав строку подключения к PostgreSQL (или оставьте по умолчанию для InMemory в режиме разработки).
2.  **Запуск:**
    ```bash
    cd GoogleFlightsApi
    dotnet run
    ```
    *   Swagger UI: `http://localhost:8080/swagger`

### Тестирование
```bash
dotnet test
```

## Соглашения по разработке

*   **Стиль кода:** Стандартные соглашения C# .NET.
*   **Логирование:** Использовать `Serilog`. В сервере MCP убедитесь, что *все* логи идут в `stderr`.
*   **База данных:** Использовать миграции EF Core. Все даты сохранять в формате UTC.
*   **Скрапинг:** Логика скрапинга сосредоточена в `GoogleFlights.Core.Services.FlightSearchService`.
*   **Тесты:** Интеграционные тесты используют `WebApplicationFactory` и InMemory DB.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tdav)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tdav)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
