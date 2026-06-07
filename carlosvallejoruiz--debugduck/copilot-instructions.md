## debugduck

> Widget de escritorio nativo (macOS/Windows) para "Rubber Duck Debugging" interactivo. Pato flotante transparente con IA local, voz, animaciones y sistema Tamagotchi.

# Contexto del Proyecto: DebugDuck

Widget de escritorio nativo (macOS/Windows) para "Rubber Duck Debugging" interactivo. Pato flotante transparente con IA local, voz, animaciones y sistema Tamagotchi.

## 1. Experiencia de Usuario

- **Doble clic** en el pato → activa micrófono (Web Speech API)
- **Flujo del bocadillo** (estilo cómic, fondo blanco, borde negro):
  1. Escuchando → anillo latiendo
  2. Pensando → puntos suspensivos + animación rasca cabeza
  3. Respuesta → texto scrolleable + botones Eureka/Pomo
- **¡Eureka!** → confeti fullscreen + +1 contador + +10 happiness (Tamagotchi)
- **Pomo** → arranca timer 25min visible encima del pato, notificación al terminar
- **Ajustes** (engranaje hover) → panel con blur

## 2. Stack Tecnológico

- **Frontend:** React + TypeScript + Vite + Tailwind CSS v4
- **Estado:** Zustand con persist
- **Desktop:** Tauri v2 (Rust)
- **IA:** LM Studio — `http://localhost:1234/v1/chat/completions`
- **HTTP:** `@tauri-apps/plugin-http` para bypass CORS en bundle firmado
- **Animaciones:** Canvas offscreen 190×190px, PNGs normalizados a 600×600

## 3. Arquitectura
src/
├── hooks/
│   ├── useVoiceRecognition.ts   # Web Speech API + fixTechTerms
│   ├── useAIResponse.ts         # LM Studio + streaming + memoria
│   ├── useAnimation.ts          # Sistema de animaciones por estado
│   ├── useWindowPosition.ts     # Posicionamiento ventana
│   ├── usePomodoro.ts           # Timer 25min + notificación
│   └── useTamagotchi.ts         # Sistema Tamagotchi (happiness)
├── components/
│   ├── DuckAvatar.tsx           # Canvas 190px + badge Tamagotchi
│   ├── SpeechBubble.tsx         # Bocadillo cómic + confeti
│   └── SettingsPanel.tsx        # Panel de ajustes completo
├── store.ts                     # Zustand store persistido
└── App.tsx                      # Coordinador principal
src-tauri/
├── src/lib.rs                   # Comandos Rust
├── capabilities/default.json    # Permisos Tauri v2
├── entitlements.plist           # Entitlements macOS
└── tauri.conf.json              # Config bundle

## 4. Reglas Críticas de Desarrollo

1. **Bundle siempre** — nunca `tauri dev`, siempre `tauri build --debug` + firmar
2. **Bundle ID:** `com.debugduck.widget`
3. **Tauri v2 API** — `import { getCurrentWindow } from '@tauri-apps/api/window'`
4. **HTTP a localhost** — usar `@tauri-apps/plugin-http`, NO fetch nativo
5. **Streaming** — usar comando Rust `stream_lm_studio` + eventos Tauri
6. **Transparencia** — `body, #root { background: transparent !important }`
7. **WebKit bug** — `* { -webkit-text-stroke: 0 !important }` en index.css
8. **Posición inicial** — calcular en Rust: bottom-right con margen 16px/80px
9. **Click-through** — canvas alpha pixel-perfect via `isDuckTransparent()`
10. **pointer-events: none** en `#root`, `auto` solo en componentes interactivos

## 5. Comandos Rust Implementados

```rust
call_lm_studio(prompt)      // fetch no-stream a LM Studio
stream_lm_studio(...)       // streaming SSE → eventos Tauri
set_ignore_cursor(ignore)   // click-through nativo
get_cursor_pos()            // posición cursor relativa ventana
launch_confetti_window()    // ventana overlay confeti fullscreen
set_fullscreen_confetti()   // DEPRECATED — usar launch_confetti_window
```

## 6. Sistema de Animaciones
src/assets/animations/
├── idle/      # 35 frames, 3s, loop continuo
├── rasca/     # 35 frames, 2.5s — thinking + idle random
├── asiente/   # 35 frames, 2s — listening
└── libreta/   # 35 frames, 4s — listening (pato más pequeño por diseño)

**Estados:**
- `idle` → loop idle, cada 8-15s animación aleatoria
- `listening` → cicla rasca/asiente/libreta con pausa 0.8-1.5s
- `thinking` → loop rasca continuo
- `responding` → loop idle

**Canvas:** 190×190px fijo, escala por altura (`190/naturalHeight`), centrado horizontal

## 7. Panel de Ajustes (SettingsPanel)

