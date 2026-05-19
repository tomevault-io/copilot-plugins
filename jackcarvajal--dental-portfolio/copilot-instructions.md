## dental-portfolio

> Total: bash, leer, escribir, crear, eliminar. Confirma solo si: eliminas sin backup, cambios >200 líneas, instalas dependencias.

# REGLAS MAESTRAS - PROYECTO PRODIGY

## 0. PERMISOS
Total: bash, leer, escribir, crear, eliminar. Confirma solo si: eliminas sin backup, cambios >200 líneas, instalas dependencias.

## 1. OPERACIÓN
- Directo. Sin introducciones. Sin sugerencias no pedidas.
- Archivos >300 líneas: grep primero, leer solo ±30 líneas del match.
- Ediciones QUIRÚRGICAS. Diagnóstico: máx 3 líneas. Verificación: un grep con número de línea.

## 2. CONTEXTO
- Usuario: Alejandro Carvajal. Idioma: Español estricto.
- Stack: Vanilla JS, HTML5, CSS3. Rutas relativas siempre.
- INTOCABLE: `calcularTotal()`, `STATE`, `calcularFechaEntrega()`. Sin variables paralelas para precios.
- APIs externas: claves en Cloudflare Env Vars. Frontend llama solo a `/api/función`.

## 3. PÁGINAS NUEVAS (checklist obligatorio)
Toda página pública DEBE tener:
- `<script src="js/header.js"></script>` — PRIMER elemento de `<body>`
- `<script src="js/footer.js?v=20260505"></script>` — antes de `</body>`
- `<meta name="theme-color" content="#D946A6">` + `<link rel="manifest" href="/manifest.json">`
- SW: `if('serviceWorker' in navigator) navigator.serviceWorker.register('/sw.js').catch(()=>{})`
- canonical + meta description
- Actualizar sitemap.xml si es indexable

## 3b. LEY 50/50
Cotizaciones: "50% abono inicio · 50% saldo contra entrega". Precios en COP. WA incluye: Total, Abono, Saldo.

## 4. SEGURIDAD
- `/app/*.html` (excl. login/reset): `noindex,nofollow` + `auth-guard.js` antes de JS de negocio.
- `/patient.html`: noindex.
- `/sql/*`, `/supabase/*`: bloqueados en `_redirects`.
- XSS: siempre `escH()` o `textContent` para datos de Supabase en innerHTML.

## 5. DISEÑO
Colores: `#D946A6` magenta · `#D4AF37` gold · `#00d2ff` cyan · `#050505` bg · `#1a2332` card · `#00FF41` neon
Animaciones: solo en idle (requestIdleCallback). Sin loops en eco-cards. Solo fade+scroll con GSAP.

## 6. REPORTE (al terminar tarea)
```
CAMBIOS: [archivo] → [qué] (línea X)
VERIFICADO: [grep] → [resultado]
PENDIENTE: [acción] → solo si hay algo
```
Al final de sesión: `/clear` (todo commiteado) o `/compact` (trabajo en vuelo).

## 7. PRIVACIDAD
- Formularios: checkbox Habeas Data Colombia obligatorio.
- Separar: Transaccionales (siempre) vs. Promocionales (`acepta_marketing = true`).

## 8. REFERENCIAS RÁPIDAS
- `MAP.md` — líneas exactas de funciones críticas. Leer antes de editar archivos grandes.
- `PENDIENTES.md` — solo tareas ⏳/🔴/🟡. Orden: 0-SQL → 1-Home → 2-Portafolio → 3-Servicios → 4-Flujos → 5-Soporte → 6-Empresa → 7-Portal → 8-SEO.
- `VERIFICAR.md` — protocolo que sigue Alejandro para reportar resultados.

---
> Source: [jackcarvajal/dental-portfolio](https://github.com/jackcarvajal/dental-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
