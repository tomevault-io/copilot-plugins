## code-301-guide

> > Instrucciones para agentes AI que trabajan en este repositorio.

# AGENTS.md - Code 301 - Professional Software Development (`code-301`)

> Instrucciones para agentes AI que trabajan en este repositorio.
> Compatible con: Claude Code, GitHub Copilot, Cursor, Gemini CLI, Windsurf, Codex.

---

## Proyecto

Repositorio del curso **Code 301 - Professional Software Development** de EnterTechSchool (código interno: `code-301`). Contiene el currículo completo del curso de desarrollo web full stack: READMEs, laboratorios, slides, guías de facilitador y rúbricas.

- **Total de clases:** 26 sesiones
- **Estructura:** 5 módulos técnicos (4 clases c/u) + 1 proyecto final (6 clases)
- **Duración por clase:** 150 min síncronos + 2 h asíncronas
- **Modalidad:** Blend (teoría + práctica)
- **Stack:** React + Vite (M1–M3), PostgreSQL + Supabase (M3), Next.js 15 + TypeScript + Prisma (M4–M6), NextAuth.js v5 (M5). **Sin Express.**
- **Gestor de paquetes:** `pnpm` en todo el curso (reemplaza a `npm` por seguridad de cadena de suministro y rendimiento)
- **IA:** Integrada transversalmente en módulos 1-5
- **Prerrequisito:** Code 201 (HTML/CSS, JavaScript, DOM, Git)

---

## Fuente de Verdad

**`README.md` es la única fuente de verdad.** Contiene:

- Estructura del curso (módulos, clases, proyectos integradores)
- Contenido clave por clase y enfoque de IA
- Sistema de evaluación (rúbricas, labs calificados, proyecto final)
- Tecnologías por módulo
- Prerrequisitos heredados de Code 201

**Regla:** Leer SIEMPRE `README.md` antes de generar o modificar contenido. Nunca hardcodear información que ya está en el syllabus. Si hay conflicto entre un archivo de clase y `README.md`, el syllabus gana.

---

## Estructura del Repositorio

```
├── README.md                        # Syllabus (fuente de verdad)
├── AGENTS.md                        # Este archivo
├── CLAUDE.md                        # Configuración Claude Code
├── curriculum/
│   └── class-{01..26}/              # 26 clases del curso
│       ├── README.md                # Teoría para el estudiante
│       ├── lab/README.md            # Laboratorio paso a paso
│       ├── lab/rubric.md            # Rúbrica (solo clases 4, 8, 12, 16, 20)
│       ├── slides/README.md         # Presentación reveal.js
│       ├── facilitator/README.md    # Guía pedagógica
│       ├── test/README.md           # Solo en clases 8 y 20 (test de M2 y M5)
│       ├── test/questions.md        # 5 preguntas que cubren las 4 clases del módulo
│       ├── infographic/index.html   # Infografía HTML (opcional)
│       └── infographic/image-prompts.md
│   └── module-{1..6}/
│       └── MODULE-PLAN.md           # Blueprint del módulo (pre-aprobación)
├── code-challenges/                 # Retos asíncronos
├── context/                         # Documentación técnica de referencia
├── configs/                         # Configuraciones de proyecto
├── _entregas/                       # Entregables del estudiante
├── tests/                           # (Legacy / placeholders — pendiente de limpieza)
└── .claude/skills/                  # Skills de generación de contenido (junctions)
```

### Mapa clase → módulo

| Clases | Módulo | Nombre | Proyecto integrador |
|--------|--------|--------|---------------------|
| 01–04 | Módulo 1 | React Fundamentals | Agenda de Contactos (vista y navegación) |
| 05–08 | Módulo 2 | Data Fetching & Forms | Agenda de Contactos + consumo de API + CRUD |
| 09–12 | Módulo 3 | Bases de Datos SQL con Supabase | Movie Match (schema + RLS + cliente React) |
| 13–16 | Módulo 4 | Backend Full-Stack con Next.js | Movie Match Full-Stack (Next.js + Prisma sobre la misma DB) |
| 17–20 | Módulo 5 | Autenticación, Roles y Middleware | MinCommerce con NextAuth.js v5 |
| 21–26 | Módulo 6 | Proyecto Final | Aplicación libre (individual o grupal) |

