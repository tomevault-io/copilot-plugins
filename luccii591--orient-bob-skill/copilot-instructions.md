## orient-bob-skill

> Convertir cualquier repositorio Node/TypeScript en un tour guiado en menos de 60 segundos. Atacamos el dolor universal de "abrí un repo nuevo y no sé por dónde empezar".

# Orient — Bob Companion para onboarding instantáneo en repos Node/TypeScript

## Misión

Convertir cualquier repositorio Node/TypeScript en un tour guiado en menos de 60 segundos. Atacamos el dolor universal de "abrí un repo nuevo y no sé por dónde empezar".

Orient es un **Bob Skill** (`/orient`) que analiza el repo activo y genera artefactos navegables: arquitectura, ruta de lectura recomendada, hot files, y onboarding checklist. Un **dashboard Next.js** los renderiza de forma interactiva.

La arquitectura es extensible a otros lenguajes (Python, Java, Go) en versiones futuras, pero el MVP se enfoca exclusivamente en Node/TypeScript para garantizar profundidad de análisis.

## Stack obligatorio

- **Skill**: archivo markdown literate-coding compatible con Bob Skills (`skill/orient.md`).
- **Dashboard**: Next.js 15 (App Router) + TypeScript estricto + Tailwind v4 + shadcn/ui + Mermaid.js.
- **Runtime**: Node 20+. Sin DB, sin auth, sin backend custom. Todo client-side leyendo un JSON.
- **Deploy demo**: Vercel.
- **Lenguaje del proyecto**: codigo en ingles (variables, comentarios), documentacion de usuario y video demo en espanol.

## Arquitectura (flujo end-to-end)

```
[repo Node/TS target]
        |
        v
[/orient skill ejecutado en Bob IDE]
        |
        v
[.orient/report.json + AGENTS.md generado en repo target]
        |
        v
[Dashboard Next.js (drag and drop del report.json)]
        |
        v
[Vistas: arquitectura, start-here, hot files, onboarding]
```

El skill produce dentro del repo target:
1. `AGENTS.md` enriquecido en la raiz (sobreescribe si existe, con confirmacion).
2. `.orient/report.json` con el analisis estructurado (consumido por el dashboard).
3. `.orient/architecture.mmd` (diagrama Mermaid embebido tambien en el JSON).

## Esquema canonico de `report.json`

```json
{
  "meta": {
    "repoName": "string",
    "analyzedAt": "ISO-8601",
    "primaryLanguage": "TypeScript | JavaScript",
    "totalFiles": 0,
    "loc": 0,
    "orientVersion": "0.1.0"
  },
  "stack": {
    "runtime": ["node@20"],
    "packageManager": "npm | pnpm | yarn | bun",
    "frameworks": ["Next.js", "Express"],
    "databases": [],
    "testing": ["Jest", "Vitest"],
    "ciCd": ["GitHub Actions"]
  },
  "architecture": {
    "mermaid": "graph TD; ...",
    "summary": "string",
    "entryPoints": [{ "path": "src/index.ts", "kind": "main | api | cli" }]
  },
  "startHere": [
    { "path": "string", "why": "string", "order": 1 }
  ],
  "hotFiles": [
    { "path": "string", "commits": 0, "uniqueAuthors": 0, "score": 0.0 }
  ],
  "onboardingChecklist": {
    "frontend": ["string"],
    "backend": ["string"],
    "fullstack": ["string"]
  },
  "risks": [
    { "severity": "low | med | high", "description": "string", "evidence": "string" }
  ]
}
```

## Deteccion de stack (reglas para el skill)

- **Package manager**: presencia de `pnpm-lock.yaml` -> pnpm; `yarn.lock` -> yarn; `bun.lockb` -> bun; default `package-lock.json` -> npm.
- **Framework**: leer `package.json.dependencies`. Reglas: `next` -> Next.js; `express` o `fastify` o `koa` -> backend; `react` sin `next` -> SPA React; `@nestjs/core` -> NestJS.
- **Testing**: `jest`, `vitest`, `mocha` en deps.
- **CI/CD**: presencia de `.github/workflows/*.yml`.
- **Entry points**: `package.json.main`, `package.json.bin`, `package.json.scripts.start|dev`.

