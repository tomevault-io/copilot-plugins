## nasacityplaner2025

> 1. [Visión General del Proyecto](#visión-general-del-proyecto)

# CLAUDE.md - Data Pathways Technical Documentation

## Tabla de Contenidos
1. [Visión General del Proyecto](#visión-general-del-proyecto)
2. [Arquitectura del Sistema](#arquitectura-del-sistema)
3. [Stack Tecnológico](#stack-tecnológico)
4. [Backend - Spring Boot](#backend---spring-boot)
5. [Frontend - React](#frontend---react)
6. [API REST Endpoints](#api-rest-endpoints)
7. [Integración con APIs Externas](#integración-con-apis-externas)
8. [Base de Datos](#base-de-datos)
9. [Configuración y Despliegue](#configuración-y-despliegue)
10. [Guía de Desarrollo](#guía-de-desarrollo)

---

## Visión General del Proyecto

**Data Pathways** es un sistema de análisis y visualización geoespacial diseñado para la Zona Metropolitana de Veracruz que combina datos demográficos, índices de desigualdad social y visualización cartográfica interactiva para apoyar decisiones de planificación urbana.

### Propósito Principal
Identificar oportunidades de expansión urbana con enfoque en:
- Reducción de desigualdad social (IISU - Índice de Inclusión Social Urbana)
- Mejora del acceso a servicios públicos (salud, educación, áreas verdes)
- Optimización de movilidad urbana

### Datos Clave de Veracruz
- **Población**: 939,046 habitantes (Censo INEGI 2020)
- **Índice de Desigualdad**: 4.0/5 (Alto) - Posición 20 de 74 zonas metropolitanas
- **Tiempo promedio al trabajo**: 28.3 minutos
- **Tiempo promedio a escuelas**: 17.6 minutos
- **Uso de transporte público**: 42.9%
- **Uso de automóvil privado**: 35.2%

---

## Arquitectura del Sistema

### Patrón de Arquitectura
El proyecto sigue una arquitectura de **3 capas** (Three-tier architecture):

```
┌─────────────────────────────────────────┐
│         CAPA DE PRESENTACIÓN            │
│    (React + Vite + Mapbox GL JS)       │
│         Puerto: 5173                    │
└────────────────┬────────────────────────┘
                 │ HTTP/REST
┌────────────────▼────────────────────────┐
│         CAPA DE APLICACIÓN              │
│       (Spring Boot 3.5.6)              │
│         Puerto: 8080                    │
│  Controllers → Services → Repositories  │
└────────────────┬────────────────────────┘
                 │ JDBC
┌────────────────▼────────────────────────┐
│         CAPA DE DATOS                   │
│    PostgreSQL (Supabase Cloud)         │
└─────────────────────────────────────────┘
```

### Componentes Principales

#### Backend (Spring Boot)
- **Controllers**: Exponen endpoints REST
- **Services**: Lógica de negocio y orquestación
- **Repositories**: Acceso a datos (JPA)
- **DTOs**: Objetos de transferencia de datos
- **Entities**: Modelos JPA mapeados a PostgreSQL
- **Clients**: Integración con APIs externas (WorldPop)

#### Frontend (React)
- **Components**: UI reutilizables (Dashboard, Map, Cards)
- **Hooks**: Lógica personalizada (useMapbox)
- **Context**: Estado global (MapContext)
- **Pages**: Vistas principales (LandingPage, Dashboard)

---

## Stack Tecnológico

### Backend
| Tecnología | Versión | Propósito |
|-----------|---------|-----------|
| Spring Boot | 3.5.6 | Framework principal |
| Java | 17 | Lenguaje de programación |
| Spring Data JPA | 3.5.6 | ORM y persistencia |
| PostgreSQL | 16+ | Base de datos relacional |
| Maven | 3.9+ | Gestión de dependencias |
| TwelveMonkeys ImageIO | 3.12.0 | Conversión TIFF → PNG |
| Jackson | 2.17+ | Procesamiento JSON |

### Frontend
| Tecnología | Versión | Propósito |
|-----------|---------|-----------|
| React | 19.1.1 (19.2.0 en root) | Framework UI |
| TypeScript | 5.8.3 | Tipado estático |
| Vite | 7.1.7 (7.1.9 en root) | Build tool y dev server |
| Tailwind CSS | 4.1.14 | Framework de estilos |
| Mapbox GL JS | 3.15.0 | Mapas interactivos |
| Lucide React | 0.544.0 | Iconos |

**Nota**: Las versiones entre paréntesis son las del package.json raíz (monorepo), usadas como meta-dependencies. Las versiones principales son las del front-city-planner/package.json.

---

## Backend - Spring Boot

### Estructura de Directorios

```
BackCityPlanner/
├── src/main/java/com/daffidev/backcityplanner/
│   ├── BackCityPlannerApplication.java
│   ├── controllers/
│   │   ├── MapController.java             # WorldPop endpoints
│   │   ├── ImageController.java           # Image URL endpoints
│   │   └── TestingController.java
│   ├── services/
│   │   ├── MapService.java                # Orchestration
│   │   ├── WorldPopClient.java            # External API client
│   │   ├── TiffConverter.java             # Image conversion
│   │   └── GraficoService.java            # DB operations
│   ├── repositories/
│   │   ├── GraficoRepository.java
│   │   └── IMapRepository.java
│   ├── entities/
│   │   └── Grafico.java                   # Population graphics
│   └── dto/
│       └── PopulationImageDto.java
└── src/main/resources/
    └── application.properties
```

### Componentes Clave

#### Controllers

**MapController.java** (`/api/worldpop/`)
- `GET /` - Obtiene datos de población por ISO3
- `GET /files` - Obtiene URLs de archivos WorldPop
- `GET /tiff/convert?url=` - Convierte TIFF a PNG desde URL
- `POST /tiff/upload` - Convierte TIFF a PNG desde archivo subido

**ImageController.java** (`/api/images`)
- `GET /map?iso3=MEX` - Obtiene URLs de imágenes de densidad poblacional

#### Services

**MapService.java** - Orquestación de lógica de negocio:
- `getPopulationByIso3(String iso3)` - Obtiene datos de WorldPop API
- `getWorldpopFilesByIso3(String iso3)` - Extrae URLs de archivos
- `downloadAndConvertTiffToPng(String tiffUrl)` - Descarga y convierte TIFF
- `getPopulationImages(String iso3)` - Lista de imágenes con URLs y años

**WorldPopClient.java** - Cliente HTTP para WorldPop API:
- Endpoints: Population data (`/WPGP`) y density (`/pd_ic_1km`)
- Timeouts: 5s conexión, 10s lectura
- Manejo de errores HTTP 4xx/5xx

**TiffConverter.java** - Conversión TIFF → PNG usando TwelveMonkeys ImageIO

**GraficoService.java** - Persistencia de gráficos en PostgreSQL

#### Entities

**Grafico.java** - Tabla `grafico`:
```java
@Entity
public class Grafico {
    @Id @GeneratedValue
    private Long id;
    private LocalDateTime createdAt;
    private String name;
    private Integer year;
    @Column(columnDefinition = "TEXT")
    private String url;
}
```

### Configuración

**application.properties**:
```properties
spring.application.name=BackCityPlanner
server.port=8080

# Database
spring.datasource.url=jdbc:postgresql://aws-1-us-east-2.pooler.supabase.com:5432/postgres
spring.datasource.username=postgres.kewcmyaivwpamlbekocv
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=update

# Logging
logging.pattern.console=WAKO_LOGS | %d{ISO8601} | %-5p | %-40.40c{1} | %m%n
```

**Nota**: `DB_PASSWORD` se carga desde archivo `.env` en la raíz.

---

## Frontend - React

### Estructura de Directorios

```
front-city-planner/src/
├── main.tsx
├── App.tsx
├── components/
│   ├── pages/
│   │   ├── LandingPage.tsx     # Página de inicio con mapa
│   │   └── Dashboard.tsx       # Dashboard con estadísticas
│   ├── layout/
│   │   └── Sidebar.tsx
│   ├── ui/
│   │   ├── StatsCard.tsx
│   │   ├── ChartCard.tsx
│   │   └── ComparisonCard.tsx
│   ├── MapboxMap.tsx           # Componente principal del mapa
│   ├── MapLeyenda.tsx          # Leyenda de capas
│   └── MapPopup.tsx
├── hooks/
│   └── useMapbox.ts            # Hook para Mapbox
└── context/
    └── MapContext.tsx          # Estado global del mapa
```

### Componentes Clave

#### MapContext.tsx
Estado global para la instancia del mapa Mapbox:
```typescript
interface MapContextType {
  map: Map | null;
  setMap: (map: Map | null) => void;
}
```

#### useMapbox.ts
Inicialización del mapa con:
- Estilo nocturno personalizado: `mapbox://styles/daffi/cmgb6w2zg000b01qp3e315yw8`
- Centro: Veracruz `[-96.11, 19.16]`
- Edificios 3D con extrusión
- Terreno 3D con DEM (exageración 1.5x)
- Controles de navegación y fullscreen

#### LandingPage.tsx
Capas disponibles:
1. **Índice de Desigualdad** (IISU por manzana)
2. **Información Vial** (red de carreteras)
3. **Referencias de Mejora**:
   - Consultorios (rojo)
   - Escuelas (azul)
   - Áreas abiertas (naranja)
   - Áreas verdes (verde)

#### Dashboard.tsx
Visualiza:
- Población: 939,046 habitantes
- Índice de desigualdad: 4.0/5
- Comparativa con otras zonas metropolitanas
- Tiempos de acceso a servicios
- Uso de transporte

### Configuración

**Variables de Entorno** (`.env`):
```env
VITE_MAPBOX_URL=pk.eyJ1IjoiZGFmZmkiLCJhIjoiY20...
```

**vite.config.ts**:
```typescript
export default defineConfig({
  plugins: [react()],
  server: { port: 5173, open: true }
});
```

---

## API REST Endpoints

**Base URL**: `http://localhost:8080` (desarrollo)

### WorldPop Endpoints

#### GET /api/worldpop/
Obtiene datos de población de WorldPop API.
- **Query**: `iso3` (string, optional, default: "MEX")
- **Response**: JSON con datos de población y URLs de archivos TIFF

#### GET /api/worldpop/files
Obtiene lista de URLs de archivos TIFF.
- **Query**: `iso3` (string, optional, default: "MEX")
- **Response**: Array de URLs

#### GET /api/worldpop/tiff/convert
Descarga TIFF desde URL y convierte a PNG.
- **Query**: `url` (string, required)
- **Response**: `image/png` (binario)

#### POST /api/worldpop/tiff/upload
Sube TIFF y convierte a PNG.
- **Body**: `multipart/form-data` con campo `file`
- **Response**: `image/png` (binario)

### Image Endpoints

#### GET /api/images/map
Obtiene URLs de imágenes de densidad poblacional con años.
- **Query**: `iso3` (string, optional, default: "MEX")
- **Response**: Array de `{popyear: number, url_img: string}`

### Códigos de Estado
- **200** OK - Solicitud exitosa
- **204** No Content - Sin datos
- **400** Bad Request - Parámetros inválidos
- **404** Not Found - Recurso no encontrado
- **500** Internal Server Error - Error en conversión

---

## Integración con APIs Externas

### WorldPop API
**Proveedor**: WorldPop - University of Southampton
**Propósito**: Datos de densidad poblacional global de alta resolución

**Endpoints Consumidos**:
1. `GET https://www.worldpop.org/rest/data/pop/WPGP?iso3={code}`
2. `GET https://www.worldpop.org/rest/data/pop_density/pd_ic_1km?iso3={code}`

**Configuración**:
- Timeouts: 5s conexión, 10s lectura
- Manejo de errores HTTP 4xx/5xx
- Parsing JSON con Jackson

### Mapbox API
**Proveedor**: Mapbox, Inc.
**Propósito**: Mapas vectoriales interactivos, geocodificación, routing

**Servicios Utilizados**:
- **Maps API (Mapbox GL JS 3.15.0)**
  - Vector Tiles
  - 3D Buildings (extrusión basada en altura)
  - 3D Terrain (DEM)
  - Estilo personalizado nocturno

- **Tilesets API**
  - IISU Veracruz (índice de desigualdad)
  - Red Vial 2021
  - Servicios Propuestos

**Configuración**:
```typescript
mapboxgl.accessToken = import.meta.env.VITE_MAPBOX_URL;
```

---

## Base de Datos

### PostgreSQL (Supabase)
- **Host**: `aws-1-us-east-2.pooler.supabase.com:5432`
- **Database**: `postgres`
- **Región**: AWS US-East-2

### Schema

**Tabla `grafico`**:
```sql
CREATE TABLE grafico (
    id BIGSERIAL PRIMARY KEY,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    name VARCHAR(255),
    year INTEGER,
    url TEXT
);

CREATE INDEX idx_grafico_year ON grafico(year);
CREATE INDEX idx_grafico_name ON grafico(name);
```

**Propósito**: Almacenar URLs de gráficos de población con metadata.

**Pool de Conexiones**:
- Default: 10 conexiones
- Max: 20 conexiones
- Timeout: 30s

---

## Configuración y Despliegue

### Requisitos del Sistema
- **Node.js**: 18.0.0+
- **Java**: JDK 17+
- **Maven**: 3.9+ (incluido Maven Wrapper)
- **PostgreSQL**: 16+ (o cuenta Supabase)

### Instalación Local

#### 1. Clonar y Configurar
```bash
git clone https://github.com/tu-usuario/NasaCityPlanner.git
cd NasaCityPlanner
```

#### 2. Variables de Entorno
Crear `.env` en la raíz:
```env
DB_PASSWORD=tu_contraseña_supabase
VITE_MAPBOX_URL=pk.eyJ1IjoiZGFmZmkiLCJhIjoiY20...
```

**Obtener credenciales**:
- **DB_PASSWORD**: Supabase Dashboard → Settings → Database
- **VITE_MAPBOX_URL**: Mapbox Dashboard → Tokens

**Nota**: El username de la base de datos (`postgres.kewcmyaivwpamlbekocv`) está hardcodeado en `application.properties`. Si necesitas cambiar el usuario, actualízalo directamente en ese archivo.

#### 3. Instalar e Iniciar
```bash
npm run install:all
npm run dev
```

Acceder a:
- Frontend: `http://localhost:5173`
- Backend API: `http://localhost:8080`

### Scripts Disponibles

| Script | Descripción |
|--------|-------------|
| `npm run dev` | Ejecuta backend + frontend en paralelo |
| `npm run dev:backend` | Solo backend (puerto 8080) |
| `npm run dev:frontend` | Solo frontend (puerto 5173) |
| `npm run build` | Build backend + frontend |
| `npm run build:backend` | Maven package (skip tests) |
| `npm run build:frontend` | Vite build |
| `npm run test:backend` | JUnit tests |
| `npm run clean` | Limpia build artifacts |

### Build para Producción

**Backend**:
```bash
npm run build:backend
java -jar BackCityPlanner/target/BackCityPlanner-0.0.1-SNAPSHOT.jar
```

**Frontend**:
```bash
npm run build:frontend
# Genera: front-city-planner/dist/
```

### Despliegue
- **Backend**: Heroku, Railway, AWS Elastic Beanstalk, Render
- **Frontend**: Vercel, Netlify, AWS S3 + CloudFront
- **Docker**: Dockerfiles disponibles en proyecto

Variables de entorno en producción:
```
DB_PASSWORD=<supabase_password>
VITE_MAPBOX_URL=<mapbox_token>
VITE_API_URL=<backend_url>
SPRING_PROFILES_ACTIVE=prod
```

---

## Guía de Desarrollo

### Convenciones de Código

**Java (Backend)**:
- Estilo: Google Java Style Guide
- Clases: `PascalCase`, Métodos: `camelCase`, Constantes: `UPPER_SNAKE_CASE`
- Packages: `controllers`, `services`, `repositories`, `entities`, `dto`

**TypeScript (Frontend)**:
- Estilo: Airbnb React/JSX Style Guide
- Componentes: `PascalCase`, Funciones: `camelCase`, Constantes: `UPPER_SNAKE_CASE`

### Agregar Nuevos Endpoints

1. Crear Controller con `@RestController` y `@RequestMapping`
2. Crear Service con `@Service` y lógica de negocio
3. Documentar en CLAUDE.md sección "API REST Endpoints"

### Agregar Capas al Mapa

1. Crear Tileset en Mapbox Studio (subir GeoJSON/Shapefile)
2. Agregar source y layer en `useMapbox.ts` dentro de `map.on('load')`
3. Agregar toggle en `MapLeyenda.tsx` con `useState` y `useEffect`

### Testing

**Backend (JUnit)**:
```bash
npm run test:backend
```

**Frontend**: Vitest no configurado aún

### Debugging

**Backend**: IntelliJ IDEA o VS Code con extensión Java
- Main class: `BackCityPlannerApplication`
- Env vars: `DB_PASSWORD=...`

**Frontend**: Chrome DevTools + React DevTools extension

---

## Decisiones Técnicas

### Stack Principal
1. **Spring Boot 3.5.6**: Ecosistema maduro, Spring Data JPA, DevTools, amplia documentación
2. **React 19 + TypeScript**: Component-based, Virtual DOM, ecosistema robusto
3. **Mapbox GL JS**: Vectorial de alta performance, soporte 3D nativo, estilos personalizables
4. **PostgreSQL (Supabase)**: Relacional robusta, hosting gratuito, PostGIS disponible
5. **TwelveMonkeys ImageIO**: Soporte TIFF nativo en Java puro
6. **Vite**: Build instantáneo, HMR ultra-rápido, bundle optimizado
7. **Tailwind CSS v4**: Utility-first, tree-shaking automático, responsive built-in

### Arquitectura
- **3 capas tradicional**: Proyecto de tamaño medio, patrón familiar, fácil de entender
- **Monorepo**: Desarrollo local simplificado, versionado sincronizado
- **Context API**: Estado global simple (solo instancia del mapa)

---

## Fuentes de Datos

1. **INEGI** - Censo de Población y Vivienda 2020: Datos demográficos de Veracruz
2. **WRI México** - IISU 2021: Índice de Inclusión Social Urbana
3. **SCT** - Red Nacional de Caminos 2021: Shapefile de carreteras
4. **WorldPop** - University of Southampton: Densidad poblacional 1km × 1km
5. **Mapbox**: Mapas base vectoriales, DEM, edificios 3D

---

**Última actualización**: 2025-10-09
**Versión**: 1.1.1 (versiones de dependencias actualizadas, aclaraciones sobre configuración)

---
> Source: [RickCabrera/NasaCityPlaner2025](https://github.com/RickCabrera/NasaCityPlaner2025) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
