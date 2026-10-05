## nest-modules

> NestJS module, controller, and service patterns for core-api


# NestJS Module Patterns

## Controllers

- Thin layer: validate input, delegate to service, return response
- Always decorate with `@ApiTags('feature')` and `@ApiOperation()`
- Use DTOs for request validation (never raw `body: any`)
- Anti-farming check: maintainers cannot earn FoxPoints in their own projects

```typescript
@ApiTags('feature')
@Controller('feature')
export class FeatureController {
  constructor(private readonly featureService: FeatureService) {}

  @Post()
  @ApiOperation({ summary: 'Create feature' })
  async create(
    @Body() dto: CreateFeatureDto,
    @CurrentUser() user: JwtPayload,
  ) {
    return this.featureService.create(dto, user.user_id);
  }
}
```

## Services

- All business logic, validations, and Prisma queries live here
- Validate existence before operations (`findUnique` → `NotFoundException`)
- Validate permissions before mutations (`ForbiddenException`)
- Use transactions for multi-step writes: `this.prisma.$transaction(async (tx) => { ... })`
- Emit events for cross-module side effects: `this.eventEmitter.emit('event.name', payload)`

## DTOs

- Use `class-validator` decorators: `@IsString()`, `@IsUUID()`, `@IsOptional()`, etc.
- Use `class-transformer`: `@Transform()`, `@Type()`
- Export types from DTOs, never define inline types in controllers
- Follow naming: `Create{Feature}Dto`, `Update{Feature}Dto`, `{Feature}ResponseDto`

## Module Registration

- Import `DatabaseModule` is global — no need to import in each module
- Export services that other modules need
- Use `forwardRef()` only when circular deps are unavoidable

---
> Source: [VelaPayments/vela-server](https://github.com/VelaPayments/vela-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