## Calculo de hot files

Formula: `score = log(commits + 1) * sqrt(uniqueAuthors)`

Comando base: `git log --pretty=format: --name-only --since="1 year ago" | sort | uniq -c | sort -rn | head -30`

Filtros: excluir `node_modules`, `dist`, `build`, `.next`, archivos lock, `*.md` no criticos.

## Generacion de start-here path

Heuristica (no LLM, deterministico):
1. Entry point principal de `package.json`.
2. README (si existe y tiene >100 LOC).
3. Archivo de config principal (`next.config.js`, `tsconfig.json`).
4. Archivo mas conectado (mas imports incoming).
5. Test principal o folder de tests.
6. Carpeta `src/lib` o `src/utils` si existe.
7. Schema/types principal (`schema.ts`, `types.ts`).

Maximo 7 entradas, minimo 3.

## Convenciones de codigo

- **Naming**: TypeScript en camelCase, archivos en kebab-case, componentes React en PascalCase.
- **Tipado estricto**: zero `any`. Schemas validados con Zod en cliente.
- **Componentes**: server components por default, `"use client"` solo cuando hay state/interactividad.
- **Sin tests** salvo un smoke test del skill que verifique que `report.json` tiene el shape correcto.
- **Comentarios solo para lo no obvio.** Asumimos que el codigo se autodocumenta.

## Out of scope (anti-scope-creep estricto)

- Auth, DB, backend custom.
- Soporte multi-idioma del dashboard (todo UI en ingles, documentacion en espanol).
- CLI standalone fuera de Bob (el skill solo se invoca desde Bob).
- Analisis cross-repo o comparativo.
- Soporte de Python/Java/Go en MVP (mencionado como roadmap).
- Persistencia, historial, accounts.
- Cualquier feature que tome mas de 2h y no aparezca en el video de 3 min.

## Demo objetivo (storyboard del video)

- **0:00-0:20** Hook: "Cuando entras a un repo nuevo, cuanto tardas en saber por donde empezar?"
- **0:20-0:40** Problema: clonar un repo OSS popular (ej: `vercel/swr` o similar Node/TS), mostrar la confusion inicial.
- **0:40-2:00** Demo en vivo:
  - Abrir Bob IDE en el repo.
  - Ejecutar `/orient` en chat.
  - Bob analiza ~30s.
  - Abrir dashboard local.
  - Tour rapido: arquitectura, start-here, hot files.
- **2:00-2:30** Mostrar AGENTS.md generado, hablar de extensibilidad.
- **2:30-3:00** Cierre: "Construido con Bob, sobre Bob, para Bob, en 22 horas. Acelera onboarding en 60 segundos."

## Restricciones de Bobcoins

40 coins totales. Presupuesto:
- Plan + arquitectura: 4
- Skill implementation: 10
- Dashboard scaffold + UI: 10
- Integracion + bugs: 6
- Polish + writeups + video script: 5
- Buffer critico: 5

Regla de oro: prompts gordos con contexto completo en una sola pasada, no chats iterativos. Plan mode antes que Code mode.

## Estructura del repo

- `skill/orient.md` — el Bob Skill (logica de analisis).
- `dashboard/` — Next.js app que renderiza report.json.
- `examples/sample-output.json` — fixture para desarrollar el dashboard sin depender del skill.
- `bob_sessions/` — exports de tareas Bob (requisito de submision, screenshots + .md).
- `docs/` — writeups de submision, video script, budget tracking.
- `AGENTS.md` — este archivo, contrato del proyecto.
- `README.md` — instrucciones de uso para reviewers.

---
> Source: [luccii591/orient-bob-skill](https://github.com/luccii591/orient-bob-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
