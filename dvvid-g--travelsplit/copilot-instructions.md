## travelsplit

> Enforces Swagger/OpenAPI documentation standards for all API endpoints in TravelSplit backend.


# API Documentation Standards (Swagger/OpenAPI)

## Rules

- Document all controllers with `@ApiTags()`.
- Document all endpoints with `@ApiOperation()`.
- Document all DTOs with `@ApiProperty()`.
- Document all response codes with `@ApiResponse()` or specific decorators.
- Document all parameters with `@ApiParam()` or `@ApiQuery()`.
- Provide descriptions and examples for all properties.
- Use consistent Spanish descriptions for user-facing documentation.
- Document error responses for all endpoints.

## Controller Documentation

- Use `@ApiTags('resource-name')` at controller level.
- Group related endpoints under same tag.
- Use descriptive tag names matching resource.

## Endpoint Documentation

- Use `@ApiOperation()` with `summary` for all endpoints.
- Provide clear, concise summaries in Spanish.
- Document request body with DTO type.
- Document all possible response codes.
- Document authentication requirements.

## Response Documentation

- Use `@ApiResponse()` for success responses (200, 201, etc.).
- Use `@ApiBadRequestResponse()` for 400 errors.
- Use `@ApiUnauthorizedResponse()` for 401 errors.
- Use `@ApiForbiddenResponse()` for 403 errors.
- Use `@ApiNotFoundResponse()` for 404 errors.
- Use `@ApiConflictResponse()` for 409 errors.
- Use `@ApiInternalServerErrorResponse()` for 500 errors.
- Provide descriptions for all error responses.

## DTO Documentation

- Use `@ApiProperty()` for all DTO properties.
- Provide `description`, `example`, and `type` for each property.
- Use `required: false` for optional properties.
- Use `enum` for enum types.
- Use `isArray: true` for array types.

## Parameter Documentation

- Use `@ApiParam()` for path parameters.
- Use `@ApiQuery()` for query parameters.
- Use `@ApiBody()` for request body when needed.
- Provide descriptions and examples for all parameters.

## Swagger Configuration

- Configure Swagger in `main.ts` using `DocumentBuilder`.
- Set title, description, version, and tags.
- Add authentication configuration if using JWT.
- Set up Swagger UI at `/api/docs` endpoint.

## DO

```typescript
// Controller with full documentation
@ApiTags('users')
@Controller('users')
export class UsersController {
  @Get()
  @ApiOperation({ summary: 'Obtener todos los usuarios activos' })
  @ApiResponse({
    status: 200,
    description: 'Lista de usuarios obtenida exitosamente',
    type: [UserResponseDto],
  })
  async findAll(): Promise<UserResponseDto[]> {
    return this.usersService.findAll();
  }
  
  @Get(':id')
  @ApiOperation({ summary: 'Obtener un usuario por ID' })
  @ApiParam({
    name: 'id',
    description: 'ID único del usuario',
    type: String,
  })
  @ApiResponse({
    status: 200,
    description: 'Usuario encontrado',
    type: UserResponseDto,
  })
  @ApiNotFoundResponse({ description: 'Usuario no encontrado' })
  async findOne(
    @Param('id', ParseUUIDPipe) id: string,
  ): Promise<UserResponseDto> {
    return this.usersService.findOne(id);
  }
  
  @Post()
  @ApiOperation({ summary: 'Crear un nuevo usuario' })
  @ApiResponse({
    status: 201,
    description: 'Usuario creado exitosamente',
    type: UserResponseDto,
  })
  @ApiBadRequestResponse({ description: 'Datos de entrada inválidos' })
  @ApiConflictResponse({ description: 'El email ya está registrado' })
  async create(@Body() createUserDto: CreateUserDto): Promise<UserResponseDto> {
    return this.usersService.create(createUserDto);
  }
  
  @Put(':id')
  @ApiOperation({ summary: 'Actualizar un usuario' })
  @ApiParam({ name: 'id', description: 'ID del usuario a actualizar' })
  @ApiResponse({
    status: 200,
    description: 'Usuario actualizado exitosamente',
    type: UserResponseDto,
  })
  @ApiNotFoundResponse({ description: 'Usuario no encontrado' })
  @ApiForbiddenResponse({ description: 'No tienes permiso para actualizar este usuario' })
  async update(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() updateUserDto: UpdateUserDto,
  ): Promise<UserResponseDto> {
    return this.usersService.update(id, updateUserDto);
  }
}

// DTO with full documentation
export class CreateUserDto {
  @ApiProperty({
    description: 'Nombre del usuario',
    example: 'Juan Pérez',
    type: String,
  })
  @IsString()
  @IsNotEmpty({ message: 'El nombre es requerido' })
  nombre!: string;
  
  @ApiProperty({
    description: 'Email del usuario (debe ser un formato válido)',
    example: 'juan.perez@example.com',
    type: String,
  })
  @IsEmail({}, { message: 'El email debe tener un formato válido' })
  @IsNotEmpty({ message: 'El email es requerido' })
  email!: string;
  
  @ApiProperty({
    description: 'Contraseña del usuario (mínimo 8 caracteres)',
    example: 'miPassword123',
    type: String,
    minLength: 8,
  })
  @IsString()
  @IsNotEmpty({ message: 'La contraseña es requerida' })
  @MinLength(8, { message: 'La contraseña debe tener al menos 8 caracteres' })
  contraseña!: string;
}

// Swagger configuration in main.ts
const config = new DocumentBuilder()
  .setTitle('TravelSplit API')
  .setDescription('API REST para la gestión de gastos de viaje en grupo')
  .setVersion('1.0')
  .addTag('users', 'Operaciones relacionadas con usuarios')
  .addTag('trips', 'Operaciones relacionadas con viajes')
  .addTag('expenses', 'Operaciones relacionadas con gastos')
  .addBearerAuth()
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);
```

