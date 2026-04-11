## mytrip-guru

> > Tu asistente de viajes inteligente que escanea tu entorno y te muestra todo lo que necesitas al alcance de un tap.

# MyTrip Guru 🧭

> Tu asistente de viajes inteligente que escanea tu entorno y te muestra todo lo que necesitas al alcance de un tap.

---

## 🎯 North Star
Crear la app de viajes definitiva que elimine la necesidad de saltar entre múltiples aplicaciones, reuniendo información de lugares cercanos con reseñas, distancias y direcciones en una experiencia visual premium.

## 👤 Usuario Target
**Viajeros de negocios y placer** que necesitan orientarse rápidamente al llegar a un nuevo destino. Su problema: actualmente usan 4-5 apps diferentes para encontrar restaurantes, transporte, farmacias, y direcciones. MyTrip Guru lo unifica todo.

**Persona Principal:**
- Edad: 25-55 años
- Viaja frecuentemente (3+ veces al año)
- Usa smartphone como herramienta principal
- Valora la eficiencia y el diseño atractivo

---

## ✅ MVP Features (Prioridad 1)

### 🔐 Autenticación
- [ ] Login con Google (OAuth)
- [ ] Login con Email + Password
- [ ] Persistencia de sesión
- [ ] Perfil básico de usuario

### 🏠 Registro de Hospedaje
- [ ] Input de dirección del alojamiento
- [ ] Autocompletado con Google Places
- [ ] Geocodificación (dirección → coordenadas)
- [ ] Guardar en historial de viajes

### 🔍 Escaneo de Alrededores (Feature Principal)
- [ ] **Efecto visual de escaneo** - animación tipo radar/sonar premium
- [ ] Radio de búsqueda: 1km desde el hospedaje
- [ ] Categorías a escanear:
  - 🍽️ Restaurantes
  - 🚌 Transporte (paradas de bus, metro, taxis)
  - 💊 Farmacias
  - 🛒 Supermercados
  - 🍺 Bares
  - 🏖️ Playas (si aplica a la ubicación)
- [ ] Integración con Google Places API
- [ ] Carga progresiva de resultados durante el escaneo

### 📋 Listado de Lugares
- [ ] Cards visuales premium por cada lugar
- [ ] Iconografía distintiva por categoría
- [ ] Información mostrada:
  - Nombre del lugar
  - Categoría (icono)
  - Distancia exacta (ej: "350m")
  - Rating de Google (estrellas)
  - Estado (abierto/cerrado)
  - Foto del lugar
- [ ] Filtro por categoría
- [ ] Ordenar por distancia/rating

### ⭐ Reseñas Integradas
- [ ] Ver reseñas de Google Places SIN salir de la app
- [ ] Mostrar: autor, rating, fecha, texto
- [ ] Scroll infinito para más reseñas
- [ ] Foto del reviewer

### 🗺️ Navegación
- [ ] Botón "Cómo llegar" en cada lugar
- [ ] Mostrar ruta paso a paso
- [ ] Integración con Google Maps Directions API
- [ ] Opción de abrir en Google Maps/Apple Maps nativo

### 📜 Historial de Viajes
- [ ] Lista de viajes anteriores
- [ ] Nombre personalizable (ej: "Barcelona 2024")
- [ ] Fecha del viaje
- [ ] Acceso rápido a resultados guardados

---

## 🚫 Out of Scope (v1)
- Reservas directas (hoteles, restaurantes)
- Pagos dentro de la app
- Modo offline completo
- Chat con otros viajeros
- Itinerarios automáticos
- Integración con aerolíneas
- Múltiples idiomas (solo español v1)

---

## 🔧 Stack Técnico

### Frontend (Mobile)
- **Framework**: React Native + Expo
- **Navegación**: Expo Router
- **Estado**: Zustand
- **Estilos**: NativeWind (Tailwind para RN)
- **Animaciones**: React Native Reanimated + Lottie
- **Mapas**: React Native Maps

### Backend / BaaS
- **Base de Datos**: Supabase (PostgreSQL)
- **Autenticación**: Supabase Auth (Google + Email)
- **Storage**: Supabase Storage (fotos de perfil)

### APIs Externas
- **Google Places API** - Búsqueda de lugares, fotos, reseñas
- **Google Geocoding API** - Convertir direcciones a coordenadas
- **Google Directions API** - Rutas y navegación

### DevOps
- **Builds**: EAS Build (Expo Application Services)
- **Updates OTA**: EAS Update
- **CI/CD**: GitHub Actions

---

## 📊 Métricas de Éxito

1. **Engagement**: Usuarios completan el escaneo en <10 segundos
2. **Retención**: 40%+ usuarios regresan en 7 días
3. **Reviews**: Rating 4.5+ en App Store / Play Store
4. **NPS**: "¿Recomendarías esta app a otro viajero?" > 50

---

## 🔗 Dependencias Externas

