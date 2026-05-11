## pointofsale

> Sistema de Punto de Venta (POS) para pequenas y medianas empresas.

# CLAUDE.md - Directivas del Proyecto PuntoDeVenta

## Descripcion del Proyecto
Sistema de Punto de Venta (POS) para pequenas y medianas empresas.
Arquitectura multicapa con cliente WPF y API REST.

## Stack Tecnologico Actual
- **Framework**: .NET 8
- **UI Desktop**: WPF (Windows Presentation Foundation) + XAML
- **UI Web**: Blazor WebAssembly + nginx
- **API REST**: ASP.NET Core 8 Web API
- **ORM**: Entity Framework Core 8
- **Base de datos**: PostgreSQL
- **Autenticacion**: JWT Bearer Tokens
- **PDF**: QuestPDF / iTextSharp 5.5.13
- **Graficos**: LiveCharts 0.9.7, Blazor-ApexCharts (Web)
- **Deploy**: Railway (API + Web como servicios separados)
- **Lenguaje**: C#

## Estructura del Proyecto

```
PuntoDeVenta/
├── Capa Entidad/              # Modelos de dominio (CE_*)
│   ├── CE_Usuarios.cs
│   ├── CE_Productos.cs
│   ├── CE_Clientes.cs
│   ├── CE_Grupos.cs
│   └── CE_Movimiento.cs
│
├── Capa Datos/                # Acceso a datos
│   ├── Context/
│   │   └── ApplicationDbContext.cs   # EF Core DbContext
│   ├── Interfaces/            # Contratos
│   │   ├── IRepository.cs
│   │   ├── IUsuarioRepository.cs
│   │   ├── IProductoRepository.cs
│   │   └── IUnitOfWork.cs
│   ├── Repositories/          # Implementaciones
│   │   ├── Repository.cs      # Generico
│   │   ├── UsuarioRepository.cs
│   │   ├── ProductoRepository.cs
│   │   └── UnitOfWork.cs
│   ├── CD_*.cs                # Legacy (ADO.NET)
│   └── ConfigurationHelper.cs # Helper de configuracion
│
├── Capa Negocio/              # Logica de negocio (CN_*)
│
├── PuntoDeVenta/              # Cliente WPF
│   ├── Views/                 # Vistas XAML
│   ├── src/
│   │   ├── Boxes/             # Dialogos modales
│   │   ├── Themes/            # Temas (Dark, Green, Red)
│   │   └── img/               # Imagenes
│   └── Resources/             # Templates PDF
│
├── PuntoDeVenta.API/          # API REST
│   ├── Controllers/           # Endpoints
│   │   ├── AuthController.cs
│   │   ├── ProductosController.cs
│   │   ├── UsuariosController.cs
│   │   ├── ClientesController.cs
│   │   ├── GruposController.cs
│   │   └── MovimientosController.cs
│   ├── DTOs/                  # Data Transfer Objects
│   ├── Auth/
│   │   └── JwtService.cs      # Generacion de tokens
│   └── Program.cs             # Configuracion de la API
│
├── docs/                      # Documentacion tecnica
│   ├── Fase1_TecnologiasNuevas.md
│   ├── Fase2_TecnologiasNuevas.md
│   └── Fase3_TecnologiasNuevas.md
│
├── Query Inicial/             # Scripts SQL PostgreSQL
├── appsettings.json           # Configuracion (NO commitear)
├── appsettings.example.json   # Template de configuracion
└── .gitignore
```

## Convenciones de Nomenclatura

### Clases por Capa
- `CE_*` = Capa Entidad (ej: CE_Usuarios, CE_Productos)
- `CN_*` = Capa Negocio (ej: CN_Usuarios, CN_Productos)
- `CD_*` = Capa Datos Legacy (ej: CD_Usuarios, CD_Productos)
- `I*Repository` = Interfaces de repositorio
- `*Repository` = Implementaciones de repositorio
- `*DTO` = Data Transfer Objects
- `*Controller` = Controladores API

### Archivos
- Views: PascalCase (ej: `Dashboard.xaml`, `POS.xaml`)
- Temas: PascalCase (ej: `Dark.xaml`, `Green.xaml`)
- DTOs: PascalCase con sufijo DTO (ej: `ProductoDTO.cs`)

## Patrones de Diseno

