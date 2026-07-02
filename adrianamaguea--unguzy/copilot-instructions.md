## unguzy

> El branding completo está en `assets/branding.png`. Úsalo como referencia visual

# UNGUZY — E-commerce de Skincare

## Identidad de marca

El branding completo está en `assets/branding.png`. Úsalo como referencia visual
permanente antes de tomar cualquier decisión de diseño.

Marca: **UNGUZY** — skincare natural con estética orgánica, cálida y editorial.
Tagline: *"Balance Natural Para Tu Piel"*

---

## Estilo visual de la página

La página debe sentirse como una publicación editorial de lujo accesible —
no una tienda genérica de skincare. Las referencias visuales son una revista
de moda slow, un mercado orgánico de autor y una botica artesanal.

Características que deben percibirse en cada sección:

- **Calidez material**: fondos en beige cálido, sombras suaves, sin blancos fríos
- **Respiración**: mucho espacio en blanco, layouts que no saturan
- **Contraste editorial**: títulos grandes en Playfair Display contra cuerpos
  pequeños en Neue Haas, creando jerarquía sin necesidad de color
- **Naturaleza contenida**: el ícono floral lineal del logo puede usarse como
  elemento decorativo recurrente y sutil entre secciones
- **Fotografía integrada**: las imágenes de productos y modelos deben tratarse
  con consistencia tonal — cálidas, con luz natural, nunca con fondos de
  estudio fríos si se pueden evitar
- **Movimiento discreto**: las animaciones deben ser lentas y orgánicas,
  nunca abruptas — entradas con fade + ligero desplazamiento vertical,
  hover states suaves en productos

---

## Paleta de colores

Define estas variables CSS globales y úsalas de forma consistente en todo
el proyecto. No uses colores fuera de esta paleta.

```css
:root {
  --color-raiz:   #5B2A18; /* marrón oscuro — fondos hero, textos principales */
  --color-terra:  #A24B2A; /* terracota — acentos, CTAs, hover states */
  --color-papel:  #E7DED1; /* beige claro — fondos secundarios, cards */
  --color-ambar:  #9B7A45; /* dorado apagado — bordes, detalles, íconos */
  --color-blanco: #FDFAF6; /* blanco cálido — fondo principal de página */
}
```

---

## Tipografía

- **Fuente de marca** (display principal): archivo `.otf` ubicado en
  `public/fonts/` — declárala con `@font-face` en `globals.css` y úsala
  exclusivamente para el logotipo, hero headlines y nombre de la marca
- **Playfair Display** — títulos de sección, nombres de productos, headings
- **Neue Haas Grotesk** (fallback: `DM Sans`) — cuerpo, navegación,
  etiquetas, precios, descripciones

```css
/* En app/globals.css */
@font-face {
  font-family: 'UnguzyFont';
  src: url('/fonts/[nombre-exacto-del-archivo].otf') format('opentype');
  font-weight: normal;
  font-style: normal;
  font-display: swap;
}

:root {
  --font-brand:   'UnguzyFont', serif;
  --font-display: 'Playfair Display', serif;
  --font-body:    'Neue Haas Grotesk', 'DM Sans', sans-serif;
}
```

> Nota: si el rendimiento de carga es prioritario, convierte el `.otf` a
> `.woff2` con Transfonter (transfonter.org) y actualiza el `src`.

---

## Stack técnico

- **Framework**: Next.js (App Router) + React
- **Estilos**: CSS puro con variables CSS — sin Tailwind, sin CSS-in-JS
- **Imágenes**: `next/image` con optimización activada
- **Assets**: todas las imágenes en `assets/`

---

## Assets disponibles

```
assets/
  branding.png                  → referencia de identidad de marca (no usar en UI)
  productos.png                 → foto grupal de línea completa — usar como hero
  hero_image.png                → imagen alternativa para hero
  logo.png                      → logotipo oficial de la marca

  /* Productos — 6 en total */
  jabon_facial.png              → Jabón Facial de Carbón Activado, Aloe Vera y Ácido Salicílico
  hidratante.png                → imagen principal: Crema Hidratante
  crema_hidratante.png          → imagen secundaria: Crema Hidratante (carrusel)
  serum_colageno.png            → Sérum de Colágeno y Ácido Hialurónico
  serum_vitaminac.png           → Sérum de Vitamina C, Extracto de Arroz, Elastina y Colágeno
  tonico_manzanilla.png         → Tónico de Manzanilla y Ácido Hialurónico
  tonico_hamamelis.png          → Tónico de Hamamelis y Ácido Salicílico

  /* Modelos */
  modelo1.PNG                   → foto de modelo — usar en hero o sección Nosotros
  modelo2.PNG                   → foto de modelo — usar en hero o sección Nosotros
```

