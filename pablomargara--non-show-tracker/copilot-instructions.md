## non-show-tracker

> Dashboard online para que el equipo UXR registre no-shows de entrevistas con usuarios. Al final del año, el que tenga más no-shows "gana".

# UXR No-Show Tracker — Contexto del Proyecto

## Qué es esto
Dashboard online para que el equipo UXR registre no-shows de entrevistas con usuarios. Al final del año, el que tenga más no-shows "gana".

## Equipo
- Anna
- Aleja
- Hector
- Pablo (pablomargara@gmail.com)

## URL en producción
**https://pablomargara.github.io/NON-SHOW-tracker-/**

## Repositorio GitHub
**https://github.com/PabloMargara/NON-SHOW-tracker-**
- Repo público
- GitHub Pages activo, branch `main`, root `/`

## Stack
- **Frontend:** HTML + CSS + JS vanilla, archivo único (`index.html`)
- **Base de datos:** Supabase (PostgreSQL)
- **Auth:** Google OAuth via Supabase
- **Hosting:** GitHub Pages (estático, gratis)

## Supabase
- **Project ID:** `sndfpwokkgqlxylkfxfx`
- **URL:** `https://sndfpwokkgqlxylkfxfx.supabase.co`
- **Publishable key:** `sb_publishable_iZ80o9a6xxn0123VYv4MLA_ARMfgS9i`
- **Organización:** PabloMargara's Org
- **Proyecto:** uxr-noshow-tracker

### Tabla en Supabase
```sql
entries (
  id          UUID PRIMARY KEY,
  member      TEXT CHECK (member IN ('Anna','Aleja','Hector','Pablo')),
  date        DATE,
  created_by  UUID REFERENCES auth.users(id),
  created_at  TIMESTAMPTZ DEFAULT now()
)
```
- RLS habilitado
- Todos los autenticados pueden leer e insertar
- Solo el creador puede eliminar sus propias entradas

## Google OAuth
- **Proyecto Google Cloud:** `uxr-noshow-tracker` (ID: 962659539569)
- **Client ID:** `962659539569-lfcb3omtmot5rpnt2s45dc7poo9ins44.apps.googleusercontent.com`
- **Authorized JS origins:** `https://PabloMargara.github.io`
- **Redirect URI:** `https://sndfpwokkgqlxylkfxfx.supabase.co/auth/v1/callback`

## Supabase Auth config
- **Site URL:** `https://pablomargara.github.io/NON-SHOW-tracker-/`
- **Redirect URLs:** `https://pablomargara.github.io/NON-SHOW-tracker-/`
- **Provider:** Google (habilitado)

## Archivos del proyecto
| Archivo | Propósito |
|---------|----------|
| `index.html` | App completa (login + dashboard) |
| `supabase_setup.sql` | SQL para crear tabla y políticas |
| `SETUP.md` | Guía de setup detallada |
| `README.md` | Descripción del proyecto |
| `CLAUDE.md` | Este archivo |

## Cómo hacer deploy
```bash
cd "/Users/pablo.andres.margara/Documents/CLAUDE/Projects/NON-SHOW - Counter"
git add .
git commit -m "mensaje"
git push
```
GitHub Pages se actualiza automáticamente en ~1-2 minutos.

## Historial de problemas resueltos
- **GitHub API 401/403:** Se eliminó el approach de GitHub API y se migró a Supabase
- **localhost redirect:** Se corrigió en Supabase → Authentication → URL Configuration
- **Deploy viejo:** Se eliminó `.github/workflows/` que dejó Copilot y causaba deploys de archivos viejos
- **Pages no actualizaba:** Había un workflow de Copilot que sobreescribía el deploy

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PabloMargara) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