### Repository Pattern
```csharp
// Interfaz
public interface IProductoRepository : IRepository<CE_Productos>
{
    Task<CE_Productos> GetByCodigoAsync(string codigo);
}

// Implementacion
public class ProductoRepository : Repository<CE_Productos>, IProductoRepository
{
    public async Task<CE_Productos> GetByCodigoAsync(string codigo)
    {
        return await _dbSet.FirstOrDefaultAsync(p => p.Codigo == codigo);
    }
}
```

### Unit of Work
```csharp
public interface IUnitOfWork : IDisposable
{
    IUsuarioRepository Usuarios { get; }
    IProductoRepository Productos { get; }
    Task<int> SaveChangesAsync();
}
```

### Dependency Injection
```csharp
// En Program.cs
builder.Services.AddScoped<ApplicationDbContext>();
builder.Services.AddScoped<IUsuarioRepository, UsuarioRepository>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

### DTO Pattern
```csharp
// Entidad (BD)
public class CE_Productos { public byte[] Img; ... }

// DTO (API)
public class ProductoDTO { public bool TieneImagen; ... }
```

## Reglas de Desarrollo

### Seguridad
1. **NUNCA** hardcodear credenciales en el codigo
2. Usar `appsettings.json` para configuracion sensible
3. `appsettings.json` esta en `.gitignore`
4. Usar JWT para autenticacion en la API
5. Las contrasenas en BD estan encriptadas con pgcrypto

### Arquitectura
1. Mantener separacion estricta de capas
2. La Capa Presentacion NO accede directamente a Capa Datos
3. Usar interfaces para abstraer dependencias (DI)
4. La API usa DTOs, nunca expone entidades directamente
5. Validacion en DTOs con DataAnnotations

### Base de Datos
1. EF Core para nuevas funcionalidades
2. Usar async/await para todas las operaciones
3. Los nombres de tablas/columnas usan PascalCase con comillas en PostgreSQL
4. Zona horaria: America/Argentina/Buenos_Aires (UTC-3)
5. Extension pgcrypto para encriptacion de contrasenas

### Mapeo EF Core - PostgreSQL
```csharp
// Los nombres deben coincidir exactamente con el SQL
entity.ToTable("Usuarios");  // No "usuarios"
entity.Property(e => e.IdUsuario).HasColumnName("IdUsuario");
```

### API REST
1. Usar verbos HTTP correctamente (GET, POST, PUT, DELETE)
2. Retornar codigos de estado apropiados (200, 201, 400, 401, 404)
3. Envolver respuestas en ApiResponse<T>
4. Documentar con Swagger
5. Proteger endpoints con [Authorize]

### Codigo
1. Usar async/await para operaciones de BD
2. Implementar IDisposable donde corresponda
3. Usar regiones (#region) para organizar codigo extenso
4. Comentarios en espanol

## Configuracion

### appsettings.json (NO commitear)
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=...;Database=...;Username=...;Password=...;SslMode=Require"
  },
  "JwtKey": "TU_CLAVE_SECRETA_DE_32_CARACTERES_MINIMO",
  "JwtIssuer": "PuntoDeVenta.API",
  "JwtAudience": "PuntoDeVenta.Clients",
  "JwtExpirationMinutes": "60"
}
```

## Comandos Utiles

```bash
# Compilar toda la solucion
dotnet build PuntoDeVenta.sln

# Ejecutar cliente WPF
dotnet run --project "PuntoDeVenta/Capa Presentacion.csproj"

# Ejecutar API
dotnet run --project PuntoDeVenta.API/PuntoDeVenta.API.csproj
# Swagger disponible en: http://localhost:5207/swagger
```

## Plan de Migracion

### Fase 1: Seguridad (COMPLETADA)
- [x] Mover credenciales a appsettings.json
- [x] Implementar ConfigurationHelper
- [x] Agregar .gitignore
- [x] Documentar tecnologias nuevas

### Fase 2: Entity Framework Core (COMPLETADA)
- [x] Migrar proyectos a .NET 8
- [x] Implementar Entity Framework Core
- [x] Crear ApplicationDbContext
- [x] Implementar Repository Pattern
- [x] Implementar Unit of Work
- [x] Inyeccion de dependencias

