## video-speed-extension

> Extensión de Chrome (Manifest V3) para controlar la velocidad de reproducción de cualquier video en cualquier sitio web. Sin dependencias externas, JavaScript vanilla puro.

# CLAUDE.md — Video Speed Controller Extension

## Qué es este proyecto
Extensión de Chrome (Manifest V3) para controlar la velocidad de reproducción de cualquier video en cualquier sitio web. Sin dependencias externas, JavaScript vanilla puro.

## Archivos clave

| Archivo | Rol |
|---|---|
| `manifest.json` | Config MV3 — permisos, content scripts, CSP |
| `content.js` | Inyectado en cada página — controla velocidad, atajos de teclado, enforcement |
| `popup.html` | UI del popup — estilos inline, fuentes locales |
| `popup.js` | Lógica del popup — envía mensajes a content.js, persiste estado |
| `fonts/` | Space Mono + Bebas Neue en WOFF2 — sin CDN externo |

## Arquitectura de mensajes

```
popup.js  →  chrome.tabs.sendMessage({ type: 'SET_SPEED', speed })  →  content.js
popup.js  →  chrome.tabs.sendMessage({ type: 'SET_ENABLED', enabled })  →  content.js
```

El estado se persiste en `chrome.storage.local` con las claves:
- `videoSpeed` — número entre 0.1 y 4
- `extensionEnabled` — boolean (default: true)

## Mecanismo de bypass de playbackRate

Reproductores como Viki/Coursera bloquean `video.playbackRate`. La solución:
```js
const descriptor = Object.getOwnPropertyDescriptor(HTMLMediaElement.prototype, 'playbackRate');
descriptor.set.call(video, speed); // setter nativo del prototipo, no el sobreescrito
```

Además hay un listener de `ratechange` por video y un `setInterval(hookAllVideos, 500)` como fallback agresivo. Ambos respetan el flag `enabled`.

## Reglas de desarrollo

- **No agregar dependencias externas** — La CSP del manifest bloquea recursos externos. Todo debe ser local.
- **No usar fetch/XHR** — Sin peticiones de red por diseño y seguridad.
- **Validar siempre los valores de velocidad** — Rango permitido: 0.1 a 4. Nunca aplicar valores no validados.
- **Respetar el flag `enabled`** — Cualquier función que modifique `playbackRate` debe verificar `if (!enabled) return`.
- **No romper iframes** — `all_frames: true` en manifest y `sendMessage` sin frameId específico.
- **La UI es española** — Labels, status y mensajes en español.

## Paleta de colores (CSS variables)

```
--bg: #0a0a0a       (fondo principal)
--surface: #111111  (superficies/botones)
--border: #222222   (bordes)
--accent: #e8ff00   (amarillo neón — activo, ON)
--accent2: #ff4d00  (naranja — peligro, slow)
--text: #f0f0f0     (texto principal)
--muted: #555       (texto secundario)
```

## Cómo probar cambios

1. Guardar los archivos
2. Ir a `chrome://extensions`
3. Click en el ícono de recarga de la extensión
4. Refrescar la pestaña con el video
5. El ZIP se regenera con PowerShell:
   ```powershell
   Compress-Archive -Path popup.js,popup.html,content.js,manifest.json,icon16.png,icon48.png,icon128.png,fonts -DestinationPath video-speed-extension.zip -Force
   ```

## Limitaciones conocidas

- Firefox no compatible (usa MV2 con APIs distintas)
- DRM muy agresivo puede vencer el setInterval de 500ms
- Atajos D/S/R pueden colisionar con atajos de algunos sitios (no pasa en inputs/textareas)
- No hay forma de instalar la extensión como .exe — Chrome bloquea instalación silenciosa por seguridad

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bacoazul)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Bacoazul)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
