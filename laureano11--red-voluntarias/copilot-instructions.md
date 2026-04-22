## red-voluntarias

> Contexto del proyecto Red Voluntarias - ONG de rescate animal


# Red Voluntarias - Contexto del Proyecto

Web institucional de una ONG dedicada al rescate, rehabilitación y adopción responsable de animales en situación de calle.

## Stack Tecnológico

- **Astro 5** — Framework para sitios estáticos
- **Tailwind CSS 4** — Estilos con `@tailwindcss/vite`
- **Sveltia CMS** — Panel admin para editar contenido sin código

## Estructura del Proyecto

```
src/
├── components/        # Componentes Astro (.astro)
├── content/
│   ├── dogs/          # JSON de perros en adopción (vía CMS)
│   ├── en-familia/    # Perros ya adoptados: solo nombre + foto (CMS)
│   ├── ventas/        # Productos solidarios (CMS)
│   ├── eventos/       # Eventos realizados con la comunidad (CMS)
│   ├── urgencias/     # Casos que necesitan ayuda urgente (CMS)
│   └── campaigns/     # JSON de campañas
├── config/            # Constantes (ej. donaciones: QR, MP, alias, formulario adopción)
├── data/              # Datos estáticos editables en código (ej. proyectos)
├── layouts/           # Layout base (Layout.astro)
├── pages/             # Páginas del sitio
├── styles/            # global.css con Tailwind
└── content.config.ts  # Esquemas Zod para colecciones
```

## Colecciones de Contenido

### Dogs (Perros)
```typescript
{
  name: string,
  sex: "Macho" | "Hembra",
  age: string,                    // Texto libre (meses / años)
  salud: {
    castrado: boolean,
    desparasitado: boolean,
    vacunado: boolean,
  },
  caracter: string,
  size: "Pequeño" | "Mediano" | "Grande" | "Sorpresa",
  observaciones: string,
  image: string,
  adopted: boolean,
  featured: boolean,
}
```
Catálogo en `/adopciones/adoptar`. URL del formulario de adopción: `src/config/adopcion.ts`.

### En familia (adoptados)
```typescript
{ name: string, image: string }
```
Página `/adopciones/en-familia`. Colección CMS independiente y simplificada.

**Orientación para adoptantes** — `/adopciones/orientacion`: contenido estático provisional (guía de cuidados); se irá definiendo con el tiempo.

**Donaciones** — `/como-ayudar/donaciones`: QR, link Mercado Pago, alias MP y alias cuenta; lista de insumos con emoji en `src/config/donaciones.ts`.

**Voluntariado** — `/como-ayudar/voluntariado`: texto fijo (qué hace el equipo, tareas); link al formulario en `src/config/voluntariado.ts`.

**Tránsito** — `/como-ayudar/transito`: texto fijo (hogar temporal); formulario en `src/config/transito.ts`.

**Hacerte socio** — `/como-ayudar/hacerte-socio`: aporte mensual recurrente; link en `src/config/socio.ts`.

**Ventas** — `/ventas`: catálogo CMS (`nombre`, `descripcion`, `precio`, `image`). Botón “Comprar” → Instagram en `src/config/ventas.ts`.

**Organizá con nosotros** — `/organiza/eventos` (CMS: eventos pasados); `/organiza/como-organizar` (guía fija de pasos, no CMS).

**Urgencia** — `/urgencia`: publicaciones CMS (`titulo`, `descripcion`, `image`, `activo`, `fecha` opcional). Solo se listan entradas con `activo: true`.

### Ventas (productos)
```typescript
{
  nombre: string,
  descripcion: string,
  precio: string,
  image: string,
}
```

### Eventos realizados
```typescript
{
  titulo: string,
  subtitulo: string,
  descripcion: string,
  fecha?: string,       // AAAA-MM-DD, orden en listado
  fotos: { src: string }[],
}
```

### Urgencias
```typescript
{
  titulo: string,
  descripcion: string,
  image: string,
  activo: boolean,      // false = ocultar sin borrar
  fecha?: string,       // AAAA-MM-DD, orden
}
```

### Campaigns (Campañas)
```typescript
{
  title: string,
  description: string,
  image: string,
  goal: number,      // Meta de donación
  raised: number,    // Monto recaudado
  active: boolean
}
```

## Paleta de Colores (CSS Variables)

| Variable | Color | Uso |
|----------|-------|-----|
| `--color-primary` | #2563eb | Azul principal, CTAs |
| `--color-primary-dark` | #1d4ed8 | Hover del primario |
| `--color-secondary` | #f59e0b | Amarillo/naranja acentos |
| `--color-accent` | #10b981 | Verde éxito, badges |
| `--color-surface` | #f8fafc | Fondo claro |

## Comandos

- `npm run dev` — Servidor en localhost:4321
- `npm run build` — Build estático en ./dist/
- CMS: entrar a `/admin` con `npm run dev` corriendo

## Convenciones

- Componentes en archivos `.astro` con Props tipadas via `interface Props`
- Contenido JSON en `src/content/` validado con Zod
- Clases Tailwind directamente en el markup
- Imágenes con `loading="lazy"` para performance
- Todo el contenido y UI en español

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Laureano11) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