```typescript
// Query parameters documentation
@Get('search')
@ApiOperation({ summary: 'Buscar usuarios por email' })
@ApiQuery({
  name: 'email',
  description: 'Email a buscar',
  required: true,
  type: String,
})
@ApiResponse({
  status: 200,
  description: 'Usuarios encontrados',
  type: [UserResponseDto],
})
async search(@Query('email') email: string): Promise<UserResponseDto[]> {
  return this.usersService.searchByEmail(email);
}
```

## DON'T

```typescript
// VIOLATION: Missing API documentation
@Controller('users')
export class UsersController {
  @Get() // VIOLATION: Missing @ApiOperation() and @ApiResponse()
  async findAll(): Promise<UserResponseDto[]> {
    return this.usersService.findAll();
  }
}
```

```typescript
// VIOLATION: Missing DTO documentation
export class CreateUserDto {
  @IsString()
  nombre!: string; // VIOLATION: Missing @ApiProperty()
  
  @IsEmail()
  email!: string; // VIOLATION: Missing @ApiProperty()
}
```

```typescript
// VIOLATION: Missing error response documentation
@Post()
@ApiOperation({ summary: 'Create user' })
@ApiResponse({ status: 201, type: UserResponseDto })
// VIOLATION: Missing @ApiBadRequestResponse() and @ApiConflictResponse()
async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
  return this.usersService.create(dto);
}
```

```typescript
// VIOLATION: Missing parameter documentation
@Get(':id')
@ApiOperation({ summary: 'Get user' })
// VIOLATION: Missing @ApiParam() for id parameter
async findOne(@Param('id') id: string): Promise<UserResponseDto> {
  return this.usersService.findOne(id);
}
```

```typescript
// VIOLATION: Incomplete Swagger configuration
const config = new DocumentBuilder()
  .setTitle('API')
  // VIOLATION: Missing description, version, tags
  .build();
```

## Documentation Best Practices

- Write descriptions in Spanish for user-facing documentation.
- Provide realistic examples in `@ApiProperty()`.
- Document all possible error scenarios.
- Keep summaries concise but descriptive.
- Use consistent terminology across documentation.
- Update documentation when endpoints change.

## Authentication Documentation

- Use `@ApiBearerAuth()` for JWT authentication.
- Document authentication requirements in endpoint descriptions.
- Use `@ApiSecurity()` for custom authentication schemes.
- Configure authentication in Swagger setup.

## Exceptions

- Health check endpoints may have minimal documentation.
- Internal endpoints may skip Swagger documentation if not exposed.
- Test controllers do not require Swagger documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DVVID-G) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
