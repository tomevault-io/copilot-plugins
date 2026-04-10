## tarea05-pauf-backend-peliculas

> Esta es una API REST de películas construida con Spring Boot 4.0.3, Java 25 y MongoDB. La arquitectura sigue el patrón MVC con servicios de negocio e integra **autenticación JWT con Spring Security**.

# AGENTS.md - Guía para Agentes de IA en Tarea5PAUF-PeliculasMongo

## Arquitectura General
Esta es una API REST de películas construida con Spring Boot 4.0.3, Java 25 y MongoDB. La arquitectura sigue el patrón MVC con servicios de negocio e integra **autenticación JWT con Spring Security**.

**Componentes principales:**
- **Modelos** (`models/`): Entidades de dominio con documentos embebidos (comentarios, valoraciones en películas; favoritas en usuarios) y referencias DBRef.
- **Repositorios** (`repositories/`): Interfaces que extienden `MongoRepository` para operaciones CRUD básicas.
- **Servicios** (`services/`): Lógica de negocio, conversiones DTO y manejo de relaciones embebidas.
- **Controladores** (`controllers/`): Endpoints REST con CORS habilitado globalmente.
- **DTOs** (`dtos/`): Objetos de transferencia para respuestas, evitando exponer entidades internas.
- **Seguridad** (`security/`): **NUEVO** - Componentes JWT (generación, validación, filtro).
- **Configuración** (`config/`): **NUEVO** - Configuración de Spring Security con JWT.

**Patrones de datos:**
- Documentos embebidos para listas (ej. `Pelicula.comentarios` como `List<Comentario>`).
- Referencias DBRef en embebidos (ej. `Comentario.autor` con `@DBRef` y `@JsonIdentityReference(alwaysAsId = true)`).
- Inicialización de listas vacías en servicios al guardar entidades nuevas.

## Convenciones de Código
- **Lombok obligatorio**: Todas las clases usan `@Setter`, `@Getter`, `@AllArgsConstructor`, `@NoArgsConstructor`, `@Builder`. Ver `Pelicula.java`.
- **Inyección de dependencias**: Usa `@Autowired` en servicios y controladores. Ejemplo en `PeliculaService.java`.
- **Manejo de fechas**: Servicios preservan fechas existentes en actualizaciones; asignan `LocalDate.now()` en creaciones. Ver `PeliculaService.save()`.
- **Conversiones DTO**: Servicios convierten entidades a DTOs para respuestas. Ejemplo: `PeliculaService.convertirDTO()` mapea listas embebidas a DTOs.
- **CORS**: Controladores anotados con `@CrossOrigin(origins = "*")` para desarrollo.
- **Seguridad**: **NUEVO** - Contraseñas cifradas con BCrypt. Tokens JWT de 24 horas. Rutas públicas: `/api/auth/login`, `/api/auth/register`. Resto requiere JWT válido en header `Authorization: Bearer <token>`.

## Flujos de Trabajo Críticos
- **Construcción**: `mvn clean install` para compilar; `mvn spring-boot:run` para ejecutar. Requiere Maven wrapper (`mvnw`).
- **Base de datos**: MongoDB en `localhost:27017`, colecciones "peliculas" y "usuarios". Configurado en `application.properties` con `spring.mongodb.uri`.
- **Pruebas**: Ejecutar con `mvn test`. Ver `ApiPaginaPeliculasApplicationTests.java` para pruebas básicas.
- **Debugging**: Usar logs de Spring; verificar serialización JSON con `@JsonIdentityInfo` comentado en modelos.
- **Autenticación**: **NUEVO** - Registrar usuario en `/api/auth/register`, obtener token en `/api/auth/login`, enviar token en header Authorization de requests posteriores.

## Puntos de Integración
- **Relaciones embebidas**: Agregar comentarios/valoraciones actualiza el documento padre. Ejemplo: `PeliculaService.addComentario()` busca usuario por ID y embebido en película.
- **Favoritas**: Simples objetos embebidos en `Usuario.favoritas`, sin referencias complejas. Ver `UsuarioService.addFavorita()`.
- **Serialización**: Evita ciclos con `@JsonIdentityReference` en referencias DBRef.
- **Autenticación**: **NUEVO** - `JwtTokenProvider` genera/valida tokens; `JwtAuthenticationFilter` extrae e inyecta en contexto de seguridad; `SecurityConfig` define rutas públicas/protegidas.

