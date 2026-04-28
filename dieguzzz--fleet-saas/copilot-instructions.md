## fleet-saas

> - Next.js 16 App Router (Turbopack)

# Reglas y errores documentados — Fleet SaaS

## Stack
- Next.js 16 App Router (Turbopack)
- Supabase (`@supabase/ssr`) con Row Level Security
- TypeScript estricto
- Railway para deploy (Dockerfile propio)

---

## REGLA 1 — Tipos de Supabase deben estar actualizados

**Error que ocurre:**
```
Type error: Argument of type '"nombre_funcion"' is not assignable to parameter of type '"funcion_a" | "funcion_b"'
```

**Causa:** Se agregó una función RPC a la base de datos pero no se regeneraron los tipos TypeScript.

**Solución correcta:**
1. Crear la función en la DB con `mcp__supabase__apply_migration`
2. Regenerar tipos con `mcp__supabase__generate_typescript_types`
3. Reemplazar el contenido de `src/types/supabase.ts` con los tipos nuevos

**Regla:** Cada vez que se agrega o modifica una función/tabla en Supabase, regenerar y actualizar `src/types/supabase.ts` antes de hacer commit.

---

## REGLA 2 — RLS: `auth.uid()` puede ser NULL en PostgREST aunque `getUser()` funcione

**Error que ocurre:**
```
new row violates row-level security policy for table "organizations"
```

**Causa:** `supabase.auth.getUser()` llama a la API de Auth directamente con el JWT del cookie — por eso funciona. Pero `supabase.from('tabla').insert()` llama a PostgREST, que necesita que el JWT llegue en el header `Authorization`. Si hay algún problema con cómo se leen los cookies (ej. inline server action en un Server Component, sesión de otro proyecto), PostgREST no recibe el JWT y `auth.uid()` es NULL.

**Solución correcta para operaciones de bootstrap (primera inserción de un usuario):**
Usar una función `SECURITY DEFINER` en Postgres que hace la inserción internamente. Ejemplo:

```sql
CREATE OR REPLACE FUNCTION create_organization_for_user(org_name TEXT, org_slug TEXT)
RETURNS TABLE(id UUID, name TEXT, slug TEXT)
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public
AS $$
DECLARE
  v_user_id UUID;
  v_org organizations%ROWTYPE;
BEGIN
  v_user_id := auth.uid();
  IF v_user_id IS NULL THEN
    RAISE EXCEPTION 'Not authenticated';
  END IF;
  -- hacer inserts aquí...
END;
$$;
GRANT EXECUTE ON FUNCTION create_organization_for_user(TEXT, TEXT) TO authenticated;
```

Llamar desde el action con `supabase.rpc('create_organization_for_user', { org_name, org_slug })`.

**Regla:** Para inserts de tablas con RLS estricta en el flujo de onboarding/bootstrap, usar SECURITY DEFINER en lugar de INSERT directo desde el cliente.

---

## REGLA 3 — RLS: Recursión infinita en políticas SELECT

**Error que ocurre:**
```
infinite recursion detected in policy for relation "organization_members"
```

**Causa:** Una política SELECT de la tabla `X` hace `SELECT FROM X WHERE user_id = auth.uid()` — se llama a sí misma.

**Solución correcta:**
Crear una función `SECURITY DEFINER` que bypasea RLS y usarla en las políticas:

```sql
CREATE OR REPLACE FUNCTION get_user_org_ids()
RETURNS uuid[] LANGUAGE sql SECURITY DEFINER SET search_path = public AS $$
  SELECT ARRAY(SELECT organization_id FROM organization_members WHERE user_id = auth.uid())
$$;

-- En las políticas, usar:
CREATE POLICY "..." ON tabla FOR SELECT
  USING (organization_id = ANY(get_user_org_ids()));
```

**Regla:** Nunca escribir una política RLS que haga SELECT en su propia tabla. Siempre usar funciones SECURITY DEFINER para romper la recursión.

---

## REGLA 4 — RLS: Bug de auto-comparación en políticas INSERT

