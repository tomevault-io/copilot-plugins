## controles-contables

> > Este archivo se lee automáticamente en cada sesión de Claude Code en este repo.

# CLAUDE.md — Instrucciones para Claude Code

> Este archivo se lee automáticamente en cada sesión de Claude Code en este repo.
> Define cómo Claude debe trabajar en este proyecto. Mantener corto y vigente.

---

## 1. Contexto del proyecto

**Nombre:** Controles Contables
**Owner:** Willy (Guille) — Payroll, IT & Implementation Manager en Hidalgo & Asociados (H&A)
**Tipo:** Herramienta interna HTML browser-side para validación de cálculos contables de payroll.
**Audiencia:** Equipo de Payroll de H&A.

**Para qué sirve:** Validar cálculos contables de payroll (provisiones, cargas sociales, conciliaciones) contra lo que asienta el ERP (Meta4 hoy, Axton en la migración), empleado por empleado y mes a mes. Primer control: **Provisión de Vacaciones**.

**Para qué NO sirve:** No reemplaza al ERP, no escribe en él. Es 100% lectura y análisis local.

> **Excepción consciente vs Controles Nómina:** ESTE proyecto sí calcula (la provisión es el insumo del control). En Nómina la regla era "no es herramienta de cálculo". Documentado en `DECISIONS.md`.

---

## 2. Stack técnico

- **Frontend:** HTML + Vanilla JS con módulos ES6 (`import` / `export`). Sin framework.
- **Estilos:** CSS plano, sin preprocesadores. Variables CSS para la paleta H&A.
- **Excel:** SheetJS (`xlsx`) vía CDN — `https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js`
- **IndexedDB:** Dexie.js vía CDN — `https://unpkg.com/dexie@4/dist/dexie.min.js`
- **Build:** Ninguno. Todo se sirve como archivos estáticos. Live Server / static server.
- **Sin transpilación, sin bundler, sin npm install.**

---

## 3. Arquitectura — motor + definiciones enchufables

```
controles-contables/
├── CLAUDE.md, DECISIONS.md, README.md
├── index.html
├── css/ (tokens, base, components)
├── js/
│   ├── main.js          ← router hash-based + bootstrap
│   ├── db.js            ← Dexie schemas + CRUD
│   ├── engine/
│   │   ├── controlRunner.js   ← orquesta: aplica compute() por fila y arma summary
│   │   └── crossTemporal.js   ← indexa la sesión final del mes anterior para el cruce
│   ├── controls/
│   │   └── provisionVacaciones.js  ← definición: inputFields + erpFields + compute()
│   ├── parsers/
│   │   └── reporteVacaciones.js    ← parsea el reporte de Meta4 (input + referencia ERP en un solo Excel)
│   ├── ui/              ← runProvisionVacaciones (preview), toast — wizard formal en bloque 8
│   ├── export/toExcel.js
│   └── utils/           ← currency, dates, validators
```

Una **definición de control** (`controls/*.js`) expone:
- `key`, `name`, `inputFields[]`, `erpFields[]`
- `compute(row, period, prevProvision)` → calcula la fila

El `controlRunner` no sabe de vacaciones: recibe filas ya parseadas + opcionalmente las filas de la sesión final del mes anterior, llama `compute(row, period, prevRow)` por fila y devuelve `{rows, summary}`. Agregar provisión de SAC = un archivo nuevo en `controls/`, cero cambios en el motor.

El cruce calculado-vs-ERP queda **implícito dentro de `compute()`** (devuelve `difDev` y `difProvAnt`) porque el reporte de Meta4 ya trae los valores del ERP en las mismas columnas que el input. Si en el futuro aparece un Excel separado del ERP, se agrega `engine/crossErp.js` y `parsers/erpVacaciones.js` (ver DECISIONS.md).

---

## 4. Convenciones de código

- **Idioma:**
  - Código (variables, funciones, archivos): **inglés**.
  - Comentarios, UI, strings visibles al usuario: **español argentino**.