**Archivos clave para entender flujos:**
- `PeliculaService.java`: Lógica de embebidos y conversiones. **MODIFICADO** - Incluye conversión a ComentarioDTO/ValoracionDTO con autorNombre y método `findAllPaginated()`.
- `PeliculaController.java`: Endpoints REST y manejo de DTOs. **MODIFICADO** - `GET /api/peliculas` soporta paginación opcional (`page`, `size`, `sortBy`, `direction`) sin romper el comportamiento sin parámetros.
- `UsuarioService.java`: **MODIFICADO** - Incluye métodos `register()`, `authenticateUser()`, `emailExists()`, `getFavoritasPeliculas()`, `updateUsuario()` (cifra contraseña con BCrypt si se envía nueva).
- `UsuarioController.java`: **MODIFICADO** - `updateUsuario()` delega al servicio. Incluye endpoints `/me`, `/me/favoritas` y delete de favoritas.
- `AuthController.java`: **NUEVO** - Endpoints `/api/auth/login` y `/api/auth/register`.
- `application.properties`: Configuración MongoDB y JWT (`app.jwt.secret`, `app.jwt.expiration`).
- `pom.xml`: Dependencias (Lombok, Spring Data MongoDB, Spring Security, JJWT 0.12.3).

---

## IMPLEMENTACIÓN DE JWT (2026-04-01)

### Clases Creadas:
1. **`com.ricardo.peliculas.security.JwtTokenProvider`**
   - Genera tokens JWT con userId y expiración de 24h
   - Valida y extrae información de tokens
   - Usa JJWT 0.12.3 y SecretKey de 256 bits

2. **`com.ricardo.peliculas.security.JwtAuthenticationFilter`**
   - Filtro que se ejecuta en cada request
   - Extrae token del header `Authorization: Bearer <token>`
   - Inyecta autenticación en `SecurityContextHolder`

3. **`com.ricardo.peliculas.config.SecurityConfig`**
   - Configura Spring Security con política STATELESS
   - Define rutas públicas: `/api/auth/login`, `/api/auth/register`
   - Resto de rutas requieren autenticación JWT
   - Bean `PasswordEncoder` (BCryptPasswordEncoder)

4. **`com.ricardo.peliculas.controllers.AuthController`**
   - **NUEVO** - Controlador específico para autenticación
   - Endpoints: `POST /api/auth/register`, `POST /api/auth/login`
   - Responsable de registro y login públicos

5. **`com.ricardo.peliculas.dtos.LoginRequest`**
   - DTO con campos `email` y `password`

6. **`com.ricardo.peliculas.dtos.RegisterRequest`**
   - DTO con campos `nombre`, `email`, `password`

7. **`com.ricardo.peliculas.dtos.AuthResponse`**
   - DTO de respuesta con `token`, `id`, `nombre`, `email`, `fechaRegistro`

### Archivos Modificados:
1. **`pom.xml`**
   - Agregadas dependencias: `spring-boot-starter-security`, `jjwt-api`, `jjwt-impl`, `jjwt-jackson`, `spring-security-test`

2. **`UsuarioRepository.java`**
   - Agregado método `Optional<Usuario> findByEmail(String email)` para búsqueda en login/register

3. **`UsuarioService.java`**
   - Inyectado `PasswordEncoder` 
   - Agregado `Optional<Usuario> register(RegisterRequest)` - registra usuario con contraseña cifrada
   - Agregado `Optional<Usuario> authenticateUser(String email, String password)` - valida credenciales
   - Agregado `boolean emailExists(String email)` - verifica email duplicado

4. **`UsuarioController.java`**
   - **MODIFICADO** - Removidos endpoints de autenticación (movidos a AuthController)
   - Solo mantiene CRUD de usuarios y gestión de favoritas
   - Incluye nuevos endpoints para usuario autenticado: `/me`, `/me/favoritas`
   - **MODIFICADO** - updateUsuario() preserva campos no enviados

5. **`application.properties`**
   - Agregadas propiedades: `app.jwt.secret`, `app.jwt.expiration=86400000` (24h)

### Objetivo del Cambio:
Implementar autenticación y autorización con JWT manteniendo la estructura existente del proyecto. Permitir registro e inicio de sesión seguro con contraseñas cifradas en BCrypt. Proteger endpoints sensibles mediante tokens JWT sin romper endpoints existentes.

---

## HISTORIAL COMPLETO DE IMPLEMENTACIONES (2026-04-01)

### 1) Reubicación de autenticación en AuthController

#### Clases Creadas:
1. **`com.ricardo.peliculas.controllers.AuthController`**
   - Endpoint público `POST /api/auth/register`
   - Endpoint público `POST /api/auth/login`