- **Personalidad:** toggle Programador 🦆 / General 🌍
- **Modo Tamagotchi:** toggle ON/OFF (oculta slider crueldad cuando ON)
- **Slider crueldad** (0-100): Mentor paciente / Equilibrado / Sin piedad
- **Modelo activo:** detectado automáticamente de LM Studio + botón ↺
- **Forzar modelo:** select manual override
- **Memoria conversación:** toggle ON/OFF + botón limpiar
- **Posición ventana:** grid 3×3 flechas

## 8. Sistema de IA

**Detección automática de modelo:**
```typescript
GET http://localhost:1234/v1/models → data[0].id
isThinkingModel: /qwen|deepseek|r1/i → usa reasoning_content
```

**Modos de personalidad:**
- `programmer` → MODO CONCEPTUAL (didáctico) + MODO DEBUG (socrático)
- `general` → responde cualquier tema con carácter sarcástico

**Slider de crueldad:**
- <30: Mentor paciente, sin sarcasmo, cálido, celebra intentos
- 30-70: Equilibrado, toque sarcástico, asume conocimiento básico
- >70: Sin piedad, técnico seco, sarcasmo inteligente tipo senior

**Memoria de conversación:**
- Historial últimos 4 mensajes (2 pares)
- Compresión automática a los 2 intercambios (resumen 200 tokens)
- NO persiste entre sesiones, solo la preferencia ON/OFF

**Streaming SSE:**
- Modelos normales (Mistral, Llama): `stream: true` via comando Rust
- Modelos thinking (Qwen): `stream: false` + extracción de reasoning_content
- max_tokens: 400 con memoria, 800 sin memoria

## 9. Sistema Tamagotchi (useTamagotchi.ts) — PENDIENTE DE IMPLEMENTAR

**Estados happiness:**
- 😊 Feliz (70-100): tono positivo y entusiasta
- 😐 Neutral (40-69): directo y equilibrado
- 😤 Malhumorado (20-39): irritado, sarcástico, se queja del abandono
- 😡 Furioso (0-19): brutal, muy corto, sin paciencia

**Eventos:**
- Eureka: +10 | Pomodoro: +5 | Pregunta compleja (>8 palabras): +3
- Tiempo sin interacción (>1h): -8/hora | Pregunta corta (<4 palabras): -2

**Badge visual (DuckAvatar):**
- Emoji estado siempre visible (top-right del pato)
- Hover → barra de vida expandida con color dinámico
- Oculto cuando tamagotchiMode=false

**Integración IA:**
- Cuando tamagotchiMode=true → tono controlado por happiness
- Cuando tamagotchiMode=false → tono controlado por slider crueldad

**En store.ts (PERSISTIDO):**
- `tamagotchiMode: boolean` (default: false)
- `duckHappiness: number` (default: 75)
- `lastInteraction: number` (default: Date.now())

## 10. Corrección de Términos Técnicos (useVoiceRecognition.ts)

`fixTechTerms()` corrige transcripciones erróneas del speech recognition:
- "Rack/Rad" → "React"
- "Types Creed" → "TypeScript"  
- "dub ar" → "debugging"
- "pitón" → "Python"
- + ~40 términos técnicos más

## 11. Bugs Conocidos y Soluciones

### TCC / SpeechRecognition
- Requiere bundle `.app` firmado, nunca binario suelto
- `tccutil reset Microphone && tccutil reset SpeechRecognition` si falla

### CORS con LM Studio
- fetch nativo bloqueado en bundle firmado
- Usar `@tauri-apps/plugin-http` o comando Rust con reqwest
- Streaming: comando Rust `stream_lm_studio` + `window.emit()` chunks

### Qwen3.5 thinking mode
- content siempre vacío, respuesta en reasoning_content
- Regex: `/"([^"]*[áéíóúñ¿¡][^"]{10,}[.?!]?)"/g` → último match >100 chars
- max_tokens: 2000 mínimo

### Click-through transparente
- CSS pointer-events NO funciona para otras apps del SO
- Solución: `get_cursor_pos()` Rust + canvas alpha pixel reading cada 30ms
- `set_ignore_cursor(true)` cuando alpha < 10 en el PNG

### Confeti segunda vez
- Usar label único con timestamp: `confetti-{Date.now()}`
- `set_ignore_cursor_events(true)` en ventana confeti para no bloquear

### Animación libreta más pequeña
- Es por diseño — encuadre más abierto al sostener libreta
- No es un bug de normalización (duck ocupa 563×600 del canvas)

## 12. Flujo de Desarrollo Diario

```bash
npm run tauri build -- --debug && \
codesign --sign - --force --deep \
  --entitlements src-tauri/entitlements.plist \
  src-tauri/target/debug/bundle/macos/DebugDuck.app && \
open src-tauri/target/debug/bundle/macos/DebugDuck.app
```

---
> Source: [CarlosVallejoRuiz/DebugDuck](https://github.com/CarlosVallejoRuiz/DebugDuck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