**Error que ocurre:** El INSERT siempre falla o siempre pasa (comportamiento incorrecto).

**Causa:** Bug de copy-paste en una política — se compara la tabla consigo misma:
```sql
-- MAL: siempre TRUE
organization_members_1.organization_id = organization_members_1.organization_id
```

**Solución correcta:** Verificar que las políticas comparen columnas contra `auth.uid()` u otros valores externos, no contra sí mismas. Usar funciones SECURITY DEFINER cuando la lógica es compleja.

---

## REGLA 5 — Páginas de onboarding / formularios deben ser Client Components con `useActionState`

**Error que ocurre:** El formulario no avanza, no muestra errores, o `auth.uid()` es NULL en la action.

**Causa:** Un Server Component con inline `'use server'` action no propaga correctamente el JWT en todos los casos. El patrón `useActionState` en Client Components es más robusto.

**Solución correcta:**
```tsx
'use client';
import { useActionState, useEffect } from 'react';
import { useRouter } from 'next/navigation';
import { miServerAction } from '@/features/xxx/actions';

export default function MiForm() {
  const router = useRouter();
  const [state, formAction, pending] = useActionState(
    async (_prev: State, formData: FormData) => miServerAction(formData),
    null
  );

  useEffect(() => {
    if (state?.success && state.slug) router.push(`/${state.slug}`);
  }, [state, router]);

  return (
    <form action={formAction}>
      {state?.error && <div className="text-red-400">{state.error}</div>}
      {/* campos */}
      <button disabled={pending}>{pending ? 'Cargando...' : 'Enviar'}</button>
    </form>
  );
}
```

**Regla:** Todos los formularios que llaman server actions deben ser Client Components usando `useActionState`. Nunca usar inline `'use server'` en Server Components para mutaciones.

---

## REGLA 6 — Variables de entorno: `.env.local` Y Railway deben coincidir

**Error que ocurre:** Widget "Connection Failed" en producción aunque funcione local, o viceversa.

**Variables requeridas:**
```
NEXT_PUBLIC_SUPABASE_URL=https://<project-ref>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...  (JWT, empieza con eyJ)
```

**Regla:** Al cambiar de proyecto Supabase, actualizar ambos: `.env.local` (local) y las variables de entorno en Railway (producción). El anon key debe ser el JWT (`eyJ...`), no el publishable key (`sb_publishable_...`).

---

## REGLA 7 — `revalidatePath` en actions debe usar rutas template para dynamic segments

**Error que ocurre:** La UI no se actualiza después de un create/update/delete.

**Causa:** Usar `revalidatePath('/org/${orgId}/items')` con una variable — Next.js no puede hacer match con la ruta dinámica en caché.

**Solución correcta:**
```ts
// Para invalidar todas las páginas de un segmento dinámico:
revalidatePath('/[orgSlug]/inventory/items', 'page');

// Para invalidar una ruta concreta cuando se tiene el slug:
revalidatePath(`/${orgSlug}/inventory/items`);

// Para invalidar todo el layout:
revalidatePath('/', 'layout');
```

**Regla:** Usar el patrón `[param]` con `'page'` o `'layout'` cuando no se tiene el valor concreto. Usar la ruta concreta cuando sí se tiene.

---

## REGLA 8 — Profiles: política INSERT requerida

**Error que ocurre:** `403` al hacer upsert en `profiles` durante onboarding.

**Causa:** La tabla `profiles` tiene RLS pero no tiene política INSERT para el propio usuario.

**Migración necesaria (ya aplicada):**
```sql
CREATE POLICY "Users can insert own profile" ON profiles
  FOR INSERT WITH CHECK (auth.uid() = id);
```

**Regla:** La tabla `profiles` necesita políticas para SELECT, INSERT y UPDATE del propio usuario. Verificar que existen al crear el schema inicial.

---

## REGLA 9 — Supabase joins con `as Invoice[]` requieren cast doble `as unknown as Type[]`

**Error que ocurre:**
```
Type error: Conversion of type '{ ...; customer: { name: string } | null; }[]' to type 'Invoice[]' may be a mistake
Types of property 'customer' are incompatible.
Type '{ name: string; }' is missing the following properties from type 'Contact': id, organization_id, ...
```

