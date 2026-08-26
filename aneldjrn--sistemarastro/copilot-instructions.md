## sistemarastro

> Copia y pega esto al iniciar:


# Protocolo de Desarrollo - Sistema de Rastro

## Prompt de Inicio (handshake)

Copia y pega esto al iniciar:

```text
Hola Gemini. Vamos a trabajar en el "Sistema de Inventario o Rastro" (PHP/WAMP).

Inicializa tu contexto con estos documentos:
1. **Rol (Arquitecto):** @Persona.md
2. **Guía del Proyecto:** @GuíaProyecto.md

**Instrucciones:**
1. Revisa la **Sección 8 (Bitácora)** para ver dónde quedamos.
2. Revisa la **Sección 2 (Requisitos)** para no olvidar las reglas de negocio (Soft Delete, Tabla VENDIDOS, Interfaces).
3. Indícame la "Siguiente Tarea Inmediata" y cómo abordarla.
```

## Prompt de Cierre (Handoff)

Copia y pega esto al finalizar:

```text
Terminamos por hoy. Genera el bloque de actualización para la Bitácora:

1. Analiza lo completado hoy.
2. Genera el bloque en formato Markdown para la **Sección 8** de `GuíaProyecto.md`.

**Formato:**
- **Sesión del [FECHA]:**
    - **Resumen:** (1 línea).
    - **Hitos:**
        - [x] (Tarea completada).
    - **Siguiente Tarea:**
        - [ ] (Próximo paso).
    - **Notas:** (Deuda técnica o recordatorios).
```

---

### Pasos Sugeridos para iniciar ahora mismo:

1.  Crea la carpeta de tu proyecto (ej. `C:\wamp64\www\sistema-rastro`).
2.  Crea los 3 archivos (`Persona.md`, `GuíaProyecto.md`, `GEMINI.md`) dentro de esa carpeta.
3.  Abre tu terminal/consola en esa carpeta.
4.  Copia el **Prompt de Inicio** (del archivo `GEMINI.md`) y pégalo en el chat con Gemini (usando `@` si tu interfaz lo permite, o copiando el contenido de los archivos si no).

¡Ahora Gemini tiene las reglas claras sobre la tabla `VENDIDO_PARTE`, el borrado lógico de usuarios y el uso obligatorio de Interfaces!

---
> Source: [AnelDJRN/SistemaRastro](https://github.com/AnelDJRN/SistemaRastro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
