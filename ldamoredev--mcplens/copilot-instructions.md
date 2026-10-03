## mcplens

> 1. Leer este archivo completo.

# Guía de agentes — MCPLens

## Inicio obligatorio

1. Leer este archivo completo.
2. Leer `STATUS.md` y confirmar la única fase activa.
3. Inspeccionar el working tree antes de editar.
4. Trabajar solo sobre la fase activa y marcarla `IN_PROGRESS` si todavía figura `NOT_STARTED`.
5. No iniciar la fase siguiente al completar la actual.

`STATUS.md` es la fuente de verdad para el roadmap, las decisiones y el handoff.

## Contratos de producto no negociables

- MCPLens analiza manifests aportados por el usuario. No escanea URLs ni infraestructura de terceros.
- No persistir ni registrar el contenido de manifests.
- No afirmar que un servidor es seguro. Informar hallazgos y límites de evaluación.
- No describir técnicas de explotación; explicar únicamente el riesgo y la corrección.
- Todo hallazgo incluye una explicación llana en `plainLanguage`.
- Sin evidencia textual exacta no hay hallazgo.
- El modelo detecta semántica, pero nunca asigna severidad ni score.
- TypeScript aplica una rúbrica determinística, pura, versionada y testeada.
- El motor de reglas y la rúbrica no importan Anthropic, Next.js ni React.
- Validar toda entrada externa y evitar `any`.

## Arquitectura prevista

- `src/app`: App Router, rutas y composición de la interfaz.
- `src/components`: componentes visuales.
- `src/domain`: tipos y lógica pura.
- `src/lib`: adaptadores e infraestructura.
- `src/fixtures`: manifests y resultados precalculados.
- `docs`: contratos, rúbrica y decisiones.

Crear directorios únicamente cuando la fase activa los necesite.

## Calidad

Antes de completar una fase ejecutar:

```bash
pnpm typecheck
pnpm lint
pnpm test
pnpm build
```

Además, probar manualmente el flujo afectado. No marcar `COMPLETED` con errores de TypeScript, lint o tests, documentación faltante, un flujo principal roto o un blocker que invalide el objetivo.

## Cierre y handoff

Actualizar `STATUS.md` con:

- estado final de la fase;
- archivos cambiados;
- comandos y resultados de validación;
- decisiones tomadas;
- deuda o riesgos conocidos;
- instrucción concreta para el siguiente agente.

Detenerse después del handoff.

---
> Source: [ldamoredev/mcplens](https://github.com/ldamoredev/mcplens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