**Causa:** Cuando se hace un join parcial en Supabase (ej. `.select('*, customer:contacts(name)')`), el tipo inferido por TypeScript es `{ name: string }`, pero el tipo local `Invoice` define `customer` como el tipo completo `Contact`. El cast directo `as Invoice[]` falla porque TypeScript detecta que los tipos no se solapan suficientemente.

**Solución correcta:**
```ts
return { data: data as unknown as Invoice[] };
```

**Regla:** Cuando el resultado de una query Supabase con joins parciales se castea a un tipo local que tiene relaciones completas, usar siempre `as unknown as TipoLocal[]`. Nunca usar cast directo `as TipoLocal[]` si el join selecciona menos campos que el tipo completo.

---

## REGLA 10 — Siempre actualizar AMBOS archivos de tipos cuando cambia el schema de DB

**Error que ocurre:**
```
Type error: Object literal may only specify known properties, and 'nombre_campo' does not exist in type 'CreateSomeInput'
```

**Causa:** El proyecto tiene DOS archivos de tipos:
- `src/types/supabase.ts` — generado automáticamente con `mcp__supabase__generate_typescript_types`
- `src/types/database.ts` — mantenido manualmente con interfaces como `Invoice`, `Contact`, etc.

El código de negocio (actions, components) importa desde `@/types/database`. Si se agrega una columna nueva a la DB y solo se actualiza `supabase.ts`, el tipo en `database.ts` queda desactualizado y el build falla en Railway.

**Ejemplo concreto:** Se agregó `invoice_type` a la tabla `invoices`. Se regeneró `supabase.ts` pero no se actualizó la interfaz `Invoice` en `database.ts`. Resultado: `'invoice_type' does not exist in type 'CreateInvoiceInput'`.

**Solución correcta — al agregar una columna nueva:**
1. Agregar la columna en Supabase con `mcp__supabase__apply_migration`
2. Regenerar `src/types/supabase.ts` con `mcp__supabase__generate_typescript_types`
3. **Agregar el campo también a la interfaz correspondiente en `src/types/database.ts`**

**Regla:** Cualquier cambio de schema de DB requiere actualizar los DOS archivos: `supabase.ts` (auto-generado) Y `database.ts` (manual). Nunca actualizar solo uno de los dos.

---

## REGLA 11 — Railway: variables `NEXT_PUBLIC_*` deben declararse como ARG en el Dockerfile

**Error que ocurre:** En producción la app no conecta a Supabase aunque las variables estén configuradas en Railway.

**Causa:** Next.js embebe las variables `NEXT_PUBLIC_*` en el bundle al momento de `npm run build`. En Docker, las variables de entorno de Railway están disponibles en runtime pero NO durante el build del Dockerfile. Si el Dockerfile no declara `ARG` para esas variables antes de `RUN npm run build`, quedan vacías en el bundle.

**Solución correcta en el Dockerfile:**
```dockerfile
# Declarar ARGs antes del build
ARG NEXT_PUBLIC_SUPABASE_URL
ARG NEXT_PUBLIC_SUPABASE_ANON_KEY

# Pasar como ENV para que Next.js los tome durante el build
ENV NEXT_PUBLIC_SUPABASE_URL=$NEXT_PUBLIC_SUPABASE_URL
ENV NEXT_PUBLIC_SUPABASE_ANON_KEY=$NEXT_PUBLIC_SUPABASE_ANON_KEY

RUN npm run build
```

En Railway, configurar `NEXT_PUBLIC_SUPABASE_URL` y `NEXT_PUBLIC_SUPABASE_ANON_KEY` como variables de entorno del servicio (bajo Settings → Variables). Railway las inyecta como build args automáticamente si coinciden con los ARG del Dockerfile.

**Regla:** Toda variable `NEXT_PUBLIC_*` usada en el código cliente DEBE tener su `ARG` + `ENV` correspondiente en el Dockerfile antes del `RUN npm run build`.

