## valu

> This is a multi-app taxi solution with the following structure:

# Taxi Solution Development Patterns

## Project Structure
This is a multi-app taxi solution with the following structure:
- `apps/admin-api/` - NestJS backend for admin operations
- `apps/admin-panel/` - Angular frontend for admin interface
- `apps/driver-api/` - NestJS backend for driver operations
- `apps/driver-frontend/` - Flutter app for drivers
- `apps/rider-api/` - NestJS backend for rider operations
- `apps/rider-frontend/` - Flutter app for riders
- `libs/` - Shared libraries and database entities

## Backend Development Patterns

### GraphQL Resolvers
- Always use `@UseGuards(JwtAuthGuard)` for protected endpoints
- Return DTOs instead of raw entities to avoid GraphQL type errors
- Use proper TypeScript types and nullable fields where appropriate
- Example resolver pattern:
```typescript
@Query(() => SomeConnection)
async someQuery(
    @Args("search", { nullable: true }) search?: string,
    @Args("limit", { type: () => Int, defaultValue: 10 }) limit: number = 10,
    @Args("offset", { type: () => Int, defaultValue: 0 }) offset: number = 0
): Promise<SomeConnection> {
    const result = await this.service.getData(search, limit, offset);
    return result;
}
```

### Service Layer
- Use TypeORM repositories with proper query builders
- Handle PostgreSQL-specific syntax (not MySQL)
- Use proper soft delete filters (`deletedAt IS NULL`)
- Cast bigint fields to text before using LIKE operations
- Example service pattern:
```typescript
async getData(search?: string, limit: number = 10, offset: number = 0) {
    const queryBuilder = this.repository
        .createQueryBuilder("entity")
        .where("entity.deletedAt IS NULL");

    if (search) {
        const isNumeric = /^\d+$/.test(search);
        let searchCondition = "(entity.name LIKE :search OR entity.email LIKE :search)";
        if (isNumeric) {
            searchCondition = "(CAST(entity.id AS TEXT) LIKE :search OR " + searchCondition.slice(1);
        }
        queryBuilder.andWhere(searchCondition, { search: `%${search}%` });
    }

    return queryBuilder
        .limit(limit)
        .offset(offset)
        .getManyAndCount();
}
```

### DTOs and Entities
- Create separate DTOs for GraphQL output types
- Use `@Field()` decorators with proper types
- Handle nullable fields appropriately
- Example DTO pattern:
```typescript
@ObjectType("SomeDTO")
export class SomeDTO {
    @Field(() => ID)
    id!: number;

    @Field()
    name!: string;

    @Field({ nullable: true })
    email?: string;

    @Field(() => Date, { nullable: true })
    createdAt?: Date;
}
```

### Module Configuration
- Always include entities in `TypeOrmModule.forFeature()` arrays
- Import required modules to resolve dependency injection errors
- Example module pattern:
```typescript
@Module({
    imports: [
        TypeOrmModule.forFeature([Entity1, Entity2, Entity3]),
        SharedModule,
        OtherModule
    ],
    providers: [Service, Resolver],
    exports: [Service]
})
export class SomeModule {}
```

## Frontend Development Patterns

### Angular Components
- Use Angular Material components (`nz-card`, `nz-table`, etc.)
- Implement proper error handling with `*ngIf` and error templates
- Use reactive forms with debounced search
- Example component pattern:
```typescript
export class SomeComponent implements OnInit {
    query?: Observable<ApolloQueryResult<any>>;
    searchControl = new FormControl("");

    ngOnInit(): void {
        this.query = this.route.data.pipe(map((data) => data.data));

        this.searchControl.valueChanges
            .pipe(debounceTime(500), distinctUntilChanged())
            .subscribe((searchTerm) => {
                this.searchData();
            });
    }

    searchData(): void {
        this.router.navigate([], {
            relativeTo: this.route,
            queryParams: { search: this.searchControl.value || null },
            queryParamsHandling: "merge",
        });
    }
}
```

### GraphQL Queries
- Use proper TypeScript types for query variables
- Handle pagination with limit/offset
- Include all necessary fields in queries
- Example query pattern:
```graphql
query SomeQuery($search: String, $limit: Int, $offset: Int) {
    someQuery(search: $search, limit: $limit, offset: $offset) {
        items {
            id
            name
            email
            createdAt
        }
        total
    }
}
```

### Routing
- Use relative routing within modules
- Implement proper route resolvers for data loading
- Handle route parameters correctly
- Example routing pattern:
```typescript
const routes: Routes = [
    {
        path: "",
        component: ListComponent,
        resolve: { data: ListResolver },
        runGuardsAndResolvers: "paramsOrQueryParamsChange",
    },
    {
        path: "view/:id",
        component: ViewComponent,
        resolve: { data: ViewResolver },
    },
];
```

## Common Issues and Solutions

### PostgreSQL Query Issues
- Replace MySQL functions with PostgreSQL equivalents:
  - `CURDATE()` → `CURRENT_DATE`
  - `UNIX_TIMESTAMP()` → `EXTRACT(EPOCH FROM ...)`
  - `ANY_VALUE()` → `MIN()`
  - `MONTH()` → `EXTRACT(MONTH FROM ...)`
  - `YEAR()` → `EXTRACT(YEAR FROM ...)`

### Dependency Injection Errors
- Ensure all entities are included in `TypeOrmModule.forFeature()`
- Import required modules (e.g., `SharedOrderModule` for services)
- Check for circular dependencies

### GraphQL Type Errors
- Use DTOs instead of raw entities
- Ensure all fields are properly decorated with `@Field()`
- Handle nullable fields correctly

### Build Issues
- Run `flutter pub run build_runner build --delete-conflicting-outputs` after model changes
- Rebuild both backend and frontend after GraphQL schema changes
- Check for TypeScript compilation errors

## File Organization
- Keep related files together (component, template, styles, resolver, GraphQL)
- Use consistent naming conventions
- Separate concerns between list and detail views
- Example structure:
```
feature/
├── feature-list/
│   ├── feature-list.component.ts
│   ├── feature-list.component.html
│   ├── feature-list.component.css
│   ├── feature-list.resolver.ts
│   └── feature-list.graphql
├── feature-view/
│   ├── feature-view.component.ts
│   ├── feature-view.component.html
│   ├── feature-view.component.css
│   ├── feature-view.resolver.ts
│   └── feature-view.graphql
├── feature.module.ts
└── feature-routing.module.ts
```

## Testing and Debugging
- Add console logging for debugging routing issues
- Check browser console for GraphQL errors
- Verify route parameters and query parameters
- Test both backend and frontend builds
- Use proper error handling in resolvers and components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sanif) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