| Servicio | Propósito | Requiere API Key |
|----------|-----------|------------------|
| **Google Places API** | Búsqueda de POIs, fotos, reseñas | ✅ Sí |
| **Google Geocoding API** | Dirección → Coordenadas | ✅ Sí |
| **Google Directions API** | Rutas y navegación | ✅ Sí |
| **Supabase** | Auth + DB + Storage | ✅ Sí (URL + Keys) |
| **Expo / EAS** | Builds y updates | ✅ Sí (cuenta Expo) |

---

## 🎨 Directrices de Diseño

### Estética
- **Dark mode** como default (premium, moderno)
- **Paleta**: Negro profundo + Acentos cyan/violeta
- **Glassmorphism** en cards
- **Animaciones suaves** estilo Apple

### Experiencia de Escaneo
El efecto de escaneo DEBE ser el diferenciador visual:
- Animación tipo radar/sonar expandiéndose
- Puntos apareciendo en tiempo real
- Feedback háptico sutil
- Transición fluida al listado

### Tarjetas de Lugares
- Foto de fondo con overlay gradiente
- Información clara y legible
- Iconografía consistente (Lucide icons)
- Hover/tap states bien definidos

---

## 📁 Estructura de Proyecto (Propuesta)

```
mytrip-guru/
├── app/                     # Expo Router pages
│   ├── (auth)/              # Pantallas de auth
│   │   ├── login.tsx
│   │   └── register.tsx
│   ├── (tabs)/              # Main app (tab navigator)
│   │   ├── index.tsx        # Home - Escaneo
│   │   ├── places.tsx       # Listado de lugares
│   │   ├── trips.tsx        # Historial de viajes
│   │   └── profile.tsx      # Perfil de usuario
│   ├── place/[id].tsx       # Detalle de lugar
│   └── _layout.tsx          # Root layout
├── components/
│   ├── ui/                  # Componentes base
│   ├── places/              # Cards, listas de lugares
│   ├── scan/                # Animación de escaneo
│   └── maps/                # Componentes de mapa
├── lib/
│   ├── supabase.ts          # Cliente Supabase
│   ├── google-places.ts     # API de Google Places
│   └── utils.ts             # Utilidades
├── stores/
│   ├── auth.store.ts        # Estado de auth
│   ├── places.store.ts      # Estado de lugares
│   └── trips.store.ts       # Estado de viajes
├── types/
│   └── index.ts             # Tipos TypeScript
└── assets/                  # Imágenes, Lottie, etc.
```

---

## ⏱️ Estimación de Desarrollo

| Fase | Tiempo Estimado |
|------|-----------------|
| Setup + Auth | 1-2 días |
| Escaneo + Places API | 2-3 días |
| UI de listado + cards | 2 días |
| Detalle + reseñas | 1-2 días |
| Navegación/direcciones | 1-2 días |
| Historial de viajes | 1 día |
| Polish + animaciones | 2 días |
| **Total MVP** | **10-14 días** |

---

## ✅ Blueprint Status: COMPLETADO

**Próximo paso**: Activar Skill L (Conector) para validar APIs y credenciales.

---

## 🏗️ Progreso de Implementación

### Skill A (Arquitecto) - ✅ COMPLETADO
- [x] Estructura de 3 capas implementada
- [x] Expo Router configurado con tabs y rutas dinámicas
- [x] TypeScript estricto sin errores
- [x] Servicios: Supabase + Google Places API
- [x] Hooks: useAuth, useLocation
- [x] Utils: formatting, validation (Result pattern)
- [x] Tipos centralizados en `/src/lib/types`

### Skill S (Artista) - ✅ COMPLETADO
- [x] Sistema de Design Tokens (colores, tipografía, sombras)
- [x] Componentes UI premium:
  - AnimatedCard (glassmorphism + press feedback)
  - AnimatedButton (gradientes + haptic)
  - FadeInView (múltiples animaciones de entrada)
  - IconButton (variantes: filled, tinted, ghost)
  - Badge (success, warning, error, etc.)
  - ShimmerPlaceholder (loading skeleton)
- [x] Pantallas rediseñadas con animaciones:
  - Home (Mis Viajes) - estado vacío premium
  - Scan Tab - pulso animado, glow effects
- [x] Logo de app generado y configurado
- [x] Dark/Light theme automático

### Pantallas Implementadas
| Pantalla | Estado | Animaciones |
|----------|--------|-------------|
| Mis Viajes | ✅ | FadeIn, SlideIn, FloatingIcons |
| Explorar | ✅ | Categorías animadas |
| Escanear (Tab) | ✅ | Pulse, Glow, SlideUp |
| Escanear (Modal) | ✅ | Viewfinder con frame |
| Mapa | 🔲 Placeholder | - |
| Perfil | ✅ | Gradiente header |
| Login | ✅ | KeyboardAvoidingView |
| Register | ✅ | Formulario completo |
| Trip Detail | ✅ | Gradient card |
| Place Detail | ✅ | Overlay, distancia |

### Próximos Pasos
1. [ ] Crear tablas en Supabase (trips, places, users)
2. [ ] Integrar Google Maps en pantalla de Mapa
3. [ ] Implementar lógica de escaneo por radar (Location-based)
4. [ ] Conectar Results de Google Places al radar
5. [ ] Persistencia de preferencias de usuario

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nabole05)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nabole05)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
