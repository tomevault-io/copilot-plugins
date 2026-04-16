## imperial

> Feature module patterns and conventions for backend development


# Feature Module Patterns

## Standard Feature Structure
Each feature in `feat/` should follow this consistent pattern:

```
feat/[feature-name]/
├── [feature].controller.ts    # HTTP request/response handling
├── [feature].service.ts       # Business logic implementation
├── [feature].route.ts         # Express route definitions
├── [feature].validation.ts    # Request validation schemas
├── [feature].repository.ts    # Data access layer
├── [feature].error.ts         # Feature-specific error definitions
├── [feature].test.ts          # Feature tests
└── common/                    # Shared feature utilities
    ├── [feature].middleware.ts
    ├── [feature].schema.ts
    └── [feature].types.ts
```

## Controller Pattern
Controllers handle HTTP requests and delegate to services:

```typescript
export class FeatureController {
  constructor(private featureService: FeatureService) {}

  async handleRequest(req: Request, res: Response, next: NextFunction) {
    try {
      // 1. Validate request
      const validatedData = featureValidationSchema.parse(req.body);
      
      // 2. Call service
      const result = await this.featureService.processData(validatedData);
      
      // 3. Return standardized response
      res.json({
        success: true,
        message: "Operation completed successfully",
        data: result
      });
    } catch (error) {
      next(error); // Let error middleware handle it
    }
  }
}
```

## Service Pattern
Services contain business logic and coordinate between repositories:

```typescript
export class FeatureService {
  constructor(
    private featureRepository: FeatureRepository,
    private externalService: ExternalService
  ) {}

  async processData(data: ValidatedData) {
    // 1. Business logic validation
    if (data.condition) {
      throw new AppError(FeatureErrorDefinitions.INVALID_CONDITION, {});
    }
    
    // 2. Coordinate data operations
    const existing = await this.featureRepository.findByCondition(data.condition);
    const processed = await this.externalService.transform(existing);
    
    // 3. Persist changes
    return await this.featureRepository.save(processed);
  }
}
```

## Repository Pattern
Repositories handle data access and database operations:

```typescript
export const createFeature = async (data: CreateFeatureData) => {
  return await db.insert(featureTable).values(data).returning();
};

export const findFeatureById = async (id: number) => {
  const result = await db.select().from(featureTable).where(eq(featureTable.id, id));
  return result[0] || null;
};
```

## Route Pattern
Routes define endpoints and connect to controllers:

```typescript
const router = express.Router();

// Apply authentication middleware
router.use(authenticateJWT);

// Define routes with validation
router.post('/endpoint', 
  validateRequest(featureValidationSchema),
  featureController.handleRequest
);

export const featureRoutes = router;
```

## Error Definition Pattern
Define feature-specific errors with consistent structure:

```typescript
export const FeatureErrorDefinitions = {
  INVALID_CONDITION: {
    code: 'INVALID_CONDITION',
    message: 'The provided condition is not valid',
    httpStatus: 400,
    detailsSchema: z.object({
      field: z.string().optional()
    })
  },
  NOT_FOUND: {
    code: 'FEATURE_NOT_FOUND',
    message: 'Feature not found',
    httpStatus: 404,
    detailsSchema: z.object({})
  }
} as const satisfies Record<string, ErrorDefinition>;
```

## Validation Schema Pattern
Use Zod for request/response validation:

```typescript
export const createFeatureSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().optional(),
  type: z.enum(['TYPE_A', 'TYPE_B']),
  metadata: z.record(z.string(), z.unknown()).optional()
});

export type CreateFeatureData = z.infer<typeof createFeatureSchema>;
```

## Testing Pattern
Follow the established testing patterns:

```typescript
describe('FeatureController', () => {
  beforeEach(async () => {
    // Setup test data
    await setupTestData();
  });

  describe('POST /api/feature', () => {
    it('should create feature successfully', async () => {
      const response = await request(app)
        .post('/api/feature')
        .send(validFeatureData)
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(response.body.data).toMatchObject(expectedResult);
    });

    it('should reject invalid data', async () => {
      await request(app)
        .post('/api/feature')
        .send(invalidData)
        .expect(400);
    });
  });
});
```

## Chat Feature Special Patterns

### LangGraph Node Structure
Chat feature uses modular nodes in `nodes/` directory:

```typescript
export const featureNode = async (state: GraphStateType) => {
  // 1. Extract state data
  const { input, previousComponent } = state;
  
  // 2. Process with shared services
  const result = await llmService.process(input.userMessage);
  
  // 3. Return updated state
  return {
    output: {
      ...state.output,
      processedData: result
    }
  };
};
```

### Shared Services Pattern
Use singleton pattern for shared services:

```typescript
class SharedService {
  private static instance: SharedService;
  
  static getInstance(): SharedService {
    if (!this.instance) {
      this.instance = new SharedService();
    }
    return this.instance;
  }
  
  async processData(data: any) {
    // Service implementation
  }
}

export const sharedService = SharedService.getInstance();
```

## Key Conventions

1. **File Naming**: Use kebab-case for files, PascalCase for classes
2. **Error Handling**: Always use `AppError` with defined error codes
3. **Validation**: Validate all inputs with Zod schemas
4. **Testing**: Write tests for controllers, services, and repositories
5. **Documentation**: Include JSDoc comments for complex business logic
6. **Type Safety**: Use TypeScript strictly, avoid `any` types
7. **Async/Await**: Prefer async/await over promises
8. **Dependency Injection**: Pass dependencies through constructors
9. **Single Responsibility**: Each module should have one clear purpose
10. **Error Codes**: Use descriptive, consistent error codes across features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/himanshunegi378) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
