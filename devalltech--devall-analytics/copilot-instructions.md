## devall-analytics

> SDK oficial de analytics do DevAll Tech para Flutter. Permite monitorar eventos, erros e comportamentos de aplicativos Flutter com facilidade.

# CLAUDE.md - DevAll Analytics SDK

## Projeto

SDK oficial de analytics do DevAll Tech para Flutter. Permite monitorar eventos, erros e comportamentos de aplicativos Flutter com facilidade.

- **Versao:** 0.1.1
- **Linguagem:** Dart 3.5.4+
- **Framework:** Flutter (>=1.17.0)
- **Licenca:** MIT
- **Pub.dev:** https://pub.dev/packages/devall_analytics
- **Repo:** https://github.com/DevAllTech/devall_analytics

## Estrutura

```
lib/
  devall_analytics.dart          # Barrel file (exporta tudo)
  enums.dart                     # DevAllEventType e DevAllEnvironment
  device_identity.dart           # Gerenciamento de deviceId (UUID + SharedPreferences)
  src/
    devall_analytics_core.dart   # Classe principal (init, trackEvent, flush, retry, batch)
    platform_info_stub.dart      # Stub para conditional import
    platform_info_io.dart        # Device info para plataformas nativas (dart:io)
    platform_info_web.dart       # Device info para Web
    offline_storage.dart         # Persistencia offline de eventos (SharedPreferences)
    user_identity.dart           # Identificacao de usuario (userId, traits)
    session_manager.dart         # Gerenciamento de sessoes (sessionId)
    breadcrumbs.dart             # Trail de breadcrumbs para contexto de erros
    error_handler.dart           # Captura global de erros (Flutter + Platform)
    rate_limiter.dart            # Rate limiting de eventos por minuto
    middleware.dart               # Middleware/interceptors para eventos (onBeforeSend)
    lifecycle_observer.dart      # Observer de lifecycle do app (resume, pause, etc.)
    consent_manager.dart         # Gerenciamento de consentimento GDPR
    debug_overlay.dart           # Widget de debug overlay + log em tempo real
    screen_tracker.dart          # Rastreamento de telas e duracao
    event_destination.dart       # Interface para multi-destination (Sentry, Firebase, etc.)
    compression_stub.dart        # Stub para compressao gzip
    compression_io.dart          # Compressao gzip para plataformas nativas
    compression_web.dart         # Stub de compressao para Web
test/
  devall_analytics_test.dart     # Testes unitarios com mock HTTP
example/
  main.dart                      # Exemplo de uso completo
```

## Comandos

```bash
flutter pub get          # Instalar dependencias
flutter analyze          # Rodar linter
flutter test             # Rodar testes
flutter pub publish      # Publicar no pub.dev
```

## Dependencias Principais

- `http: ^1.4.0` - Cliente HTTP para envio de eventos (+ `http/testing.dart` para mock)
- `shared_preferences: ^2.5.3` - Persistencia local do deviceId e consent
- `uuid: ^4.5.1` - Geracao de UUID para identificacao do dispositivo e sessoes
- `flutter_lints: ^4.0.0` - Regras de lint

## Arquitetura

- **DevAllAnalytics** (`lib/src/devall_analytics_core.dart`) - Classe estatica principal com:
  - `init()` - Inicializa com token, URL, client, config de batch, sampling, rate limit, breadcrumbs, compression
  - `trackEvent()` - Envia ou enfileira eventos (com consent, sampling, rate limit, middleware)
  - `flush()` - Forca envio de eventos na fila (batch mode)
  - `reset()` - Reseta estado (para testes)
  - `retryOfflineEvents()` - Reenvia eventos salvos offline
  - `offlinePendingCount` - Numero de eventos na fila offline
  - `clearOfflineEvents()` - Limpa fila offline
  - `identify()` / `clearIdentity()` - Associa/remove usuario logado
  - `startSession()` / `endSession()` - Gerenciamento de sessoes
  - `addBreadcrumb()` / `clearBreadcrumbs()` - Trail de breadcrumbs
  - `captureFlutterErrors()` / `stopCapturingErrors()` - Captura global de erros
  - `enableLifecycleTracking()` / `disableLifecycleTracking()` - Lifecycle automatico
  - `trackScreen()` / `endScreen()` - Rastreamento de telas com duracao
  - `addMiddleware()` / `removeMiddleware()` - Interceptors de eventos
  - `setConsent()` / `isConsentGranted()` - Consent GDPR
  - `addDestination()` / `removeDestination()` - Multi-destination forwarding
  - Retry automatico com backoff exponencial (ate 3 tentativas em 5xx)
  - Fila offline persistente com retry periodico (SharedPreferences)
  - Sampling rate configuravel (0.0-1.0)
  - Rate limiting por minuto
  - Compressao gzip opcional
  - Trunca message em 500 caracteres automaticamente
  - Envia via POST para `{baseUrl}/events` (single) ou `{baseUrl}/events/batch` (batch)