- **Indentación:** 2 espacios.
- **Strings:** comillas simples por defecto. Template literals con interpolación.
- **Punto y coma:** siempre.
- **Async:** `async/await`.
- **Nombres:** `camelCase` funciones/variables, `PascalCase` clases, `UPPER_SNAKE` constantes globales.
- **No `var`.** Solo `const` y `let`.
- **Imports relativos** dentro del proyecto.
- **Constantes con nombre** para números mágicos (ej: `DIVISOR_DIA_VACACIONES = 25`).

---

## 5. Marca H&A — uso obligatorio

Aplicar siempre el skill **`hya-brand`**.

Reglas mínimas:
- Celeste primario **`#00ACD4`**.
- Gris wordmark **`#8C837B`** para "Hidalgo & Asociados".
- Tipografía **Source Sans Pro** (Google Fonts, fallback Arial).
- Logo H&A en header con fallback CSS si no hay red.
- Footer con datos de contacto corporativos.

**Aviso de privacidad obligatorio** (banner visible antes de cualquier input de archivo):

> ⚠ **Aviso de privacidad:** Esta herramienta procesa los datos 100 % en tu navegador — nada se sube a internet. Aun así, **no compartas información personal identificable de empleados o clientes** fuera de los canales autorizados por H&A. Usá esta herramienta solo en equipos corporativos.

---

## 6. Privacidad y seguridad

1. **Nada sale del navegador.** No hay backend, no hay API calls (salvo CDNs). Datos en IndexedDB local.
2. **No loguear datos sensibles a consola.** En producción, `console.log` con datos de empleados están prohibidos.
3. **Export Excel/JSON incluye datos personales.** Avisar al usuario al exportar.
4. **No telemetría, no analytics, no tracking.**

---

## 7. Git workflow — obligatorio

**Cada cambio termina con: commit → push → PR → merge a main.** Sin pedir confirmación.

Secuencia:
```
git add <archivos>
git commit -m "..."
git checkout -b feat/nombre-descriptivo
git push -u origin feat/nombre-descriptivo
"C:\Program Files\GitHub CLI\gh.exe" pr create --base main --head feat/... --title "..." --body "..."
"C:\Program Files\GitHub CLI\gh.exe" pr merge --merge --delete-branch
```

Notas:
- `gh` no está en el PATH — usar ruta completa.
- Willy es único owner — mergear directo.
- Branch names: descriptivos del **contenido real**, no del primer fix.

---

## 8. Estilo de commits — Conventional Commits

- `feat:` nueva funcionalidad
- `fix:` corrección de bug
- `docs:` documentación
- `refactor:` cambio sin cambiar comportamiento
- `chore:` mantenimiento

Mensajes en español argentino.

---

## 9. Cómo trabajar con Willy

- **Brainstorming antes de código** si el pedido tiene ambigüedad.
- **Datos reales, no suposiciones.** Si Claude no sabe algo (ej: cómo viene exactamente un archivo), preguntar.
- **Idioma:** español argentino, registro directo e informal.
- **Working output rápido** antes de pulir. Iteración visual > planificación exhaustiva.
- **Explicaciones simples y con analogías** — Willy está aprendiendo programación.
- **No sobre-formatear** las respuestas.

---

## 10. Testing

- Test de aceptación crítico de la fórmula (`provisionVacaciones.test.js`): base 10.000.000, días teóricos 14, mes=5 → 2.333.333,33; mes=4 → 1.866.666,67.
- Antes de cada commit: `node --check` sobre `.js` modificados.
- Antes de mergear cambios en `js/parsers/`, `js/engine/`, `js/db.js`: correr subagente `privacy-reviewer`.

---

## 11. Documentos vivos

| Documento | Cuándo actualizar |
|---|---|
| `CLAUDE.md` | Cuando cambian convenciones o stack |
| `DECISIONS.md` | Cuando se toma una decisión técnica no obvia |
| `README.md` | Cuando cambia el flujo de uso |

---

**Última actualización:** 4 de junio de 2026 — bootstrap inicial del proyecto.

---
> Source: [bhidalgo-ar/controles-contables](https://github.com/bhidalgo-ar/controles-contables) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