---

## Convenciones

### Idioma y Encoding

- **Idioma:** Español latinoamericano
- **Encoding:** UTF-8 puro (tildes directas: á, é, í, ó, ú, ñ, ü)
- **Signos:** ¿?, ¡! (nunca omitir el signo de apertura)
- **Corrupción:** Si aparecen `�` o `ó`, el archivo está corrupto — regenerar

### Terminología técnica de desarrollo web

- **Conservar en inglés:** React, JSX, props, state, hooks, useState, useEffect, Context API, Router, async/await, fetch, REST, API, endpoint, middleware, MVC, ORM, schema, query, foreign key, join, OAuth, JWT, Next.js, App Router, Route Handler, Server Action, Server Component, Client Component, TypeScript, interface, deploy, build, bundler, pnpm, package, Supabase, Row Level Security (RLS), Prisma, prop drilling, lifting state, controlled component.
- **Traducir al español:** Explicaciones pedagógicas, objetivos de aprendizaje, instrucciones del lab, prompts para el estudiante.
- **Formato:** Primera vez que aparece un acrónimo explicarlo: "ORM (Object-Relational Mapping)". Las siguientes apariciones, solo el acrónimo.

### Enlaces Externos (GitHub Pages / Kramdown)

```markdown
# Externos: SIEMPRE con {:target="_blank"}
[React Docs](https://react.dev/){:target="_blank"}

# Internos: NUNCA con target
[Clase 01](../class-01/)
[Sección](#seccion)
```

### Emojis

Estratégicos en headers para escaneo visual. No decorativos ni excesivos.

---

## Tono por Tipo de Archivo

| Archivo | Audiencia | Tono | Persona |
|---------|-----------|------|---------|
| `README.md` | Estudiante (pre-clase) | Profesional, inspirador | Segunda persona ("construirás", "implementarás") |
| `lab/README.md` | Estudiante (en clase) | Práctico, paso a paso, preciso | Segunda persona ("crea", "instala", "ejecuta") |
| `slides/README.md` | Facilitador (en clase) | Educativo, conversacional | Mixta |
| `facilitator/README.md` | Instructor (pre-clase) | Reflexivo, mentor-a-mentor, técnico | Narrativo estilo Medium |

---

## Límites por Archivo

| Archivo | Límite | Notas |
|---------|--------|-------|
| `README.md` de clase | ~150 líneas | Solo resumen y preparación |
| `lab/README.md` | Regular: <400, Integrador (C4): <500, Demo Day: <150 | |
| `slides/README.md` | ≤13 slides | Reveal.js markdown (separador `---`) |
| `facilitator/README.md` | <300 líneas | ~8 min lectura |
| `infographic/index.html` | ~500 líneas | HTML autocontenido con CSS |
| `class-{08,20}/test/README.md` | ~30 líneas | Solo info para estudiante |
| `class-{08,20}/test/questions.md` | 5 preguntas por test | Cubre las 4 clases del módulo (M2: 5-8, M5: 17-20) |

---

## Tipos de Lab

Determinar tipo según posición de la clase en el módulo:

```
Posición = ((clase - 1) % 4) + 1   # válido para módulos 1-5 (clases 1-20)
Lab integrador = (Posición == 4)   # clases 4, 8, 12, 16, 20
Demo Day = (clase == 26)            # única clase de Demo Day del curso
```

