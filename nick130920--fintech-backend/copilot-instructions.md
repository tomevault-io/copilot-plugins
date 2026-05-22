## rule-cursor

> Reglas de diseño y desarrollo para Fintech Backend


# Fintech Backend - Reglas de Desarrollo

## 🏗️ Arquitectura y Estructura

### Clean Architecture
- Mantener separación clara entre capas: handlers, use cases, entities, repositories
- No mezclar lógica de negocio con detalles de implementación
- Dependencias deben apuntar hacia el centro (domain)

### Estructura de Directorios
```
internal/
├── controller/http/v1/     # Handlers HTTP
├── usecase/               # Lógica de negocio
├── entity/                # Entidades de dominio
├── repository/            # Implementaciones de repositorio
└── app/                   # Configuración de la aplicación
```

## 📝 Estándares de Código

### Nomenclatura
- **Handlers**: Terminar con `Handler` (ej: `ExpenseHandler`)
- **Use Cases**: Usar verbos descriptivos (ej: `CreateExpense`, `GetUserBudget`)
- **Entities**: Nombres singulares (ej: `User`, `Expense`, `Budget`)
- **DTOs**: Terminar con el propósito (ej: `CreateExpenseRequest`, `ExpenseResponse`)

### Manejo de Errores
```go
// ✅ CORRECTO: Errores descriptivos
if err != nil {
    return nil, fmt.Errorf("failed to create expense: %w", err)
}

// ✅ CORRECTO: Errores de dominio
var ErrExpenseNotFound = errors.New("expense not found")

// ❌ INCORRECTO: Errores genéricos
if err != nil {
    return nil, err
}
```

### Validación de Datos
```go
// ✅ CORRECTO: Validar en el handler
func (h *ExpenseHandler) CreateExpense(c *gin.Context) {
    var req dto.CreateExpenseRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": "invalid request"})
        return
    }
    
    if err := h.validator.Validate(req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
}
```

## 🔒 Seguridad y Autenticación

### JWT y Middleware
- Siempre validar tokens JWT en endpoints protegidos
- Extraer user ID del contexto autenticado
- No exponer información sensible en logs

```go
// ✅ CORRECTO: Obtener user ID del contexto
userID, exists := c.Get("user_id")
if !exists {
    c.JSON(401, gin.H{"error": "unauthorized"})
    return
}
```

### Validación de Permisos
- Verificar que el usuario tiene acceso a los recursos solicitados
- Validar ownership de datos antes de operaciones CRUD

## 💾 Base de Datos

### Transacciones
```go
// ✅ CORRECTO: Usar transacciones para operaciones complejas
tx := r.db.Begin()
defer func() {
    if r := recover(); r != nil {
        tx.Rollback()
    }
}()

if err := tx.Create(&expense).Error; err != nil {
    tx.Rollback()
    return err
}

if err := tx.Create(&budgetUpdate).Error; err != nil {
    tx.Rollback()
    return err
}

return tx.Commit().Error
```

### Migraciones
- Siempre crear migraciones para cambios de esquema
- Nombrar migraciones con timestamp y descripción clara
- Incluir rollback en todas las migraciones

## 📊 DTOs y Responses

### Estructura Consistente
```go
// ✅ CORRECTO: Response estructurado
type ExpenseResponse struct {
    ID          uint      `json:"id"`
    Amount      float64   `json:"amount"`
    Description string    `json:"description"`
    CategoryID  uint      `json:"category_id"`
    Category    Category  `json:"category"`
    CreatedAt   time.Time `json:"created_at"`
}

// ✅ CORRECTO: Request con validaciones
type CreateExpenseRequest struct {
    Amount      float64 `json:"amount" validate:"required,gt=0"`
    Description string  `json:"description" validate:"required"`
    CategoryID  uint    `json:"category_id" validate:"required"`
}
```

### Paginación Estándar
```go
type PaginatedResponse struct {
    Data       interface{} `json:"data"`
    Total      int64      `json:"total"`
    Page       int        `json:"page"`
    PerPage    int        `json:"per_page"`
    TotalPages int        `json:"total_pages"`
}
```

## 🧪 Testing

### Estructura de Tests
```go
func TestExpenseHandler_CreateExpense(t *testing.T) {
    tests := []struct {
        name           string
        request        dto.CreateExpenseRequest
        expectedStatus int
        expectedError  string
    }{
        {
            name: "valid expense creation",
            request: dto.CreateExpenseRequest{
                Amount:      100.0,
                Description: "Test expense",
                CategoryID:  1,
            },
            expectedStatus: 201,
        },
        // ... más casos
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Test implementation
        })
    }
}
```