---

## REGLA 12 — Supabase Storage: subida y visualización de archivos adjuntos

### Upload desde el cliente (Client Component)

```ts
const supabase = createClient(); // @/services/supabase/client — usa sesión del cookie
const path = `${orgId}/carpeta/${itemId}.${ext}`;

const { error } = await supabase.storage
  .from('nombre-bucket')
  .upload(path, file, { upsert: true });

const { data } = supabase.storage.from('nombre-bucket').getPublicUrl(path);
// Guardar data.publicUrl en la DB via server action
```

**Políticas RLS necesarias en storage.objects:**
- `INSERT` — `with_check`: verifica que `(storage.foldername(name))[1]` (primer segmento del path) sea un org_id al que pertenece el usuario
- `UPDATE` — igual que INSERT
- `SELECT` — público (si el bucket es público)

**Regla:** El primer segmento del path SIEMPRE debe ser el `organization_id` del usuario para que las políticas RLS puedan verificarlo.

### Visualización de PDFs en el browser (sin abrir Adobe)

Supabase Storage envía el header `X-Frame-Options: SAMEORIGIN`, lo que bloquea cargar la URL directamente en `<iframe>`, `<embed>` u `<object>`. Las alternativas con blob URL del cliente también fallan por CORS o porque el browser no puede cargar el blob.

**Solución correcta y definitiva: API route proxy en Next.js**

El servidor hace el fetch (sin CORS), responde con `Content-Type: application/pdf` y sin `X-Frame-Options`. El `<object>` carga desde la misma origin de la app.

```ts
// src/app/api/pdf-proxy/route.ts
export async function GET(request: NextRequest) {
  const url = request.nextUrl.searchParams.get('url');
  // Validar que la URL sea del bucket propio de Supabase
  const upstream = await fetch(url, { cache: 'no-store' });
  const buffer = await upstream.arrayBuffer();
  return new NextResponse(buffer, {
    headers: {
      'Content-Type': 'application/pdf',
      'Content-Disposition': 'inline',
    },
  });
}
```

```tsx
// PdfViewer.tsx — sin useEffect, sin blob URLs
export function PdfViewer({ url }: { url: string }) {
  const proxyUrl = `/api/pdf-proxy?url=${encodeURIComponent(url)}`;
  return (
    <object data={proxyUrl} type="application/pdf" className="w-full" style={{ height: '640px' }}>
      <a href={url} download>Descargar PDF</a>
    </object>
  );
}
```

**Usar `<object>` en lugar de `<iframe>` o `<embed>`** — mejor compatibilidad entre Chrome, Firefox y Edge, y permite fallback content dentro del tag.

**NUNCA intentar blob URL en el cliente para PDFs de Supabase** — falla por CORS o por timing de revocación.

### Estado inicial en componentes de adjunto

**Error:** Inicializar `preview` state con la URL del adjunto existente y luego renderizarla como `<img>` — si es un PDF, muestra imagen rota.

**Solución correcta:**
```tsx
// NO hacer esto:
const [preview, setPreview] = useState<string | null>(currentUrl);
// ...
{preview && <img src={preview} />}  // ← roto si preview es una URL de PDF

// SÍ hacer esto: separar la indicación de adjunto existente del preview de imagen
const hasAttachment = !!currentUrl;
// Solo mostrar <img> si se acaba de subir una imagen
const [imagePreview, setImagePreview] = useState<string | null>(null);
```

### React error #418 (hydration mismatch)

El error `Minified React error #418` con `args[]=text&args[]=` en producción es casi siempre causado por **extensiones del browser** (Grammarly, Google Translate, ad blockers) que modifican el DOM antes de que React hidrate.

**No es un bug del código.** Para confirmar: abrir la misma página en modo Incógnito sin extensiones — el error desaparece.

Causas reales de #418 en el código:
- `new Date().toLocaleDateString()` — locale diferente entre server (Node.js) y cliente (browser)
- `new Date()` en `defaultValue` de inputs — puede dar fecha diferente si se renderiza justo a medianoche
- Comentarios JSX `{/* */}` dentro de elementos flex no causan el error (son válidos)