#### Archivos Modificados:
1. **`UsuarioController.java`**
   - Removidos endpoints de auth que estaban bajo `/api/usuarios/...`
   - Mantenido solo CRUD de usuarios y favoritas

#### Objetivo del Cambio:
Alinear rutas públicas reales con `SecurityConfig` (`/api/auth/login` y `/api/auth/register`) para resolver acceso no público por path incorrecto.

---

### 2) Endpoints del usuario autenticado

#### Archivos Modificados:
1. **`UsuarioController.java`**
   - Agregado `GET /api/usuarios/me`
   - Agregado `GET /api/usuarios/me/favoritas`
   - Obtención de `userId` desde `SecurityContextHolder`

2. **`UsuarioService.java`**
   - Agregado método `getFavoritasPeliculas(String userId)`
   - Reutiliza `PeliculaService` para devolver `PeliculaValComDTO`

#### Objetivo del Cambio:
Permitir al frontend consultar datos y favoritas del usuario autenticado sin exponer contraseña ni requerir el id por URL.

---

### 3) Enriquecimiento de respuesta de películas con nombre de autor

#### Clases Creadas:
1. **`com.ricardo.peliculas.dtos.ComentarioDTO`**
   - Campos: `autorId`, `autorNombre`, `texto`, `fechaComentario`

2. **`com.ricardo.peliculas.dtos.ValoracionDTO`**
   - Campos: `autorId`, `autorNombre`, `puntuacion`, `fechaValoracion`

#### Archivos Modificados:
1. **`PeliculaValComDTO.java`**
   - Cambiado `comentarios` a `List<ComentarioDTO>`
   - Cambiado `valoraciones` a `List<ValoracionDTO>`

2. **`PeliculaService.java`**
   - `convertirDTO()` ahora incluye `autorNombre` para comentarios y valoraciones

#### Objetivo del Cambio:
Evitar llamadas extra desde React para resolver nombres de autores al mostrar comentarios y valoraciones.

---

### 4) Corrección de CORS para frontend React

#### Archivos Modificados:
1. **`SecurityConfig.java`**
   - Añadido bean `CorsConfigurationSource`
   - Origen permitido: `http://localhost:5174`
   - Métodos permitidos: `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`
   - Headers permitidos: `Authorization`, `Content-Type`, `Accept`, `X-Requested-With`
   - Integrado con `.cors(...)` en `SecurityFilterChain`

#### Objetivo del Cambio:
Resolver bloqueo de preflight y habilitar login/register desde React sin romper JWT ni rutas protegidas.

---

### 5) Actualización parcial segura de usuario (PUT)

#### Archivos Modificados:
1. **`UsuarioController.java`**
   - `updateUsuario()` delega al servicio

2. **`UsuarioService.java`**
   - Agregado `updateUsuario(String id, Usuario usuarioData)`
   - Preserva campos no enviados (`nombre`, `email`, `password`, más `favoritas/fechaRegistro` según lógica existente)
   - Si llega contraseña nueva, la cifra con `PasswordEncoder` (BCrypt)

#### Objetivo del Cambio:
Mantener comportamiento de actualización parcial sin pérdida de datos y evitando persistir contraseñas en texto plano.

---

### 6) Eliminación de favoritas por endpoint REST

#### Archivos Modificados:
1. **`UsuarioController.java`**
   - Agregado `DELETE /api/usuarios/{usuarioId}/favoritas/{peliculaId}`
   - Reutiliza `usuarioService.removeFavorita(...)`

#### Objetivo del Cambio:
Completar CRUD de favoritas con eliminación explícita mediante endpoint dedicado.

---

### 7) Paginación opcional en `GET /api/peliculas`

#### Archivos Modificados:
1. **`PeliculaController.java`**
   - `GET /api/peliculas` acepta params opcionales: `page`, `size`, `sortBy`, `direction`
   - Sin `page/size`: devuelve todo (comportamiento original)
   - Con `page/size`: devuelve contenido paginado + metadatos

2. **`PeliculaService.java`**
   - Agregado `findAllPaginated(int page, int size, String sortBy, String direction)`
   - Usa `PageRequest` y `Sort`

#### Objetivo del Cambio:
Agregar paginación/ordenación sin romper integraciones existentes que dependen de la lista completa sin parámetros.

---

### Resumen Global del Día
- Se mantuvo la estructura MVC y paquetes existentes.
- Se evitó romper endpoints previos; los cambios fueron incrementales y compatibles.
- Se centralizó la lógica sensible (cifrado/actualización) en servicios.
- Se reforzó integración frontend-backend (CORS, JWT, respuestas enriquecidas).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlegreCarcamusa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AlegreCarcamusa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
