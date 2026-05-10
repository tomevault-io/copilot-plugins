## trpc-implementation

> - **MUST:** Create typed context with Supabase client and user authentication

# tRPC Implementation Guide

## **Server Architecture Patterns**

### **Context Setup**
- **MUST:** Create typed context with Supabase client and user authentication
- **MUST:** Export context type for use in procedures

```typescript
// ✅ DO: Proper context setup with authentication
export async function createTRPCContext() {
  const supabase = await createClient();
  const { data: { user }, error: userError } = await supabase.auth.getUser();
  
  return {
    supabase,
    user,
    userError,
  };
}

export type Context = Awaited<ReturnType<typeof createTRPCContext>>;
```

### **Procedure Hierarchy**
- **MUST:** Define clear procedure hierarchy: `publicProcedure` → `protectedProcedure` → `adminProcedure`
- **MUST:** Use middleware for authentication and authorization checks
- **CRITICAL:** Ensure auth users exist in custom DB tables before allowing operations with foreign key constraints

```typescript
// ✅ DO: Layered procedure security with DB user synchronization
export const protectedProcedure = t.procedure.use(async (opts) => {
  const { user, supabase } = opts.ctx;
  
  if (!user) {
    throw new TRPCError({
      code: 'UNAUTHORIZED',
      message: '로그인이 필요합니다.',
    });
  }

  // CRITICAL: Ensure user exists in custom users table for foreign key integrity
  const { data: dbUser, error: userCheckError } = await supabase
    .from('users')
    .select('id')
    .eq('id', user.id)
    .single();

  if (userCheckError && userCheckError.code === 'PGRST116') {
    // User doesn't exist in users table, create it
    const { error: createError } = await supabase
      .from('users')
      .insert({
        id: user.id,
        email: user.email,
        name: user.user_metadata?.name || user.email?.split('@')[0],
        auth_provider: user.app_metadata?.provider || 'email',
      });

    if (createError) {
      console.error('Failed to create user record:', createError);
      throw new TRPCError({
        code: 'INTERNAL_SERVER_ERROR',
        message: '사용자 정보 생성 중 오류가 발생했습니다.',
      });
    }
  } else if (userCheckError) {
    console.error('User check error:', userCheckError);
    throw new TRPCError({
      code: 'INTERNAL_SERVER_ERROR',
      message: '사용자 확인 중 오류가 발생했습니다.',
    });
  }
  
  return opts.next({
    ctx: { ...opts.ctx, user },
  });
});
```

### **Auth-DB Synchronization Rules**
- **CRITICAL:** Always ensure Supabase Auth users have corresponding records in custom users table
- **MUST:** Handle the gap between auth.users and custom users table in protectedProcedure
- **MUST:** Auto-create missing user records when foreign key constraints require them
- **MUST NOT:** Assume auth user existence guarantees DB user record existence

```typescript
// ❌ DON'T: Assume auth user exists in custom tables
export const badProtectedProcedure = t.procedure.use(async (opts) => {
  const { user } = opts.ctx;
  if (!user) throw new TRPCError({ code: 'UNAUTHORIZED' });
  
  // This will fail if user doesn't exist in custom users table
  return opts.next({ ctx: { ...opts.ctx, user } });
});

// ✅ DO: Verify and sync user records before operations
// See the improved protectedProcedure example above
```

## **Domain Router Structure**

### **Router Organization**
- **MUST:** Separate routers by domain ([template.ts](mdc:src/lib/trpc/routers/template.ts), [version.ts](mdc:src/lib/trpc/routers/version.ts))
- **MUST:** Group related procedures within domain routers
- **MUST:** Export domain routers and combine in [root.ts](mdc:src/lib/trpc/root.ts)

```typescript
// ✅ DO: Domain-specific router structure
export const templateRouter = router({
  getAll: publicProcedure.query(/* ... */),
  getById: publicProcedure.input(/* ... */).query(/* ... */),
  create: protectedProcedure.input(/* ... */).mutation(/* ... */),
  update: protectedProcedure.input(/* ... */).mutation(/* ... */),
  delete: protectedProcedure.input(/* ... */).mutation(/* ... */),
  getMine: protectedProcedure.query(/* ... */),
});
```

### **Procedure Naming Conventions**
- **MUST:** Use clear, RESTful naming: `getAll`, `getById`, `create`, `update`, `delete`
- **MUST:** Add domain-specific procedures like `getMine`, `restore`, `compare`
- **MUST NOT:** Mix CRUD and business logic procedures in unclear names