**Solución para fechas:** usar formato fijo sin locale:
```ts
function formatDate(dateStr: string) {
  const [year, month, day] = dateStr.split('T')[0].split('-');
  return `${day}/${month}/${year}`;
}
```

---

## REGLA 13 — `useActionState`: el `initialState` debe ser `null`, no un objeto con campos opcionales

**Error que ocurre:**
```
Type error: No overload matches this call.
Argument of type '{ error: string; success: boolean; }' is not assignable to parameter of type
'{ error: string; success?: undefined; } | { success: boolean; error?: undefined; }'
```

**Causa:** Cuando una server action retorna un tipo union discriminado como `{ error: string } | { success: boolean }`, TypeScript infiere que los dos campos nunca coexisten. Un `initialState = { error: '', success: false }` tiene ambos campos simultáneamente, lo que no es compatible con ninguna de las ramas del union.

**Solución correcta — usar `null` como initialState:**
```tsx
// MAL:
const initialState = { error: '', success: false };
const [state, formAction, isPending] = useActionState(miAction, initialState);

// BIEN:
const [state, formAction, isPending] = useActionState(miAction, null);

// Y en el JSX usar optional chaining:
{state?.error && <p>{state.error}</p>}
{state?.success && <p>¡Éxito!</p>}
```

**Afecta a:** `forgot-password/page.tsx`, `reset-password/page.tsx`, `profile/page.tsx` — cualquier página que use `useActionState` con una action que retorna union discriminado.

**Regla:** Siempre usar `null` como `initialState` en `useActionState`. Nunca pasar un objeto con campos de distintas ramas del union type de la action.

---

## REGLA 14 — Middleware: el matcher debe excluir todos los archivos estáticos de `public/`

**Error que ocurre:**
```
Setting up fake worker failed: "Failed to fetch dynamically imported module: /pdf.worker.min.mjs"
The server responded with a non-JavaScript MIME type of "text/html".
```

**Causa:** El matcher del middleware captura rutas que coinciden con nombres de archivos estáticos (ej. `/pdf.worker.min.mjs`). La regex `^\/([^\/]+)` captura `"pdf.worker.min.mjs"` como `orgSlug`, no encuentra membership, y redirige a `/unauthorized` devolviendo HTML en lugar del archivo JS.

**Solución correcta — matcher en `src/middleware.ts`:**
```ts
export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp|mjs|js|css|map|woff|woff2|ttf|otf|ico)$).*)',
  ],
};
```

Extensiones que DEBEN estar excluidas: `svg`, `png`, `jpg`, `jpeg`, `gif`, `webp`, `mjs`, `js`, `css`, `map`, `woff`, `woff2`, `ttf`, `otf`, `ico`.

**Regla:** Cada vez que se agrega un archivo a `public/` con una extensión nueva (ej. `.wasm`, `.json`), agregar esa extensión al matcher del middleware.

---

## REGLA 15 — Zod: usar `.issues` en lugar de `.errors` sobre `ZodError`

**Error que ocurre:**
```
Type error: Property 'errors' does not exist on type 'ZodError<...>'
```

**Causa:** `ZodError` tiene `.issues` como propiedad tipada en TypeScript. `.errors` existe en runtime (es un alias) pero **no está en la definición de tipos** de Zod, por lo que TypeScript strict lo rechaza en el build.

**Solución correcta:**
```ts
// MAL:
if (!validated.success) return { error: validated.error.errors[0].message };

// BIEN:
if (!validated.success) return { error: validated.error.issues[0].message };
```

**Regla:** Siempre usar `.issues[0].message` (o `.flatten()`) para acceder a los errores de Zod. Nunca usar `.errors`.

---

## REGLA 16 — Nuevos módulos: checklist de archivos a crear/actualizar

Al crear un módulo nuevo (ej. `employees`, `fuel`), estos son los archivos que SIEMPRE hay que tocar:

