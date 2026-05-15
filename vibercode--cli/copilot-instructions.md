## cli

> ViberCode CLI es una herramienta de línea de comandos para generar APIs Go con arquitectura limpia, incluyendo operaciones CRUD completas, integración con bases de datos, y un sistema de chat AI interactivo.

# ViberCode CLI

## Información General

ViberCode CLI es una herramienta de línea de comandos para generar APIs Go con arquitectura limpia, incluyendo operaciones CRUD completas, integración con bases de datos, y un sistema de chat AI interactivo.

## Nuevas Características

### 🔌 Servidor MCP (Model Context Protocol)

El nuevo servidor MCP permite que agentes de IA interactúen directamente con ViberCode para:

- **Edición de componentes en vivo**: Actualizar propiedades, posición y tamaño de componentes en tiempo real
- **Chat integrado**: Enviar mensajes al asistente Viber AI desde agentes externos
- **Generación de código**: Crear APIs Go completas usando agentes IA
- **Gestión de estado**: Obtener y actualizar el estado de la vista y componentes

#### Uso Rápido

```bash
# Iniciar servidor MCP
vibercode mcp

# Probar el servidor
./test-mcp-server.sh
```

#### Configuración para Claude Desktop

Agregar al archivo `.mcp.json`:

```json
{
  "mcpServers": {
    "vibercode": {
      "name": "ViberCode MCP Server",
      "description": "ViberCode CLI integration for live component editing",
      "command": "vibercode",
      "args": ["mcp"],
      "env": {
        "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}",
        "VIBE_DEBUG": "true"
      }
    }
  }
}
```

#### Herramientas Disponibles

- `vibe_start`: Iniciar modo vibe con chat AI y preview
- `component_update`: Actualizar componentes en tiempo real
- `view_state_get`: Obtener estado actual de la vista
- `chat_send`: Enviar mensaje al asistente Viber AI
- `generate_code`: Generar código Go API
- `project_status`: Estado del proyecto y servidores

## Arquitectura

### CLI Structure

```
vibercode-cli-go/
├── main.go                      # Application entry point
├── cmd/                         # Cobra CLI commands
│   ├── root.go                 # Root command definition
│   └── generate.go             # Generate subcommands
├── internal/
│   ├── generator/              # Code generation logic
│   │   ├── api.go             # API project generator
│   │   └── resource.go        # Resource CRUD generator
│   ├── models/                 # Data structures
│   │   └── field.go           # Field types and resource models
│   └── templates/              # Go template strings
│       ├── model.go           # Model template
│       ├── handler.go         # HTTP handler template
│       ├── service.go         # Business logic template
│       └── repository.go      # Data access template
├── go.mod                      # Go module definition
└── README.md                   # Documentation
```

### Generated Project Architecture

The CLI generates Go projects following clean architecture principles:

```
generated-project/
├── cmd/server/main.go          # Application entry point
├── internal/
│   ├── handlers/               # HTTP layer (Gin framework)
│   ├── services/               # Business logic layer
│   ├── repositories/           # Data access layer
│   ├── models/                 # Domain models and DTOs
│   └── middleware/             # HTTP middleware
├── pkg/
│   ├── database/               # Database connection utilities
│   ├── config/                 # Configuration management
│   └── utils/                  # Shared utilities
└── go.mod                      # Go module
```

### Integración MCP

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   AI Agent      │    │   MCP Server    │    │   ViberCode     │
│   (Claude)      │◄──►│   (JSON-RPC)    │◄──►│   Services      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   WebSocket     │
                       │   + HTTP API    │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │  React Editor   │
                       │  + Live Preview │
                       └─────────────────┘
