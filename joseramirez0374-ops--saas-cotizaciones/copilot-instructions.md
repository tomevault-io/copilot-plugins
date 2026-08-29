## saas-cotizaciones

> Quotofy es un SaaS multiempresa para administrar clientes, productos, cotizaciones y dos documentos PDF: cotización comercial y documento de entrega sin precios.

# Instrucciones de continuidad para Codex

## Objetivo

Quotofy es un SaaS multiempresa para administrar clientes, productos, cotizaciones y dos documentos PDF: cotización comercial y documento de entrega sin precios.

## Entorno principal

- El usuario desarrolla y prueba en Windows 10/11 con PowerShell.
- Usa Node.js 22.13 o superior, npm y Docker Desktop.
- La web local corre en `http://localhost:3000`.
- La API NestJS corre en `http://localhost:3001/api/v1`.
- PostgreSQL 17 corre con Docker Compose en `localhost:5432`.
- Entregar cambios futuros como ZIP con únicamente los archivos modificados, salvo que el usuario pida un proyecto completo.

## Reglas técnicas que no deben romperse

- No incluir `.env`, contraseñas reales, cookies, tokens, respaldos, archivos subidos, `node_modules`, `dist` ni carpetas temporales en entregables.
- No modificar una migración que ya haya sido aplicada. Crear una migración nueva y ordenada, excepto al corregir una migración que se confirmó que falló y no quedó registrada.
- Toda tabla multiempresa debe incluir `company_id`, claves compuestas cuando corresponda, RLS habilitado y `FORCE ROW LEVEL SECURITY`.
- Las operaciones empresariales deben ejecutarse dentro de `DatabaseService.withTenant`.
- Mantener separados `quotofy_app`, `quotofy_platform` y `quotofy_owner`.
- Las contraseñas se almacenan con Argon2id y las sesiones guardan hash del token, nunca el token original.
- Los PDF se generan exclusivamente en el servidor con PDFKit.
- El documento de entrega nunca debe mostrar costo, margen, precio, descuento, impuesto ni total.
- Las cotizaciones emitidas conservan snapshot y revisiones inmutables; no regenerar historia desde datos actuales.
- Mantener ambas plantillas: `CLASSIC` (Ondas) y `MODERN` (Ejecutiva), con orientación independiente para comercial y entrega.
- Conservar compatibilidad con `npm.cmd` y PowerShell; no depender de Bash para el arranque o la configuración local de la base de datos.

## Validación obligatoria antes de entregar cambios

```powershell
npm.cmd run lint
npm.cmd run typecheck:api
npm.cmd run test:unit
npm.cmd run build
```

Cuando el cambio afecte PostgreSQL, probar además:

```powershell
npm.cmd run db:setup
npm.cmd run db:check
```

## Estado que debe asumirse

- Las migraciones `001` a `007` fueron ejecutadas con éxito en Windows el 15 de agosto de 2026.
- Login, configuración, Agenda y Productos ya están conectados a la API/PostgreSQL.
- La siguiente prioridad es conectar cotizaciones, ambos PDF e historial para completar la rebanada vertical persistente.
- Antes de implementar una fase nueva, leer `CODEX_HANDOFF.md` y `PROJECT_STATUS.md`.

---
> Source: [joseramirez0374-ops/SaaS_Cotizaciones](https://github.com/joseramirez0374-ops/SaaS_Cotizaciones) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
