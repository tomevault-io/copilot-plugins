## propi-solicitada

> **Cada vez que se implemente un cambio relevante en la app, agregar una entrada en la sección "Historial de Cambios" del README.md.**

# Instrucciones para Claude Code — Bóveda Personal

## Regla principal: actualizar README.md con cada cambio

**Cada vez que se implemente un cambio relevante en la app, agregar una entrada en la sección "Historial de Cambios" del README.md.**

### Qué documentar en el historial:
- Nuevas funcionalidades (secciones, filtros, componentes)
- Cambios de comportamiento en funciones existentes
- Correcciones de bugs relevantes (especialmente los que afectan dispositivos)
- Cambios en la estructura de archivos
- Cambios en el Service Worker (versión de caché)
- Cambios en el GAS (nuevas acciones, hojas, lógica)
- Actualizaciones de la guía de ayuda (helpSlides)

### Formato de entrada en el historial:
```markdown
#### YYYY-MM-DD — Título descriptivo
- Descripción de qué se hizo y por qué.
- Si fue un bug fix: describir causa + fix aplicado.
- Si es una feature: describir comportamiento nuevo.
```

---

## Contexto del Proyecto

**App:** PWA para socios del Fondo Solidario de Propina, Casino de Puerto Varas.  
**Repositorio:** `33Javier33/propi.solicitada`  
**Rama principal:** `main` — siempre hacer push aquí.  
**Despliegue:** Vercel automático desde `main`.

### Archivos clave

| Archivo | Propósito |
|---|---|
| `index.html` | Estructura HTML (no incluye CSS ni JS inline) |
| `app.css` | Todos los estilos personalizados |
| `app.js` | Todo el JavaScript de la app |
| `originalindex.html` | Auto-generado por `build.sh` — NO editar manualmente |
| `sw.js` | Service Worker — incrementar versión al publicar cambios |
| `gas/code.gs` | Script GAS de referencia — los cambios al GAS van aquí |
| `README.md` | **Documentación completa — mantener actualizada** |

### Workflow al hacer cambios

1. Editar `index.html`, `app.css` o `app.js` según corresponda
2. El hook de Claude Code ejecuta `./build.sh` automáticamente → regenera `originalindex.html`
3. Si cambian archivos cacheados por el SW, incrementar `CACHE_NAME` en `sw.js`
4. **Actualizar README.md** con el cambio realizado
5. Commit y push a `main`

### Comportamiento del Service Worker

- Versión actual: `boveda-personal-v19`
- Archivos cacheados: `index.html`, `app.css`, `app.js`, `supabase-api.js`, `manifest.json`, iconos
- Estrategia: network-first para archivos propios; GAS siempre por red
- Al cambiar archivos del front → incrementar versión en `sw.js`

### iOS / iPhone — Safe Area

El `viewport-fit=cover` está activo. Las clases `.pt-safe-top` y `.pb-safe-bottom` están definidas en `app.css` y son esenciales para que el header y la nav no queden debajo del notch/Dynamic Island.

### GAS — Scripts Google Apps Script

- **SCRIPT_URL_SOCIOS**: gestión socios, anticipos, conexiones, chat, auditoría
- **SCRIPT_URL_RECAUDACIONES**: recaudaciones diarias, estadísticas, notas
- El archivo `gas/code.gs` es la fuente de referencia; los cambios deben copiarse al GAS real en Google

---
> Source: [33Javier33/propi.solicitada](https://github.com/33Javier33/propi.solicitada) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
