## polish-my-sword

> Juego de Roblox (Luau) llamado **Sword Polish Simulator**. El jugador lustra espadas con minijuegos, pelea en un coliseo, consigue espadas nuevas y progresa por islas. Multijugador con trading, PvP y grupos cooperativos.

# Sword Polish Simulator — Claude Code Instructions

## Qué es este proyecto
Juego de Roblox (Luau) llamado **Sword Polish Simulator**. El jugador lustra espadas con minijuegos, pelea en un coliseo, consigue espadas nuevas y progresa por islas. Multijugador con trading, PvP y grupos cooperativos.

## Stack y herramientas
- **Lenguaje:** Luau (Roblox Studio, `--!strict` en todos los archivos)
- **Sync:** Rojo (`default.project.json` en la raíz) — ejecutar `rojo serve` para sincronizar con Roblox Studio
- **Estructura:** Cliente/Servidor con RemoteEvents. NUNCA lógica de juego en el cliente.
- **Persistencia:** DataStoreService con pcall + 3 reintentos

## Reglas de arquitectura (no negociables)
1. Todo cálculo de recompensas y validación ocurre en el **servidor**, nunca en el cliente
2. El cliente solo envía intenciones (ej: "quiero lustrar"), el servidor valida y responde
3. Un módulo = una responsabilidad (ver `docs/ARCHITECTURE.md` para la lista completa)
4. Antes de crear un archivo nuevo, verificar si ya existe un módulo responsable
5. `Constants.lua` en ReplicatedStorage es la fuente de verdad para todos los números del juego

## Convenciones de código
- Variables locales: `camelCase` | Módulos y funciones públicas: `PascalCase`
- Comentarios de negocio: español | Comentarios técnicos: inglés
- Usar `task.spawn` para operaciones asíncronas, nunca `coroutine` directo
- Cada módulo empieza con bloque de comentario explicando su responsabilidad
- Rate limiting: máximo 1 lustrada cada 3 segundos por jugador (verificar en servidor)

## Cómo verificar tu trabajo
```bash
# Verificar sintaxis de Luau (si tienes selene instalado)
selene src/

# Ver estructura del proyecto
cat default.project.json

# Estado actual de progreso
cat docs/PROGRESS.md
```

## Dónde está todo
- **Spec técnica completa:** `@docs/TECH_SPEC.md` — leer antes de implementar cualquier sistema
- **Progreso actual:** `@docs/PROGRESS.md` — qué está hecho y qué sigue
- **Arquitectura:** `@docs/ARCHITECTURE.md` — mapa de módulos y responsabilidades
- **Datos de espadas:** `src/ReplicatedStorage/SharedModules/SwordData.lua`
- **Constantes:** `src/ReplicatedStorage/SharedModules/Constants.lua`

## Flujo de trabajo por etapa
1. Leer la etapa completa en `docs/TECH_SPEC.md` antes de empezar
2. Implementar los módulos del servidor primero
3. Crear los RemoteEvents necesarios
4. Implementar la parte del cliente
5. Marcar como completado en `docs/PROGRESS.md`
6. No avanzar a la siguiente etapa sin que la actual funcione

## Errores comunes a evitar
- NO usar `wait()` — usar `task.wait()`
- NO usar `spawn()` — usar `task.spawn()`
- NO confiar en el cliente para calcular daño o coins
- NO hacer llamadas a DataStore sin pcall
- NO crear RemoteEvents en el script — crearlos como objetos en la carpeta `RemoteEvents` de ReplicatedStorage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jazielitopro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
