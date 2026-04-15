## webrtc-demo

> Всё интерактивное общение с разработчиком ведётся **на русском языке**.

# GEMINI.md — инструкции для Gemini (Google DeepMind)

## Язык общения
Всё интерактивное общение с разработчиком ведётся **на русском языке**.  
Комментарии в генерируемом коде — тоже на русском.

## Описание проекта
`webrtc-demo` — учебное демо P2P видеозвонка через браузер.

- Сигнальный сервер написан на **Go** (Gin + gorilla/websocket).
- Клиент — единственная HTML-страница на **Vanilla JavaScript** без фреймворков.
- Связь: **WebSocket** для сигнализации, **WebRTC** для медиапотока.

## Структура репозитория
```
webrtc-demo/
├── cmd/
│   └── webrtc-demo/
│       └── main.go      # Сервер: Hub, WebSocket-хендлер, маршруты
├── gui/
│   └── call.html        # Клиент: UI + весь JS-код
├── go.mod               # Go-модуль и зависимости
├── README.md            # Документация на русском
├── .github/
│   └── copilot-instructions.md
├── AGENTS.md
├── CLAUDE.md
└── GEMINI.md            # Этот файл
```

## Архитектура сигнализации
Сервер реализует простой broadcast-хаб:
1. Клиент подключается по `/ws`.
2. Любое сообщение от клиента рассылается всем **остальным** клиентам.
3. Сервер не интерпретирует содержимое — клиенты сами разбирают JSON.

Типы сообщений:
| `type` | Содержимое | Назначение |
|--------|-----------|-----------|
| `offer` | `offer: RTCSessionDescription` | Инициация звонка |
| `answer` | `answer: RTCSessionDescription` | Принятие звонка |
| `ice` | `candidate: RTCIceCandidate` | ICE-кандидат |
| `hangup` | — | Завершение звонка |

## Правила работы с кодом

### Обязательно
- Сохранять потокобезопасность `Hub` (`sync.Mutex` на всех операциях с `conns`).
- Использовать `r.StaticFile("/", ...)` вместо `r.Static("/", ...)` — иначе конфликт с `/ws`.
- Проверять наличие `remoteDescription` перед `addIceCandidate`.

### Запрещено
- Добавлять серверную логику WebRTC (сервер — только relay).
- Менять `CheckOrigin` на проверку за пределами dev-окружения без явного задания.
- Делать необратимые git-операции без подтверждения разработчика.

## Команды разработки
```bash
# Запуск
go run ./cmd/webrtc-demo/

# Проверка сборки
go build ./...

# Линтер
go vet ./...
```

Приложение будет доступно на **http://localhost:3000**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vadim-ivlev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vadim-ivlev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