## **Input Validation & Schemas**

### **Zod Schema Patterns**
- **MUST:** Define schemas at the top of router files
- **MUST:** Use descriptive error messages in Korean for user-facing errors
- **MUST:** Separate input/output schemas for different operations

```typescript
// ✅ DO: Clear schema definition with localized errors
const createTemplateSchema = z.object({
  name: z.string().min(1, '템플릿 이름은 필수입니다'),
  description: z.string().optional(),
  content: z.string().min(1, '템플릿 내용은 필수입니다'),
  fields: z.array(templateFieldSchema).optional(),
});

const templateIdSchema = z.object({
  id: z.string().uuid(),
});
```

### **Schema Reusability**
- **MUST:** Extract common schemas (like ID validation) for reuse
- **MUST:** Build complex schemas from simpler base schemas
- **MUST NOT:** Duplicate validation logic across procedures

## **Error Handling Patterns**

### **Consistent Error Structure**
- **MUST:** Use TRPCError with appropriate HTTP status codes
- **MUST:** Provide user-friendly Korean error messages
- **MUST:** Log detailed errors for debugging while hiding sensitive information

```typescript
// ✅ DO: Consistent error handling
try {
  // Database operation
} catch (error) {
  console.error('템플릿 조회 오류:', error);
  throw new TRPCError({
    code: 'INTERNAL_SERVER_ERROR',
    message: '템플릿을 불러오는 중 오류가 발생했습니다.',
    cause: error, // Include for debugging (not exposed to client)
  });
}
```

### **Error Code Mapping**
- **MUST:** Use correct tRPC error codes:
  - `UNAUTHORIZED`: Authentication required
  - `FORBIDDEN`: Access denied
  - `NOT_FOUND`: Resource doesn't exist
  - `INTERNAL_SERVER_ERROR`: Server/database errors
  - `BAD_REQUEST`: Invalid input

## **Security & Authorization**

### **Ownership Validation**
- **MUST:** Check resource ownership before modification operations
- **MUST:** Use consistent ownership checking pattern

```typescript
// ✅ DO: Ownership validation pattern
const { data: existingTemplate, error: checkError } = await supabase
  .from('templates')
  .select('created_by')
  .eq('id', input.id)
  .single();

if (checkError || !existingTemplate) {
  throw new TRPCError({
    code: 'NOT_FOUND',
    message: '템플릿을 찾을 수 없습니다.',
  });
}

if (existingTemplate.created_by !== user.id) {
  throw new TRPCError({
    code: 'FORBIDDEN',
    message: '본인이 생성한 템플릿만 수정할 수 있습니다.',
  });
}
```

### **Database Transaction Patterns**
- **MUST:** Use transactions for multi-table operations
- **MUST:** Implement cleanup on partial failures

```typescript
// ✅ DO: Transaction with cleanup on failure
const { data: template, error: templateError } = await supabase
  .from('templates')
  .insert(templateData)
  .select()
  .single();

if (templateError) {
  throw new TRPCError({ /* ... */ });
}

// If fields creation fails, clean up the created template
if (fieldsError) {
  await supabase.from('templates').delete().eq('id', template.id);
  throw new TRPCError({ /* ... */ });
}
```

## **Database Integration**

### **Supabase Query Patterns**
- **MUST:** Use select with specific fields for performance
- **MUST:** Include related data with join syntax
- **MUST:** Apply RLS (Row Level Security) through user context

```typescript
// ✅ DO: Efficient query with relations
const { data: templates, error } = await supabase
  .from('templates')
  .select(`
    *,
    template_fields(*)
  `)
  .eq('is_active', true)
  .order('created_at', { ascending: false });
```

### **Database Function Usage**
- **MUST:** Use database functions for complex operations like versioning
- **MUST:** Handle function errors appropriately

```typescript
// ✅ DO: Database function with error handling
const { data: versionResult, error: versionError } = await supabase
  .rpc('create_template_version', {
    p_template_id: input.template_id,
    p_name: input.name,
    p_description: input.description || null,
    p_content: input.content,
    p_change_summary: input.change_summary,
    p_created_by: user.id,
  });
```

### **Foreign Key Integrity Rules**
- **CRITICAL:** Always verify referenced records exist before inserting/updating with foreign keys
- **MUST:** Handle foreign key constraint violations gracefully with meaningful error messages
- **MUST:** Use transactions when inserting records with dependencies
- **MUST NOT:** Assume auth user IDs automatically exist in custom tables

