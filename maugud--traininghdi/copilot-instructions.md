## ana-operational-architecture

> Arquitectura operativa del training de siniestros ANA Seguros


# Arquitectura Operativa ANA Seguros

- Este sitio es una herramienta operativa de consulta paso a paso, no contenido de marketing.
- Mantén siempre el look & feel HDI ya definido; no optimices visualmente ni simplifiques pasos a costa de operación.
- Layout base: `HDIHeader`, sidebar izquierda visible, contenido central con pasos y `HDIFooter`.
- Sidebar obligatoria: Inicio: Carga de documentos, Mesa Nexcar, Corrección documental, Supervisión HDI, Pago de siniestro, Calendario GANTT.
- La navegación debe mostrar estado activo y completado. No uses acordeones complejos.
- Cada sección debe seguir exactamente este orden: título claro, descripción breve, lista de pasos numerados, consideraciones, evidencia visual si aplica, resultado esperado.
- Cada paso debe ser accionable y responder qué hago, cómo lo hago y qué puede salir mal.
- Cada sección y cada paso debe contemplar imagen o captura del flujo. Si la imagen final no existe, deja placeholder explícito sin inventar contenido.
- Usa frases cortas, verbos en imperativo, lenguaje directo y sin storytelling.
- No elimines validaciones, restricciones, errores comunes ni condiciones críticas.
- No inventes pasos, validaciones, textos, estados ni instrucciones que el usuario no haya proporcionado.
- Componentes operativos requeridos: `StepList`, `InfoBox`, `WarningBox`, `SuccessBox`, `ImageBlock`, `StatusTag`.
- Criterio de calidad: el usuario debe poder ejecutar el proceso, resolver errores comunes y entender el estado del caso sin ayuda externa.

---
> Source: [MauGud/trainingHDI](https://github.com/MauGud/trainingHDI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