### Mocks y Dependencias
- Usar interfaces para todas las dependencias
- Crear mocks para testing unitario
- Tests de integración para endpoints completos

## 📡 API Design

### RESTful Endpoints
```
GET    /api/v1/expenses              # Listar gastos
POST   /api/v1/expenses              # Crear gasto
GET    /api/v1/expenses/:id          # Obtener gasto específico
PUT    /api/v1/expenses/:id          # Actualizar gasto
DELETE /api/v1/expenses/:id          # Eliminar gasto
```

### Status Codes Estándar
- `200` - OK (GET, PUT exitosos)
- `201` - Created (POST exitoso)
- `204` - No Content (DELETE exitoso)
- `400` - Bad Request (validación falló)
- `401` - Unauthorized (no autenticado)
- `403` - Forbidden (no autorizado)
- `404` - Not Found (recurso no existe)
- `500` - Internal Server Error

### Headers de Response
```go
// ✅ CORRECTO: Headers consistentes
c.Header("Content-Type", "application/json")
c.Header("X-API-Version", "v1")
```

## 🔍 Logging y Monitoring

### Structured Logging
```go
// ✅ CORRECTO: Logs estructurados
log.WithFields(log.Fields{
    "user_id":    userID,
    "expense_id": expenseID,
    "amount":     amount,
}).Info("expense created successfully")

// ❌ INCORRECTO: Logs no estructurados
log.Printf("User %d created expense %d with amount %f", userID, expenseID, amount)
```

### Métricas
- Instrumentar endpoints críticos
- Medir latencia de operaciones de base de datos
- Trackear errores por tipo

## 🚀 Performance

### Database Queries
```go
// ✅ CORRECTO: Preload relaciones necesarias
var expenses []entity.Expense
err := r.db.Preload("Category").Where("user_id = ?", userID).Find(&expenses).Error

// ❌ INCORRECTO: N+1 queries
for _, expense := range expenses {
    r.db.First(&expense.Category, expense.CategoryID)
}
```

### Caching
- Implementar cache para datos frecuentemente accedidos
- Cache de categorías, configuraciones de usuario
- TTL apropiado para cada tipo de data

## 🔧 Configuración

### Environment Variables
```go
// ✅ CORRECTO: Configuración centralizada
type Config struct {
    Port       string `env:"PORT" envDefault:"8080"`
    DBHost     string `env:"DB_HOST" envDefault:"localhost"`
    DBPort     string `env:"DB_PORT" envDefault:"5432"`
    JWTSecret  string `env:"JWT_SECRET,required"`
}
```

### Secrets Management
- Nunca hardcodear secrets en código
- Usar variables de entorno para configuración sensible
- Rotar secrets regularmente

## 📚 Documentación

### Swagger/OpenAPI
- Documentar todos los endpoints
- Incluir ejemplos de request/response
- Mantener documentación actualizada con el código

### Comments
```go
// CreateExpense handles the creation of a new expense for the authenticated user.
// It validates the request, checks budget limits, and persists the expense.
//
// @Summary Create a new expense
// @Tags expenses
// @Accept json
// @Produce json
// @Param expense body dto.CreateExpenseRequest true "Expense data"
// @Success 201 {object} dto.ExpenseResponse
// @Failure 400 {object} dto.ErrorResponse
// @Router /expenses [post]
func (h *ExpenseHandler) CreateExpense(c *gin.Context) {
    // Implementation
}
```

## 🔄 Migration y Deployment

### Database Migrations
- Versionar todas las migraciones
- Incluir rollback para cada migración
- Testar migraciones en staging antes de production

### Deployment
- Zero-downtime deployments
- Health checks configurados
- Rollback automático en caso de fallos

## ⚡ Quick Reference

### Checklist Pre-Commit
- [ ] Tests pasan (unit + integration)
- [ ] Linting sin errores (`golangci-lint`)
- [ ] Documentación actualizada
- [ ] Migraciones incluidas si es necesario
- [ ] Logs estructurados agregados
- [ ] Validaciones de entrada implementadas
- [ ] Manejo de errores apropiado
- [ ] No hay secrets hardcodeados

### Comandos Útiles
```bash
# Testing
go test ./... -cover

# Linting
golangci-lint run

# Build
go build -o bin/fintech-backend cmd/server/main.go

# Migration
migrate -path migrations -database "postgres://..." up
```

---

**Recordatorio**: Estas reglas aseguran código mantenible, seguro y escalable para la aplicación fintech. La consistencia es clave para el trabajo en equipo y la calidad del producto.

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