| Tipo | Clases | Partes | Tiempo | Rúbrica |
|------|--------|--------|--------|---------|
| Regular | 1-3, 5-7, 9-11, 13-15, 17-19 | 3 | ~90 min (en clase) | No |
| Integrador (C4) | 4, 8, 12, 16, 20 | 4-5 | ~135 min (60% clase + 40% post) | Sí: `rubric.md` separado (5 secciones, NO inline) |
| Proyecto Final (M6) | 21-25 | Sprints (Ideation, Alpha, Validation, Beta, Polish) | Variable | Hitos por clase |
| Demo Day | 26 | Especial (presentación pública) | Variable | Sí: rúbrica de proyecto final |

**Nota sobre "Lab Integrador":** No existe como elemento separado. El Lab de la Clase 4 de cada módulo (4, 8, 12, 16, 20) **es** el laboratorio integrador — su diseño abarca y conecta los temas de las clases 1–3 del módulo. El sistema lo genera con el mismo skill `/class-lab`.

**Nota sobre Módulo 6:** Es estructuralmente distinto a los módulos 1-5. Son 6 clases de proyecto libre con sprints de desarrollo, no contenido teórico nuevo. El skill `/class-readme` debe enfocarse en hitos del sprint, no en teoría.

**Nota sobre Demo Day:** Ocurre **única y exclusivamente** en la Clase 26 (Módulo 6). Es la presentación pública del proyecto final.

---

## Sistema de Evaluación

Leer de `README.md`. Reglas derivadas:

- Solo la **última clase de los módulos 1-5 (C4 de cada módulo)** tiene lab calificado/integrador con rúbrica
- Rúbrica estándar: **5 criterios × 20 pts = 100 pts** (4 técnicos + 1 "Logros Adicionales")
- Estructura de `rubric.md`: 5 secciones (criterios + escala + checklist de entrega + banderas rojas + notas para el evaluador)
- Escala: A (90-100), B (80-89), C (70-79), F (<70)
- El Módulo 6 (Proyecto Final) usa rúbrica de proyecto: funcionalidad + calidad de código + presentación + documentación
- "Demo Day" es EXCLUSIVO de clase 26; las demás clases 4/8/12/16/20 tienen "Presentación del Lab Integrador" (interna)

### Labs evaluados

| Módulo | Clase | Lab Evaluado |
|--------|-------|--------------|
| M1 | 4 | Contact Manager v1.0 |
| M2 | 8 | Contact Manager + CRUD |
| M3 | 12 | Movie Match en Supabase (schema + RLS + cliente React) |
| M4 | 16 | Movie Match Full-Stack (Next.js + Prisma) |
| M5 | 20 | MinCommerce + Auth |
| M6 | 26 | Proyecto Final (Demo Day) |

### Tests Diagnósticos

> **Cambio de diseño (2026):** El curso ya NO tiene un test por módulo. Solo existen DOS tests diagnósticos en todo el curso.

| Test | Cubre clases | Carpeta | Preguntas |
|------|--------------|---------|-----------|
| Test del Módulo 2 | Clases 5–8 (Asincronismo, Service Layer, Formularios+CRUD, Validación+localStorage) | `curriculum/class-08/test/` | **5** |
| Test del Módulo 5 | Clases 17–20 (NextAuth.js v5 fundamentos, Persistencia con Prisma Adapter, Roles, Middleware) | `curriculum/class-20/test/` | **5** |

**Reglas de diseño de tests:**

- Cada test cubre **las 4 clases de su módulo** de forma balanceada.
- Distribución fija: **4 preguntas focalizadas (1 por clase del módulo) + 1 pregunta integradora** que conecta conceptos de varias clases. **No incluir pregunta de autoevaluación** — los tests son a libro abierto e intentos ilimitados, así que la autoevaluación aporta poco.
- Formato Blackboard en archivo único `questions.md` con 4 alternativas (A–D) por pregunta y `> Respuesta: X` debajo.
- Tests son a libro abierto, intentos ilimitados.
- Los tests **no afectan calificación final** — son control interno pedagógico y preparación para Code 401.
- Al usar `/module-test`, **override** del default del skill (que asume 8 preguntas por test): aquí son **5 preguntas (4 focalizadas + 1 integradora)**.