### Fase 3: API REST (COMPLETADA)
- [x] Crear proyecto ASP.NET Core Web API
- [x] Implementar JWT Authentication
- [x] Crear Controllers (Auth, Productos, Usuarios, Clientes, Grupos, Movimientos)
- [x] Crear DTOs
- [x] Configurar Swagger
- [x] Configurar CORS

### Fase 4: Frontend Web (COMPLETADA)
- [x] Crear proyecto Blazor WebAssembly
- [x] Migrar vistas principales

### Fase 5: Testing y DevOps (EN PROGRESO)
- [ ] Agregar xUnit tests
- [ ] Configurar GitHub Actions
- [x] Docker compose
- [x] Deploy en Railway

## Dependencias NuGet

### Capa Datos
- Microsoft.EntityFrameworkCore 8.0.10
- Npgsql.EntityFrameworkCore.PostgreSQL 8.0.10
- Microsoft.Extensions.Configuration.Json 8.0.1

### PuntoDeVenta.API
- Microsoft.AspNetCore.Authentication.JwtBearer 8.0.10
- Swashbuckle.AspNetCore 6.6.2
- BCrypt.Net-Next 4.0.3

### Capa Presentacion (WPF)
- iTextSharp 5.5.13.3
- LiveCharts 0.9.7
- LiveCharts.Wpf 0.9.7

## Deploy en Railway

### Arquitectura de servicios
El proyecto se despliega en Railway con **2 servicios separados** desde el mismo repositorio:

| Servicio | Dominio | Dockerfile | Root Directory (Railway) |
|----------|---------|------------|--------------------------|
| **API** (.NET) | `pointofsale-production.up.railway.app` | `PuntoDeVenta.API/Dockerfile` | *(vacío - raíz del repo)* |
| **Web** (Blazor WASM + nginx) | `lafamilia-gestion.up.railway.app` | `PuntoDeVenta.Web/Dockerfile` | `PuntoDeVenta.Web` |

### Archivos de configuración de Railway
- `railway.json` (raíz) → usado por el servicio **API** (apunta a `PuntoDeVenta.API/Dockerfile`)
- `PuntoDeVenta.Web/railway.json` → usado por el servicio **Web** (apunta a `Dockerfile` relativo)
- `.dockerignore` (raíz) → usado por la **API**, excluye `PuntoDeVenta.Web/` y `PuntoDeVenta/` (WPF)
- `PuntoDeVenta.API/.dockerignore` → usado cuando se builda la API con docker-compose local

### Variables de entorno requeridas en Railway (servicio API)
```
DATABASE_URL=postgresql://user:password@host:port/database
JWT_KEY=cadena_de_32_caracteres_minimo
JWT_ISSUER=PuntoDeVenta.API
JWT_AUDIENCE=PuntoDeVenta.Clients
JWT_EXPIRATION_MINUTES=60
ALLOWED_ORIGINS=https://lafamilia-gestion.up.railway.app
ASPNETCORE_ENVIRONMENT=Production
```

### URL de la API en el frontend
Configurada en `PuntoDeVenta.Web/wwwroot/appsettings.Production.json`:
```json
{
  "ApiBaseUrl": "https://pointofsale-production.up.railway.app/"
}
```

### Consideraciones importantes
1. **CORS**: `UseCors()` debe ir **antes** de `UseRateLimiter()` en el pipeline HTTP de `Program.cs`, para que los preflight OPTIONS no sean bloqueados
2. **`.dockerignore` de la raíz**: Está diseñado para el build de la API. NO debe excluir `PuntoDeVenta.API/`, `Capa Datos/`, `Capa Entidad/`, `Capa Negocio/`
3. **Root Directory en Railway**: El servicio API NO debe tener root directory (usa la raíz). El servicio Web debe tener root directory `PuntoDeVenta.Web`
4. Si Railway muestra el builder incorrecto, verificar que el `railway.json` correspondiente esté apuntando al Dockerfile correcto

## Notas Importantes

1. La BD usa extension `pgcrypto` para encriptar contrasenas
2. Las contrasenas se validan con raw SQL usando `pgp_sym_decrypt`
3. El campo `Patron` del usuario es la clave de encriptacion
4. Usuario demo: admin / admin (patron: PuntoDeVenta)

## Contacto
- GitHub: https://github.com/EmmaVZ89
- Proyecto: Portfolio Upwork

---
> Source: [EmmaVZ89/PointOfSale](https://github.com/EmmaVZ89/PointOfSale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
