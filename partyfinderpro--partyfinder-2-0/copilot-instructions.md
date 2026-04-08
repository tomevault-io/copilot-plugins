## partyfinder-2-0

> **Proyecto**: venuz-app (VENUZ)

# Instrucciones para Claude (Anthropic) — VENUZ

**Proyecto**: venuz-app (VENUZ)
**Contexto**: Arreglar feed duplicados, deploy, seguridad.
**Workspace**: `c:\Users\pablo\Downloads\VENUZ-Complete-App\venuz-app`

## Contexto para Claude
El usuario ya ha realizado los siguientes pasos manuales:
1. Rotar las API keys en Vercel, Supabase, Google AI Studio y Upstash.
2. Ejecutar la migración SQL detallada abajo en Supabase.

## Tareas a Ejecutar por Claude
Por favor, ubícate en la ruta del workspace y ejecuta los siguientes comandos en orden:

### 1. Build del Proyecto
Ejecuta el siguiente comando para compilar el proyecto y verificar que no hay errores que rompan el sistema:
```bash
npm run build
```

### 2. Despliegue en Vercel
Si el build pasa sin arrojar errores fatal, despliega los últimos cambios a producción (asegúrate de que todo está commiteado si usas git + vercel, o usa el CLI):
```bash
npx vercel --prod --yes
```

### 3. (Opcional) Scraping de Vallarta
Para actualizar la base de datos con los eventos y lugares de la noche de hoy en Vallarta:
```bash
npm run scrape:vallarta
```

---

## Código SQL de Deduplicación (Ejecutado por el Usuario)

Solo como referencia arquitectónica de lo que el usuario aplicó en la base de datos (Supabase) para eliminar duplicados del feed:

```sql
-- Agregar columna hash para URLs
ALTER TABLE public.content ADD COLUMN IF NOT EXISTS source_url_hash TEXT;

-- Popular el hash
UPDATE public.content
SET source_url_hash = md5(lower(trim(source_url)))
WHERE source_url IS NOT NULL AND source_url != ''
  AND (source_url_hash IS NULL OR source_url_hash = '');

-- Eliminar duplicados en 'content' dejando solo los mas recientes
DELETE FROM public.content WHERE id IN (
  SELECT id FROM (
    SELECT id, ROW_NUMBER() OVER (PARTITION BY source_url_hash ORDER BY scraped_at DESC NULLS LAST, created_at DESC NULLS LAST) AS rn
    FROM public.content WHERE source_url_hash IS NOT NULL
  ) sub WHERE rn > 1
);

-- Recrear el indice y forzar restriccion de unicidad
DROP INDEX IF EXISTS idx_content_source_url_hash ON public.content;
CREATE UNIQUE INDEX IF NOT EXISTS idx_content_source_url_hash ON public.content(source_url_hash) WHERE source_url_hash IS NOT NULL;

-- Limpiar la tabla base de scrapers 'scraped_items'
DELETE FROM public.scraped_items WHERE id IN (
  SELECT id FROM (
    SELECT id, ROW_NUMBER() OVER (PARTITION BY original_url ORDER BY created_at DESC NULLS LAST) AS rn
    FROM public.scraped_items WHERE original_url IS NOT NULL AND original_url != ''
  ) sub WHERE rn > 1
);
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/partyfinderpro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/partyfinderpro)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
