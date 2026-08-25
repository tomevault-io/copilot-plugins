## vita

> Router del proyecto. Sistema Operativo Personal impulsado por IA (Flutter +

# CLAUDE.md — VITA

Router del proyecto. Sistema Operativo Personal impulsado por IA (Flutter +
Supabase). Costo objetivo **$0**.

- **Para entender VITA** (qué es, principios, voz, visual, módulos, docs):
  lee **`docs/VITA_MASTER.md`**.
- **Para ejecutar trabajo en VITA** (método, calidad, contexto progresivo):
  aplica el Skill **`vita-builder`** (`.claude/skills/vita-builder/`).
- **Para profundidad:** consulta **solo** el documento específico que la tarea
  requiera (mapa y matriz en `VITA_MASTER §9–§10`). No cargues todo por rutina.

Repo: `github.com/yurne21-arch/vita` · Deploy: https://yurne21-arch.github.io/vita/

## Reglas técnicas universales (siempre vigentes)

- **Clean Architecture:** `presentación → dominio ← datos`; dominio Dart puro.
  **Los features nunca se importan entre sí** (se comunican vía `core/`).
- Repositorios devuelven **entidades de dominio**, nunca modelos de BD. Sin
  lógica de negocio en widgets. Riverpod **sin codegen**. `async/await`; sin
  `catch` vacíos; inmutabilidad por `copyWith`.
- **Secretos por `--dart-define`, nunca en el repo.** `SUPABASE_ANON_KEY` es
  pública por diseño; protege la RLS.
- **Git:** trunk-based, Conventional Commits en presente, commits atómicos,
  `main` siempre desplegable.

## Comandos (desde `app/`)

```bash
flutter pub get
flutter analyze          # cero warnings nuevos
flutter test
dart format .            # antes de cada commit
flutter run -d chrome \
  --dart-define=SUPABASE_URL=https://TU-PROYECTO.supabase.co \
  --dart-define=SUPABASE_ANON_KEY=TU_ANON_KEY
flutter build web --release --base-href "/vita/" --pwa-strategy=none \
  --dart-define=SUPABASE_URL=... --dart-define=SUPABASE_ANON_KEY=...
```

Deploy automático: push a `main` → `.github/workflows/ci.yaml` (analyze → test →
build → GitHub Pages). Secrets `SUPABASE_URL` y `SUPABASE_ANON_KEY` en
*Settings → Secrets → Actions*; *Pages → Source: GitHub Actions*. El repo debe
llamarse `vita` y ser público.

## Estructura

```
app/lib/
  core/     theme/ (design system vivo) · widgets/ · router/ · providers.dart
            config/ · data/{remote,local,sync}/ · ai/ · content/
            design/tokens/  ← MUERTO, no usar (el vivo es core/theme/)
  features/<dominio>/{presentation,domain,data}/
            auth · profile · mi_vida · proyectos · agenda · salud · ajustes
supabase/migrations/   docs/{diseno,adr}/
```

## Base de datos (operativo)

Supabase free: Postgres + RLS + Auth + Storage. Proyecto `vita-dev`
(`stxnyopihcfhgpizllxg`, sa-east-1). Toda tabla con RLS `user_id = auth.uid()`.
Tablas `_events` **append-only** (solo SELECT/INSERT). Migraciones
**forward-only** en `supabase/migrations/` (`0001_init`, `0002_dominios_mvp`);
una migración aplicada **nunca se edita**.

Reglas de negocio en triggers: máx. 3 prioridades/día (`dp_max_tres`); un solo
proyecto principal activo (`projects_principal_guard` + índice único parcial);
`completada_at` al completar un paso.

`db pull`/`db dump` requieren Docker (no instalado). Para inspeccionar el
esquema remoto, Management API:

```bash
curl -s -X POST "https://api.supabase.com/v1/projects/stxnyopihcfhgpizllxg/database/query" \
  -H "Authorization: Bearer $(security find-generic-password -s 'Supabase CLI' -w)" \
  -H "Content-Type: application/json" -d '{"query":"select 1"}'
```

Decisiones cerradas y pendientes reales (offline/Drift, proveedor de IA,
`anonKey`, Inter, tokens muertos, `vita-prod`): **`VITA_MASTER §11–§12`.**

---
> Source: [yurne21-arch/vita](https://github.com/yurne21-arch/vita) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