```typescript
// ❌ DON'T: Insert without verifying foreign key references
const createTemplate = protectedProcedure
  .input(createTemplateSchema)
  .mutation(async ({ input, ctx }) => {
    // This will fail if user.id doesn't exist in users table
    const { data, error } = await ctx.supabase
      .from('templates')
      .insert({
        ...input,
        created_by: ctx.user.id, // Foreign key constraint violation risk
      });
  });

// ✅ DO: Ensure foreign key references are valid
const createTemplate = protectedProcedure
  .input(createTemplateSchema)
  .mutation(async ({ input, ctx }) => {
    // protectedProcedure already ensures user exists in users table
    // But for other foreign keys, verify they exist
    
    try {
      const { data, error } = await ctx.supabase
        .from('templates')
        .insert({
          ...input,
          created_by: ctx.user.id, // Safe due to protectedProcedure sync
        })
        .select()
        .single();

      if (error) {
        // Handle foreign key constraint violations specifically
        if (error.code === '23503') {
          throw new TRPCError({
            code: 'BAD_REQUEST',
            message: '참조된 데이터가 존재하지 않습니다.',
          });
        }
        throw error;
      }

      return data;
    } catch (error) {
      console.error('Template creation error:', error);
      throw new TRPCError({
        code: 'INTERNAL_SERVER_ERROR',
        message: '템플릿 생성 중 오류가 발생했습니다.',
      });
    }
  });
```

## **Client Integration**

### **Provider Setup**
- **MUST:** Wrap tRPC Provider around QueryClientProvider in [providers.tsx](mdc:src/app/providers.tsx)
- **MUST:** Share QueryClient instance between tRPC and React Query

```typescript
// ✅ DO: Proper provider hierarchy
return (
  <trpc.Provider client={trpcClient} queryClient={queryClient}>
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>
        {children}
      </ThemeProvider>
    </QueryClientProvider>
  </trpc.Provider>
);
```

### **API Route Handler**
- **MUST:** Use fetchRequestHandler in [route.ts](mdc:src/app/api/trpc/[trpc]/route.ts)
- **MUST:** Include error logging for development

```typescript
// ✅ DO: Proper API route setup
const handler = (req: NextRequest) =>
  fetchRequestHandler({
    endpoint: '/api/trpc',
    req,
    router: appRouter,
    createContext: createTRPCContext,
    onError: process.env.NODE_ENV === 'development'
      ? ({ path, error }) => {
          console.error(`❌ tRPC failed on ${path ?? "<no-path>"}: ${error.message}`);
        }
      : undefined,
  });
```

## **Performance Considerations**

### **Query Optimization**
- **MUST:** Use appropriate indexes for frequent queries
- **MUST:** Limit data selection to required fields
- **MUST NOT:** Return entire objects when partial data suffices

### **Caching Strategy**
- **SHOULD:** Leverage React Query's built-in caching
- **SHOULD:** Set appropriate staleTime for different data types
- **MUST NOT:** Disable caching without performance justification

## **Common Anti-Patterns**

```typescript
// ❌ DON'T: Mix unrelated procedures in one router
export const mixedRouter = router({
  createTemplate: /* ... */,
  sendEmail: /* ... */, // Wrong domain
  calculateTax: /* ... */, // Wrong domain
});

// ❌ DON'T: Expose raw database models
.mutation(async ({ ctx }) => {
  return await ctx.supabase.from('templates').select('*'); // Too much data
});

// ❌ DON'T: Skip ownership validation
.mutation(async ({ ctx, input }) => {
  // Missing ownership check
  return await ctx.supabase
    .from('templates')
    .update(input)
    .eq('id', input.id);
});

// ❌ DON'T: Silent error handling
.mutation(async ({ ctx }) => {
  try {
    // operation
  } catch (error) {
    // Silent failure - no logging or user notification
  }
});

// ❌ DON'T: Ignore auth-DB synchronization
export const badProtectedProcedure = t.procedure.use(async (opts) => {
  const { user } = opts.ctx;
  if (!user) throw new TRPCError({ code: 'UNAUTHORIZED' });
  
  // This assumes user exists in custom tables - WRONG!
  return opts.next({ ctx: { ...opts.ctx, user } });
});

// ❌ DON'T: Insert with foreign keys without verification
.mutation(async ({ ctx, input }) => {
  // This will cause foreign key constraint violation
  return await ctx.supabase
    .from('templates')
    .insert({
      ...input,
      created_by: ctx.user.id, // Might not exist in users table
    });
});
```

---
> Source: [greatSumini/document-parser](https://github.com/greatSumini/document-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