### DB
1. `mcp__supabase__apply_migration` — crear tabla con RLS usando `get_user_org_ids()`
2. `mcp__supabase__generate_typescript_types` → reemplazar `src/types/supabase.ts`
3. `src/types/database.ts` — agregar interface y entrada en `Database.public.Tables`

### Feature
4. `src/features/<módulo>/actions.ts` — server actions con Zod (usar `.issues`, no `.errors`)
5. `src/features/<módulo>/components/` — componentes de lista y formulario

### Pages
6. `src/app/(org)/[orgSlug]/<módulo>/page.tsx` — página principal
7. `src/app/(org)/[orgSlug]/<módulo>/new/page.tsx` — si aplica
8. `src/app/(org)/[orgSlug]/<módulo>/[id]/edit/page.tsx` — si aplica

### Navegación
9. `src/components/layout/sidebar.tsx` — agregar entrada al nav

### Formularios
- Usar `useActionState(action, null)` — initialState siempre `null`
- Usar `state?.error` con optional chaining
- En actions: `validated.error.issues[0].message`

---

## REGLA 17 — Leaflet / Mapas: siempre usar altura en píxeles explícita

**Error que ocurre:** El mapa no se renderiza o aparece con altura 0.

**Causa:** Leaflet necesita que el contenedor del mapa tenga una altura concreta en píxeles en el momento del montaje. Usar `h-full` en un contenedor cuya altura depende del contenido (CSS grid/flex sin altura definida en el padre) crea una referencia circular: el mapa no tiene altura → el contenedor tampoco → Leaflet renderiza con 0px.

**Solución correcta:**
```tsx
// MAL — h-full sin padre con altura fija:
<TripMap className="h-full w-full" />

// BIEN — altura explícita en píxeles:
<TripMap className="h-[450px] w-full" />
```

**Regla:** Cualquier componente que use Leaflet (o cualquier biblioteca de mapas) debe tener su altura definida con un valor fijo en px o rem, nunca con `h-full` a menos que todos sus ancestros tengan alturas explícitas.

---

## REGLA 18 — PWA: manifest.json debe tener line endings LF (no CRLF)

**Error que ocurre:** El browser reporta error al parsear el manifest o lo ignora silenciosamente en producción.

**Causa:** En Windows, archivos creados con `echo` o editores con configuración CRLF quedan con `\r\n`. Algunos parsers de JSON en browsers móviles o en el build de Next.js fallan con CRLF en archivos de `public/`.

**Solución correcta:** Crear el archivo con `printf` en bash (que usa LF) o verificar que el archivo tenga LF antes de hacer commit:
```bash
file public/manifest.json  # debe decir "ASCII text" no "with CRLF line terminators"
```

**Regla:** Los archivos en `public/` (manifest.json, sw.js, etc.) deben tener LF. Configurar `.gitattributes` si es necesario para forzar LF en esos archivos.

---

## REGLA 19 — Documentación Obsidian: actualizar junto con el código

**Vault:** `C:\Users\Diegu\Obisdian-BlackdogAPP\Proyectos\Fleet SaaS\`

**Estructura:**
```
00 - Index.md          ← tabla de módulos + links a docs base
01 - Arquitectura.md   ← auth, middleware, rutas, zustand
02 - Base de Datos.md  ← tablas, enums, funciones PL/pgSQL
03 - Patrones.md       ← server actions, forms, errores, checklist
Módulos/
  Vehículos.md · Viajes.md · Mantenimiento.md · Combustible.md
  Finanzas.md · Inventario.md · Empleados.md · Contactos.md
  Terreno.md · Equipo.md · Ajustes.md
```

**Regla:** Cada vez que se crea o modifica un módulo, actualizar el archivo `.md` correspondiente en el vault de Obsidian. Si se agrega un módulo nuevo, crear su archivo en `Módulos/` y agregarlo al índice en `00 - Index.md`. Los links entre notas usan sintaxis `[[NombreNota]]` de Obsidian.

---
> Source: [dieguzzz/fleet-saas](https://github.com/dieguzzz/fleet-saas) — distributed by [TomeVault](https://tomevault.io/claim/dieguzzz).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
