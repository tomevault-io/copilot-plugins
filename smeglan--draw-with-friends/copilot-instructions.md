## draw-with-friends

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

# Estructura recomendada para el MVP del juego de dibujo multiplayer

## Proyecto dividido en:

- frontend Next.js desplegado en Vercel
- signaling server Node.js desplegado en Railway
- WebRTC DataChannels para comunicación realtime P2P
- Socket.IO usado SOLO para signaling y rooms
- no usar base de datos
- no usar Redis
- no usar autenticación
- no usar microservicios
- mantener el proyecto simple y minimalista

## Objetivo MVP:

- conectar dos clientes
- unirlos a una room
- intercambiar signaling
- crear conexión WebRTC
- enviar coordenadas simples de dibujo
- renderizar puntos remotos

**Priorizar simplicidad sobre escalabilidad.**

## Filosofía del proyecto

Este proyecto NO busca ser:

* Photoshop online
* Un framework enterprise disfrazado
* Un sistema ultraescalable desde día 1
* Un SaaS corporativo

El objetivo es:

* Iterar rápido
* Mantener código entendible
* Separar responsabilidades sin sobreingeniería
* Facilitar agregar modos de juego
* Permitir vibecoding sin destruir el proyecto
* Tener una arquitectura suficientemente limpia para crecer sin convertirse en un monstruo

**La prioridad es VELOCIDAD DE ITERACIÓN.**

---

# Stack principal

## Frontend

* Next.js
* React
* TypeScript
* Tailwind

## Render

* Canvas API inicialmente
* PixiJS opcional después

## Realtime

* Socket.io para signaling/eventos
* WebRTC opcional más adelante

## Estado

* Zustand o reducers simples

## Deploy

* Frontend: Vercel
* Realtime server: VPS pequeño

---

# Filosofía arquitectónica

NO usar:

* Arquitectura hexagonal compleja
* CQRS
* Event bus gigante
* Microservicios
* DDD académico completo
* 18 capas de abstracción
* Repositories innecesarios

SÍ usar:

* Módulos claros
* Separación por dominio funcional
* Event-driven simple
* Tipos compartidos
* Código fácil de borrar y reescribir

La arquitectura ideal es:

"DDD simplificado orientado a features"

Cada feature controla:

* lógica
* estado
* eventos
* tipos
* UI relacionada

---

# Estructura sugerida

```txt
src/
  app/
  shared/
  rooms/
  game/
  canvas/
  network/
  replay/
  modes/
  ui/
```

---

# app/

Responsabilidad:

* Rutas Next.js
* Layouts
* Providers globales
* Bootstrap de aplicación

NO meter lógica de negocio aquí.

---

# shared/

Código reutilizable.

```txt
shared/
  types/
  utils/
  constants/
  hooks/
```

Ejemplos:

* tipos compartidos
* generador de room codes
* helpers matemáticos
* utilidades de tiempo

---

# rooms/

Responsabilidad:

* creación de salas
* join/leave
* estado básico de room
* jugadores conectados
* ownership/host

Ejemplo:

```ts
Room
- id
- players
- hostId
- currentGame
- settings
```

NO meter lógica específica de modos aquí.

---

# game/

Responsabilidad:

* ciclo de juego
* turnos
* rondas
* temporizador
* estado global de partida

Ejemplo:

```ts
GameState
- phase
- currentTurn
- timer
- scores
- activeMode
```

Aquí vive el flow principal.

---

# modes/

Cada modo es autocontenido.

```txt
modes/
  telephone/
  guessing/
  turns/
```

Cada modo define:

```ts
interface GameMode {
  startRound(): void
  endRound(): void
  validateInput(): boolean
  calculateScore(): void
}
```

Objetivo:
Poder agregar modos nuevos sin romper todo.

---

# canvas/

Uno de los módulos MÁS importantes.

Responsabilidad:

* render strokes
* herramientas
* brush state
* undo/redo
* snapshots
* sincronización visual

IMPORTANTE:
Trabajar con eventos, NO imágenes completas.

NO:

```ts
canvasImage = "base64 gigante"
```

SÍ:

```ts
StrokeEvent
- tool
- color
- size
- points
- playerId
- timestamp
```

Todo debe poder:

* reproducirse
* sincronizarse
* deshacerse
* grabarse

---

# network/

Responsabilidad:

* sockets
* sincronización
* mensajes
* conexión/desconexión

Separar:

```txt
network/
  client/
  server/
  events/
```

Todos los mensajes deben estar tipados.

Ejemplo:

```ts
ClientToServerEvents
ServerToClientEvents
```

IMPORTANTE:
NO mezclar lógica visual con networking.

---

# replay/

Responsabilidad:

* reproducción de partidas
* timeline
* exportación
* reveal final

Idealmente basado en:

```ts
ReplayEvent[]
```

No grabar frames.
Grabar eventos.

---

# ui/

Responsabilidad:

* componentes reutilizables
* botones
* modales
* overlays
* HUD

Separar:

```txt
ui/
  components/
  layout/
  overlays/
```

---

# Reglas importantes

## 1. Evitar premature optimization

NO optimizar antes de necesitar.

El objetivo inicial:

* diversión
* estabilidad básica
* iteración rápida

---

## 2. El código puede ser reescrito

No enamorarse de la primera arquitectura.

Si una parte molesta:

* se borra
* se simplifica
* se rehace

---

## 3. Priorizar event sourcing ligero

Todo lo importante debería ser eventos.

Ejemplos:

```ts
PLAYER_JOINED
STROKE_ADDED
ROUND_STARTED
TURN_ENDED
MESSAGE_SENT
```

Esto facilita:

* replay
* debugging
* sincronización
* espectador
* exportación

---

## 4. Mantener estado simple

Evitar:

* estados profundamente anidados
* stores gigantes
* reducers monstruosos

Preferir:

* módulos pequeños
* estado local cuando sea posible

---

## 5. Multiplayer tolerante

No hacer lógica hiper competitiva.

El juego debe tolerar:

* lag
* paquetes tardíos
* pequeños desyncs

Es un party game.
La diversión importa más que la precisión perfecta.

---

# MVP real

## Features necesarias

* Crear sala
* Entrar con código
* Canvas compartido
* Brush básico
* Undo simple
* Timer
* Turnos
* Replay final
* 3 modos:

  * dibujo por turnos
  * adivinar
  * teléfono roto

---

# Features NO necesarias para MVP

NO hacer inicialmente:

* cuentas
* perfiles
* matchmaking complejo
* monetización
* cosméticos
* chat de voz
* layers avanzadas
* filtros
* mobile perfecto
* IA generativa
* persistencia permanente
* ranking global

---

# Regla final

La prioridad del proyecto es:

"hacer algo divertido rápido"

No:

"impresionar arquitectos de LinkedIn"

<!-- END:nextjs-agent-rules -->

---
> Source: [smeglan/draw-with-friends](https://github.com/smeglan/draw-with-friends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
