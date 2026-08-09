## asistente-de-estudio

> > Lee SPEC.md antes de cada sesión. Este archivo es la gramática.

# [Nombre del Proyecto] — Constitución Arquitectónica

> Lee SPEC.md antes de cada sesión. Este archivo es la gramática.
> SPEC.md es la fuente de verdad. En conflicto, SPEC.md gana.

## Proyecto
Tutor con IA para estudiantes universitarios y opositores/profesionales que convierte sus apuntes y libros en cuestionarios adaptativos con retroalimentación precisa sobre sus vacíos de conocimiento.

## Arquitectura
- **En capas:** Frontend (Next.js) → API (FastAPI) → Módulos de dominio (document_processor, knowledge_engine, quiz_generator, evaluator_engine, reporter) → Infraestructura (PostgreSQL + pgvector)
- **Dirección de dependencias:** dominio → aplicación → infraestructura, nunca invertido. Los módulos de dominio no conocen la web ni la base de datos.
- **RAG-first:** Toda generación de preguntas y evaluación se basa en recuperación de contexto vectorial del material subido. No se usa conocimiento pre-entrenado del modelo para evaluar respuestas.
- **Sin estado en el backend:** Las sesiones de usuario se mantienen en el frontend o en almacenamiento efímero. El backend es stateless para facilitar escalado horizontal.

## Estándares
- **Lenguaje backend:** Python 3.12+, runtime: FastAPI (asíncrono)
- **Lenguaje frontend:** TypeScript, framework: Next.js 14+ (React)
- **Base de datos:** PostgreSQL con pgvector
- **Framework de pruebas backend:** pytest
- **Framework de pruebas frontend:** Vitest + Playwright
- **Cobertura mínima:** 80%
- **Longitud máxima de archivo:** 400 líneas
- **Longitud máxima de función:** 50 líneas
- **Formato de commit:** Conventional Commits (feat/fix/chore/refactor/test/docs)

## Estructura de Módulos
| Módulo | Responsabilidad |
|--------|----------------|
| `document_processor` | Extrae, limpia y fragmenta texto de archivos subidos (PDF/txt) |
| `knowledge_engine` | Administra embeddings, búsqueda vectorial (RAG) y recuperación de contexto |
| `quiz_generator` | Crea preguntas adaptadas al nivel de dificultad seleccionado |
| `evaluator_engine` | Compara respuestas del usuario con la fuente original y redacta retroalimentación |
| `reporter` | Compila resultados de la sesión y genera el diagnóstico final de aprendizaje |
| `web_frontend` | Interfaz de usuario: carga de documentos, visualización de notas, cuestionario y reporte |
| `web_backend` | API REST que orquesta los módulos anteriores |

## La IA Nunca Debe
- Nunca ser complaciente: no dar por correcta una respuesta errónea o incompleta solo porque el usuario responda con tono seguro
- Nunca alucinar datos o citas: si la respuesta a un tema no está en el material ni en las fuentes permitidas, debe declarar explícitamente que no la conoce
- Nunca eliminar registros sin un paso explícito de confirmación en el prompt
- Nunca cambiar la dificultad en medio de una sesión sin notificarlo al usuario
- Nunca correr migraciones de base de datos sin revisión humana
- Nunca saltarse la autenticación por conveniencia (cuando se implemente)
- Nunca realizar búsquedas web sin límites: la búsqueda externa debe restringirse a fuentes académicas o limitarse estrictamente al material subido

## Checklist de Cierre de Sesión
- [ ] Pruebas pasan
- [ ] Sin errores de linting
- [ ] STATUS.md actualizado
- [ ] Commit sigue formato Conventional Commits

---
> Source: [DedSecRisk/Asistente_de_estudio](https://github.com/DedSecRisk/Asistente_de_estudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
