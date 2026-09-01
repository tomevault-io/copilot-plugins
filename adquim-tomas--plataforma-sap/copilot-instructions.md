## plataforma-sap

> Plataforma de carga masiva de datos hacia SAP Business One (B1) vía Service Layer.

# Adquim — SAP Mass Upload Platform

## Descripción

Plataforma de carga masiva de datos hacia SAP Business One (B1) vía Service Layer.
El usuario sube un Excel; las filas válidas se insertan en SAP y las inválidas se reportan.
**Inserciones parciales** — no es todo-o-nada.

> **Excepción de entrada:** el módulo **Factura de Proveedores** no usa Excel. Las facturas de combustible (ENAP / Esmax) se cargan subiendo sus **XML DTE** (varios a la vez), y la carga **inter-empresa** (Adquim→Adgreen) se ejecuta por **rango de fechas leyendo directo de SAP** (sin subir archivos). En ambos casos los folios ya cargados se omiten (no se duplican).

- **Repo:** PedroPedia  
- **SAP Provider:** H&Co  
- **Deploy:** Azure  

---

## Stack (resumen)

| Capa | Tecnologías |
|------|-------------|
| Frontend | React 19, Vite 8, TypeScript 6, Tailwind 4, shadcn/base-nova + design system *Bitácora* (light canónico, JetBrains Mono Variable como única voz, terminal-data layout), react-router-dom v7, axios — **bun** |
| Backend | FastAPI, SQLAlchemy, Alembic, PostgreSQL, httpx async — **uv** |

> Detalles completos → [`backend/CLAUDE.md`](backend/CLAUDE.md) y [`frontend/CLAUDE.md`](frontend/CLAUDE.md)

---

## Autenticación

1. Validar credenciales del usuario contra SAP B1 Service Layer.
2. Si válidas → emitir **JWT propio** (backend).
3. Operaciones SAP posteriores usan un **service account** dedicado.

---

## Módulos SAP B1

### Socios de Negocios
| # | Módulo | Estado |
|---|--------|--------|
| 1 | Datos Maestros SN | ✅ |
| 2 | Gestión de Clientes | ✅ |
| 3 | Log de Precios | ✅ |

### Compras — Proveedores
| # | Módulo | Estado |
|---|--------|--------|
| 4 | Orden de Compra | ✅ |
| 5 | Factura de Proveedores | ✅ |

### Ventas — Clientes
| # | Módulo | Estado |
|---|--------|--------|
| 6 | Nota de Venta | ✅ |
| 7 | Entrega | ✅ |

### Artículos
| # | Módulo | Estado |
|---|--------|--------|
| 8 | Datos Maestros Artículos | ✅ |

---

## Reglas de Negocio Clave

- **CardCode:** `CN` + RUT para clientes, `PN` + RUT para proveedores.
- **Datos Maestros SN:** solo PATCH (nunca POST).
- **Dirección:** campos opcionales, pero todos requeridos en conjunto si se edita alguno.
- **Vaciar campo en SAP:** el operador escribe `<VACIO>` (case-insensitive) en la celda → el backend lo traduce a `null` y SAP vacía el campo. Una celda vacía, en cambio, omite el campo y SAP no lo toca. Convención compartida frontend/backend en la constante `CLEAR_SENTINEL`.

---

## Modelo de acciones

Cada módulo se particiona en **acciones discretas**: cada acción es un endpoint con su propio allowlist de campos. El operador elige primero qué acción ejecutar (selector dentro de la página del módulo), y solo los campos relevantes a esa acción son aceptados — el backend rechaza columnas fuera del allowlist por Pydantic. No hay endpoint "genérico" en ningún módulo: si una operación no está expuesta como acción, no es ejecutable. Cada acción se registra como `{categoria}/{modulo}/{accion}` en el `HANDLERS` del backend.

**Regla de scope:** una acción se crea solo cuando tiene respaldo concreto en el repo de referencia [`Conexion_Service_Layer_SAP/`](Conexion_Service_Layer_SAP/) (la base de Pedro). Esto evita inventar abstracciones sin fundamento.

**Acciones implementadas hoy:**

| Módulo | Acción | Origen Pedro |
|--------|--------|--------------|
| Datos Maestros | Activar / Desactivar | `classsocio.py::SN.update_SN_activo` |

---

## Audiencia del producto

El producto lo usan **operadores que viven en Excel y conocen SAP** — no desarrolladores. Las páginas de carga (`/uploads/*`) hablan lenguaje de negocio: nada de paths de API, "specification sheets", códigos internos de módulo (`SN.DM`), `PATCH`/`POST`, ni jerga de implementación. Términos SAP que el operador ve en su Excel (`CardCode`, `CardType`, `AddressType`, etc.) **sí** son aceptables. Detalle completo en [`frontend/CLAUDE.md`](frontend/CLAUDE.md).

---

## Estructura del Repo

```
/
├── CLAUDE.md             ← este archivo (contexto general)
├── backend/
│   ├── CLAUDE.md         ← arquitectura, módulos, estado backend
│   └── ...
└── frontend/
    ├── CLAUDE.md         ← diseño, routing, componentes, estado frontend
    └── ...
```

---

## 🤖 Auto-actualización

**Claude Code debe actualizar este archivo cuando:**
- Cambie el nombre del repo, proveedor SAP, o plataforma de deploy.
- Se agregue o elimine un módulo SAP de la lista.
- Cambie el stack general (nueva dependencia mayor, cambio de package manager, etc.).
- Cambie el flujo de autenticación.
- Se modifique una regla de negocio de alcance global.

**Instrucción:** Al finalizar cualquier tarea que implique los cambios anteriores, editar este `CLAUDE.md` actualizando las tablas y secciones afectadas. No reescribir secciones no modificadas.

---
> Source: [adquim-tomas/plataforma-sap](https://github.com/adquim-tomas/plataforma-sap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