```

## Comandos Disponibles

### CLI Commands

- `vibercode generate api` - Generate a complete Go API project with clean architecture
- `vibercode generate resource` - Generate CRUD resources with models, handlers, services, and repositories
- `vibercode --help` - Show help information
- `vibercode generate --help` - Show help for generate commands

### Development Commands

- `go build -o vibercode main.go` - Build the CLI binary
- `go mod tidy` - Clean up dependencies
- `go test ./...` - Run tests
- `go run main.go` - Run CLI during development

### `vibercode mcp`

Inicia el servidor MCP para integración con agentes IA.

```bash
vibercode mcp
```

**Características:**

- Protocolo MCP 2024-11-05 compatible
- Comunicación JSON-RPC via stdin/stdout
- Herramientas bien definidas con validación
- Integración con WebSocket y HTTP API

## Key Features

### Field Type System

The CLI supports a comprehensive field type system defined in `internal/models/field.go`:

- **Basic Types**: `string`, `text`, `number`, `float`, `boolean`
- **Special Types**: `date`, `uuid`, `json`
- **Relations**: `relation` (foreign key), `relation-array` (one-to-many)

Each field type maps to appropriate Go types and generates corresponding:

- Struct field definitions with JSON and GORM tags
- Validation logic
- Required imports

### Template System

Uses Go's native `text/template` package with custom helper functions:

- `ToCamel`, `ToLowerCamel`, `ToSnake`, `ToKebab` for string case conversion
- Template variables for resource name variations
- Conditional rendering based on field types and requirements

### Interactive Prompts

Uses `promptui` library for user-friendly CLI interactions:

- Text input with validation
- Select menus for predefined options
- Confirmation prompts
- Error handling and retry logic

## Code Generation Process

### API Project Generation

1. Collect project information (name, port, database type, module)
2. Create directory structure
3. Generate Go module file
4. Generate main application file
5. Generate database connection package
6. Generate basic handler setup
7. Generate Docker configuration
8. Generate environment configuration

### Resource Generation

1. Collect resource information (name, description, module)
2. Collect field definitions interactively
3. Process field types and generate Go struct definitions
4. Generate model file with GORM annotations
5. Generate handler file with CRUD endpoints
6. Generate service file with business logic
7. Generate repository file with database operations

## Dependencies

### Core Dependencies

- **github.com/spf13/cobra**: CLI framework for command structure
- **github.com/manifoldco/promptui**: Interactive prompts
- **github.com/iancoleman/strcase**: String case conversion utilities

### Generated Project Dependencies

- **github.com/gin-gonic/gin**: HTTP web framework
- **gorm.io/gorm**: ORM library
- **github.com/joho/godotenv**: Environment variable management
- **github.com/google/uuid**: UUID generation
- Database drivers (postgres, mysql, sqlite)

## Development Notes

### Template Debugging

Templates are defined as string constants in `internal/templates/`. To debug template issues:

1. Check template syntax in the constant definitions
2. Verify helper function names match the template usage
3. Ensure data structure fields match template variables

### Adding New Field Types

1. Add the new type to `FieldType` constants in `internal/models/field.go`
2. Update the `GoType()` method to return appropriate Go type
3. Update the `GoStructField()` method for proper struct field generation
4. Add validation logic in `GoValidation()` method
5. Update required imports in `RequiredImports()` method

### Extending Commands

1. Add new command definitions in `cmd/generate.go`
2. Create corresponding generator in `internal/generator/`
3. Add templates in `internal/templates/`
4. Update help text and documentation

## Task Management System

### Task Organization

- All project tasks are managed in `tasks.md` with prioritized dependency order
- Individual task details are stored in `/tasks/` directory as separate markdown files
- When adding new tasks, analyze dependencies and insert in correct order for coherent development flow

### Task Management Rules

1. **New Task Addition**: Always check `tasks.md` for existing tasks and dependencies before adding new ones
2. **Dependency Analysis**: New tasks must be placed in correct position relative to existing tasks
3. **Coherent Development**: Task order should ensure logical development progression
4. **Task Documentation**: Each task requires detailed specification in `/tasks/` directory

### Database Provider Priority

- **Supabase Integration**: High priority database provider to be added alongside existing PostgreSQL, MySQL, and SQLite support
- Supabase includes: connection configuration, auth integration, real-time subscriptions, and storage setup

## Best Practices

- Follow Go naming conventions for generated code
- Use GORM best practices for database models
- Include proper error handling in generated code
- Generate comprehensive validation logic
- Maintain clean architecture separation
- Include proper HTTP status codes and responses
- Always check task dependencies before implementing new features
- Update task status and documentation as development progresses

---
> Source: [vibercode/cli](https://github.com/vibercode/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