---

## Pipeline de Generación de Contenido

### Flujo para un módulo nuevo

```
1. /module-planner           →  MODULE-PLAN.md (requiere aprobación humana)
2. Para cada clase (secuencial, C1 → C4):
   /class-readme → /class-lab → /class-slides → /class-facilitator
3. /evaluation-class         →  Verificar cada clase generada
4. /lint-markdown            →  Validar enlaces, encoding, formato
5. /class-infographic        →  Opcional, post-clase (resumen visual)
6. /module-updater           →  Mantenimiento posterior si los skills cambian
```

### Dependencias entre archivos

| Al generar... | Leer primero... |
|---------------|-----------------|
| `README.md` de clase | `README.md` del curso, `MODULE-PLAN.md` del módulo, clase anterior |
| `lab/README.md` | `README.md` de la clase, `slides/README.md` si existe |
| `slides/README.md` | `README.md` de la clase, `lab/README.md` |
| `facilitator/README.md` | Todos los anteriores de la clase |
| `class-08/test/` | READMEs y slides de clases 5, 6, 7, 8 |
| `class-20/test/` | READMEs y slides de clases 17, 18, 19, 20 |

### Regla de contexto

Nunca generar contenido sin leer los archivos de dependencia. Si un archivo de dependencia no existe, generarlo primero o pedir al usuario que lo proporcione.

---

## Scaffolding para laboratorios técnicos de desarrollo web

> Variables de nivel para que los skills compartidos adapten su output a un curso técnico intermedio (con prerrequisito Code 201).

### Perfil del estudiante

- Egresados de Code 201 con dominio sólido de HTML/CSS, JavaScript (paradigmas, callbacks, prototipos), DOM, LocalStorage y Git (branching, PRs)
- **Con base técnica previa:** NO introducir conceptos básicos (qué es un array, qué es una función, qué es un commit)
- **Sin experiencia previa en frameworks/backend:** sí introducir desde cero React, Node, Next.js, PostgreSQL, Supabase, Prisma, Auth, etc.
- **Requisito técnico:** equipo con Node.js LTS, navegador moderno y editor con soporte de extensiones (VS Code recomendado)

### Variables de Nivel

| Variable | Valor | Descripción |
|----------|-------|-------------|
| `course_level` | 2 | Complejidad intermedia (curso técnico con base JS previa) |
| `scaffolding_style` | descriptive | Gaps progresivos: laboratorios con pistas contextuales, no spec-based puros |
| `part_naming` | Parte | Secciones del lab: "Parte 1", "Parte 2"... |
| `checkpoint_style` | functional | Checkpoints verificables por comportamiento (UI funciona, endpoint responde, query devuelve filas) |
| `instruction_style` | step-by-step | Instrucciones paso-a-paso con comandos exactos y validaciones intermedias |
| `gap_types` | comment-placeholders+blank-lines | El estudiante completa código en huecos guiados (no escribe componentes desde specs) |

### Tabla de Autonomía por Módulo

La autonomía del estudiante aumenta progresivamente. En los primeros módulos todo está guiado; al final se esperan decisiones independientes (diseñar arquitectura, modelar DB, integrar OAuth).

| Módulo | Guiado | Autonomía | Descripción |
|--------|--------|-----------|-------------|
| Módulo 1 (React Fundamentals) | 90% | 10% | Componentes y rutas con código casi listo, props guiadas |
| Módulo 2 (Data Fetching) | 80% | 20% | Service layer y CRUD con pistas; el estudiante decide nombres de funciones |
| Módulo 3 (DB SQL con Supabase) | 75% | 25% | SQL guiado en SQL Editor + cliente `supabase-js` con scaffolding; modelado con criterios provistos |
| Módulo 4 (Backend Next.js + Prisma) | 65% | 35% | Migración `supabase-js`→Prisma con guía estructural; Route Handlers con criterio propio |
| Módulo 5 (Auth, Roles, Middleware) | 60% | 40% | NextAuth + RBAC con scaffolding; políticas de protección con criterio propio |
| Módulo 6 (Proyecto Final) | 30% | 70% | Decisiones independientes: stack, arquitectura, UX, presentación |