**Uso del logo**: usa `logo.png` en el header/navbar y footer.
No uses el texto "UNGUZY" en tipografía genérica donde debería ir el logo.

**Carrusel de detalle**: solo la Crema Hidratante tiene dos imágenes (`hidratante.PNG`
como principal, `Crema hidratante.PNG` como secundaria). Los demás productos muestran
imagen estática — no renderices carrusel si hay una sola imagen.

**Uso de fotos de modelos**: úsalas en el hero, en la sección "Nosotros" y
como contexto visual en páginas de producto. Prioriza imágenes que muestren
el producto en uso sobre las que solo muestren la persona.

**Edición de imágenes**: si alguna imagen de producto o modelo no es
consistente en tono o recorte con el resto, ajústala (crop, temperatura
de color, contraste) para mantener coherencia visual en todo el catálogo.

---

## Estructura de rutas

```
/               → Home: hero + productos destacados + fragmento de "Nosotros"
/productos      → Catálogo completo (6 productos)
/nosotros       → Historia de la marca + valores + ingredientes clave
/contactanos    → Información de contacto + botón de WhatsApp prominente
```

---

## Productos

Son **6 productos**. Cada página de detalle debe mostrar: nombre, tipo,
descripción corta, activos destacados, beneficios, tipo de piel y modo de uso.
En el catálogo (/productos) muestra solo: nombre, activos destacados, descripción
corta y botón de compra.

Precios confirmados por la marca:

| Producto | Precio |
|---|---|
| Jabón Facial | $47.000 |
| Crema Hidratante | $52.000 |
| Tónico de Manzanilla | $47.000 |
| Tónico de Hamamelis | $47.000 |
| Sérum de Vitamina C | $51.000 |
| Sérum de Colágeno | $43.000 |

Botón de compra — patrón de URL para todos los productos:
```
https://wa.me/[NUMERO]?text=Hola,%20quiero%20pedir%20[nombre-del-producto]
```

**No hay carrito de compras ni pasarela de pago. El único canal de compra es WhatsApp.**

---

### 1. Jabón Facial de Carbón Activado, Aloe Vera y Ácido Salicílico
- **Imagen**: `jabon_facial.png`
- **Tipo**: Limpiador facial de limpieza profunda
- **Descripción corta**: Sistema limpiador purificante diseñado para piel grasa y con tendencia acneica.
- **Activos destacados**: Carbón activado, Aloe vera, Ácido salicílico, Vitamina E
- **Beneficios**:
  - Elimina impurezas, exceso de grasa y residuos ambientales
  - Contribuye a disminuir puntos negros y brotes de acné
  - Limpia profundamente sin generar resequedad extrema
  - Refresca y ayuda a equilibrar la piel
- **Tipo de piel**: Mixta, grasa y con tendencia acneica
- **Modo de uso**: Aplicar sobre el rostro húmedo con masajes circulares y retirar con abundante agua. Usar día y noche.

---

### 2. Crema Hidratante de Colágeno, Elastina, Vitamina E y Niacinamida
- **Imagen principal**: `hidratante.png`
- **Imagen secundaria (carrusel)**: `crema_hidratante.png`
- **Tipo**: Crema facial hidratante y revitalizante
- **Descripción corta**: Emulsión hidratante facial desarrollada para mejorar hidratación, elasticidad y apariencia de la piel.
- **Activos destacados**: Colágeno hidrolizado, Elastina hidrolizada, Vitamina E, Niacinamida, Glicerina
- **Beneficios**:
  - Aporta hidratación profunda y sensación de suavidad
  - Ayuda a mejorar la elasticidad y apariencia de la piel
  - Contribuye a fortalecer la barrera cutánea
  - Favorece una apariencia luminosa y uniforme
- **Tipo de piel**: Todo tipo de piel, especialmente piel seca o con signos de envejecimiento
- **Modo de uso**: Aplicar sobre el rostro limpio con suaves movimientos ascendentes hasta su absorción.

---