- **DevAllUserIdentity** (`lib/src/user_identity.dart`) - Identificacao de usuario (userId + traits)
- **DevAllSessionManager** (`lib/src/session_manager.dart`) - Sessoes com UUID e timestamps
- **DevAllBreadcrumbs** (`lib/src/breadcrumbs.dart`) - Buffer circular de breadcrumbs com limite configuravel
- **DevAllErrorHandler** (`lib/src/error_handler.dart`) - Captura FlutterError.onError e PlatformDispatcher.onError
- **DevAllRateLimiter** (`lib/src/rate_limiter.dart`) - Limita eventos por janela de 1 minuto
- **DevAllMiddlewareManager** (`lib/src/middleware.dart`) - Chain de middleware para modificar/bloquear eventos
- **DevAllLifecycleObserver** (`lib/src/lifecycle_observer.dart`) - WidgetsBindingObserver para lifecycle do app
- **DevAllConsentManager** (`lib/src/consent_manager.dart`) - Consentimento GDPR persistido via SharedPreferences
- **DevAllDebugOverlay** / **DevAllDebugLog** (`lib/src/debug_overlay.dart`) - Widget overlay + log em tempo real (so em debug)
- **DevAllScreenTracker** (`lib/src/screen_tracker.dart`) - Rastreia telas e calcula duracao automaticamente
- **DevAllEventDestination** / **DevAllDestinationManager** (`lib/src/event_destination.dart`) - Interface + manager para forwarding multi-servico
- **DevAllOfflineStorage** (`lib/src/offline_storage.dart`) - Persistencia local de eventos que falharam no envio
- **DevAllDeviceIdentity** (`lib/device_identity.dart`) - Gera e persiste UUID do dispositivo via SharedPreferences
- **Platform Info** (`lib/src/platform_info_*.dart`) - Conditional imports para compatibilidade Web/IO
- **Compression** (`lib/src/compression_*.dart`) - Conditional imports para compressao gzip (IO/Web)
- **Enums** (`lib/enums.dart`) - `DevAllEventType` (error, warning, info, log, metric, custom) e `DevAllEnvironment` (dev, staging, prod)

## Convencoes

- Codigo em Dart com null-safety
- Linting via `flutter_lints`
- Prefixo `DevAll` em todas as classes publicas
- Logs de debug via `kDebugMode` (nao polui release)
- Commits seguem conventional commits com emojis (ex: :sparkles:, :recycle:, :tada:)
- Testes usam `MockClient` do package `http/testing.dart`
- Codigo interno fica em `lib/src/` (nao exportado diretamente)

## API Endpoint

- **POST** `{baseUrl}/events` (default: `https://api-logs.devalltech.com.br/api/v1/events`) - evento unico
- **POST** `{baseUrl}/events/batch` - envio em lote (2+ eventos)
- Header: `x-project-token` com o token do projeto
- Header: `Content-Encoding: gzip` (quando compressao habilitada)
- Body (single): JSON com deviceId, timestamp, type, environment, category, message (max 500 chars), payload, deviceInfo, ip, userId, userTraits, sessionId, sessionStart, screen, breadcrumbs
- Body (batch): `{"events": [...]}`

## Plataformas Suportadas

Android, iOS, Web, macOS, Windows, Linux

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DevAllTech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