### Formato de Gaps (nivel descriptive)

**Ejemplo — componente React:**

```jsx
function ContactCard({ contact }) {
  return (
    <div className="contact-card">
      <h3>{/* completar: nombre del contacto */}</h3>
      <p>{contact.____}</p>      {/* completar: campo email */}
      <button onClick={() => /* completar: handler para eliminar */}>
        Eliminar
      </button>
    </div>
  );
}
```

**Ejemplo — Route Handler de Next.js:**

```ts
// app/api/movies/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';

export async function GET(req: NextRequest, { params }: { params: { id: string } }) {
  const id = ____;                            // completar: parsear params.id a número
  const movie = await prisma.movie.findUnique({ where: { ____ } });  // completar
  if (!movie) return NextResponse.json({ error: 'Not found' }, { status: ____ });  // completar status
  return NextResponse.json(movie);
}
```

### Checkpoints (nivel functional)

> ✅ **Checkpoint Parte 1:** Tu app de React monta sin errores en la consola y muestra la lista de contactos hardcodeada. Captura de pantalla del navegador con DevTools abiertos (sin warnings).

> ✅ **Checkpoint Parte 2:** El Route Handler `GET /api/movies` (Next.js) devuelve un array JSON con al menos 3 películas desde Postgres vía Prisma. Verifica con `curl` o el navegador y adjunta el response.

---

## Tecnologías Principales del Curso

Registradas aquí para referencia rápida. El detalle de en qué clase se usa cada una está en `README.md`.

| Tecnología | Uso | Módulo(s) |
|------------|-----|-----------|
| **Node.js + pnpm** | Runtime y gestor de paquetes (reemplaza npm en todo el curso) | Todos |
| **Vite** | Build tool y dev server para React | 1, 2, 3 |
| **React + React Router** | Framework de UI y enrutamiento SPA | 1, 2, 3 |
| **PostgreSQL + Supabase** | Base de datos relacional administrada (misma DB de M3 a M6) | 3, 4, 5, 6 |
| **`supabase-js`** | Cliente JS desde el navegador (SQL-first) | 3 |
| **Row Level Security (RLS)** | Autorización a nivel de base de datos | 3, 5 |
| **Next.js 15 (App Router)** | Framework full-stack (Route Handlers + Server Actions). Reemplaza a Express. | 4, 5, 6 |
| **TypeScript** | Tipado estático introducido en M4-C1 | 4, 5, 6 |
| **Prisma** | ORM tipado introducido como migración del cliente Supabase en M4-C1 | 4, 5, 6 |
| **NextAuth.js v5 + Prisma Adapter** | Autenticación OAuth2 (no se usa Supabase Auth) | 5 |
| **Vercel** | Despliegue (free tier) | 4, 5, 6 |
| **GitHub Copilot / Claude** | Asistencia de IA integrada | 1-5 |
| **Git + GitHub** | Control de versiones | Todos |

---

## Modificar Contenido Existente

1. Actualizar PRIMERO `README.md` (syllabus) si el cambio afecta estructura
2. Los archivos de clase referencian al syllabus — no duplicar tablas
3. Verificar sincronización con clases vecinas (la teoría se acumula)
4. Ejecutar `/lint-markdown` antes de considerar completo
5. Para refrescar un módulo completo tras cambios en skills: usar `/module-updater`

---

## Estado del Repositorio

```
README.md       ✅  Syllabus consolidado
AGENTS.md       ✅  Este documento (en preparación de edición)
CLAUDE.md       ✅  Configuración Claude Code
.claude/skills/ ✅  Junctions a shared-skills sincronizados
FASE 3          🔄  Generación/edición de contenido en curso
```

---
> Source: [entertechschool/code-301-guide](https://github.com/entertechschool/code-301-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