### 3. Tónico de Manzanilla y Ácido Hialurónico
- **Imagen**: `tonico_manzanilla.png`
- **Tipo**: Tónico facial calmante e hidratante
- **Descripción corta**: Tónico calmante e hidratante para piel sensible o irritada.
- **Activos destacados**: Extracto de manzanilla, Ácido hialurónico, Glicerina, Extractos botánicos
- **Beneficios**:
  - Ayuda a calmar la piel sensible y reducir el enrojecimiento
  - Brinda hidratación ligera y sensación refrescante
  - Prepara la piel para potenciar la absorción de otros productos
- **Tipo de piel**: Sensible, seca o irritada
- **Modo de uso**: Aplicar después de la limpieza facial con algodón o directamente sobre el rostro.

---

### 4. Tónico de Hamamelis y Ácido Salicílico
- **Imagen**: `tonico_hamamelis.png`

- **Tipo**: Tónico facial purificante
- **Descripción corta**: Tónico facial purificante e hidratante enfocado en balancear piel grasa.
- **Activos destacados**: Hamamelis, Ácido salicílico, Aloe vera, Extracto de sauce
- **Beneficios**:
  - Ayuda a controlar el exceso de grasa
  - Contribuye a minimizar visualmente los poros
  - Refresca y mejora la textura de la piel
- **Tipo de piel**: Grasa y con tendencia acneica
- **Modo de uso**: Aplicar sobre la piel limpia evitando el contorno de ojos.

---

### 5. Sérum de Vitamina C, Extracto de Arroz, Elastina y Colágeno
- **Imagen**: `serum_vitaminac.png`
- **Tipo**: Sérum facial iluminador y rejuvenecedor
- **Descripción corta**: Sérum antioxidante e iluminador enfocado en rejuvenecimiento facial.
- **Activos destacados**: Vitamina C, Extracto de arroz, Elastina, Colágeno
- **Beneficios**:
  - Ayuda a iluminar y unificar el tono de la piel
  - Contribuye a mejorar la apariencia de líneas finas
  - Favorece una apariencia más firme y luminosa
- **Tipo de piel**: Todo tipo de piel
- **Modo de uso**: Aplicar de 3 a 5 gotas sobre el rostro limpio antes de la crema hidratante.

---

### 6. Sérum de Colágeno y Ácido Hialurónico
- **Imagen**: `serum_colageno.png`
- **Tipo**: Sérum facial hidratante y reafirmante
- **Descripción corta**: Sérum hidratante y reafirmante de rápida absorción.
- **Activos destacados**: Colágeno, Ácido hialurónico, Glicerina, Activos humectantes
- **Beneficios**:
  - Aporta hidratación intensiva
  - Ayuda a mejorar la elasticidad y firmeza de la piel
  - Contribuye a prevenir signos visibles de envejecimiento
- **Tipo de piel**: Piel seca, madura o deshidratada
- **Modo de uso**: Aplicar mañana y noche sobre el rostro limpio.

---

## Skills a usar

- **Taste skill** → todas las decisiones de diseño visual: spacing,
  jerarquía tipográfica, composición de secciones, selección y tratamiento
  de imágenes
- **Emil skill** → animaciones: entradas de elementos al hacer scroll,
  hover states en productos y botones, transiciones entre páginas

---

## Criterios de calidad

- La estética debe sentirse editorial y de lujo accesible — nunca genérica
- Los colores deben coincidir exactamente con la paleta definida
- Mobile-first: diseño funcional y elegante desde 375px
- Consistencia tonal entre todas las imágenes del catálogo
- El botón de WhatsApp debe ser visible y claro en cada producto
- La fuente de marca debe cargar correctamente — usarla solo donde corresponde
- Sin errores de consola en `next dev`
- Todas las rutas deben funcionar correctamente

---

## Lo que NO debes hacer

- No uses Tailwind
- No uses colores fuera de la paleta definida
- No agregues carrito, checkout ni pasarela de pago
- No uses la fuente de marca para cuerpo de texto — solo para display
- No mezcles estilos inline con CSS modular — elige uno y sé consistente
- No uses fondos blancos puros (`#FFFFFF`) — siempre usa `--color-blanco`
- No uses animaciones rápidas o bruscas — el ritmo visual de la marca es lento y orgánico
- No reemplaces el logo con texto tipografiado en el navbar

---
> Source: [AdrianaMaguea/unguzy](https://github.com/AdrianaMaguea/unguzy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
