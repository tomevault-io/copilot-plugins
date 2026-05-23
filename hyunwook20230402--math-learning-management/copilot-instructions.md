## database

> - **NEVER** use `user.id` (Supabase Auth) as foreign key

# 🗄️ Database & API Guidelines

## Database Architecture Rules

### ID Usage (CRITICAL)
- **NEVER** use `user.id` (Supabase Auth) as foreign key
- **ALWAYS** use `profile.id` for all foreign key references
- **ALWAYS** fetch profile first, then use `profile.id` for database operations

```typescript
// ✅ CORRECT: Get profile.id first
const { data: { user } } = await supabase.auth.getUser();
const { data: profile } = await supabase
  .from('profiles')
  .select('id')
  .eq('user_id', user.id)
  .single();

const problemData = {
  teacher_id: profile.id, // ✅ Use profile.id
  title: 'Problem Title'
};

// ❌ WRONG: Using user.id directly
const problemData = {
  teacher_id: user.id, // ❌ Foreign key constraint violation
  title: 'Problem Title'
};
```

### Database Schema Understanding

#### Core Tables
```sql
profiles (id, user_id, name, role, email)
├── problems (id, teacher_id, title, difficulty, category, unit)
├── problem_sets (id, teacher_id, name, description)
│   └── problem_set_items (problem_set_id, problem_id)
├── distributions (id, teacher_id, problem_set_id, due_date)
│   └── distribution_students (distribution_id, student_id)
├── student_answers (id, student_id, problem_id, answer, is_correct)
└── wrong_answers (id, student_id, problem_id, attempts)
```

#### Key Relationships
- **Teacher-Student**: Through `distributions` and `distribution_students`
- **Problem-Set**: Many-to-many through `problem_set_items`
- **Student-Answer**: One-to-many for tracking attempts

## API Service Patterns

### Standard API Structure
```typescript
export const domainApi = {
  // GET operations
  getItems: async (filters?: FilterType): Promise<ItemType[]> => {
    const { data, error } = await supabase
      .from('table_name')
      .select('*')
      .eq('field', value);
    
    if (error) throw error;
    return data || [];
  },

  // CREATE operations
  createItem: async (itemData: CreateItemType): Promise<ItemType> => {
    const { data, error } = await supabase
      .from('table_name')
      .insert(itemData)
      .select()
      .single();
    
    if (error) throw error;
    return data;
  },

  // UPDATE operations
  updateItem: async (id: string, updates: UpdateItemType): Promise<ItemType> => {
    const { data, error } = await supabase
      .from('table_name')
      .update(updates)
      .eq('id', id)
      .select()
      .single();
    
    if (error) throw error;
    return data;
  },

  // DELETE operations
  deleteItem: async (id: string): Promise<void> => {
    const { error } = await supabase
      .from('table_name')
      .delete()
      .eq('id', id);
    
    if (error) throw error;
  }
};
```

### Error Handling Patterns
```typescript
// Always include comprehensive error handling
const apiCall = async () => {
  try {
    console.log('Starting API call...');
    const result = await supabase.from('table').select('*');
    
    if (result.error) {
      console.error('Database error:', result.error);
      throw new Error(`Database operation failed: ${result.error.message}`);
    }
    
    console.log('API call successful:', result.data);
    return result.data;
  } catch (error) {
    console.error('API call failed:', error);
    throw error;
  }
};
```

## Data Validation

### Input Validation
```typescript
// Use Zod for schema validation
import { z } from 'zod';

const problemSchema = z.object({
  title: z.string().min(1, 'Title is required'),
  difficulty: z.number().min(1).max(5),
  category: z.string().min(1, 'Category is required'),
  unit: z.string().min(1, 'Unit is required')
});

// Validate before API calls
const createProblem = async (data: unknown) => {
  const validatedData = problemSchema.parse(data);
  return await problemApi.createProblem(validatedData);
};
```

### Type Safety
```typescript
// Define clear interfaces for database operations
interface CreateProblemData {
  teacher_id: string;
  title: string;
  problem_number: number;
  difficulty: number;
  category: string;
  unit: string;
  answer_type: 'multiple_choice' | 'short_answer';
  correct_answer: string;
  choices?: string[];
  explanation?: string;
  image_url?: string;
  explanation_image_url?: string;
}

interface Problem extends CreateProblemData {
  id: string;
  created_at: string;
  updated_at: string;
}
```

## Query Optimization

### Efficient Queries
```typescript
// Use specific select statements
const { data } = await supabase
  .from('problems')
  .select('id, title, difficulty, category') // Only select needed fields
  .eq('teacher_id', profileId)
  .order('created_at', { ascending: false })
  .limit(20);

// Use joins for related data
const { data } = await supabase
  .from('problem_sets')
  .select(`
    *,
    problems:problem_set_items(
      problem:problems(*)
    )
  `)
  .eq('teacher_id', profileId);
```

### Pagination
```typescript
// Implement pagination for large datasets
const getProblemsPaginated = async (
  page: number = 1, 
  pageSize: number = 20
) => {
  const from = (page - 1) * pageSize;
  const to = from + pageSize - 1;
  
  const { data, error, count } = await supabase
    .from('problems')
    .select('*', { count: 'exact' })
    .range(from, to)
    .order('created_at', { ascending: false });
  
  return {
    data: data || [],
    totalCount: count || 0,
    hasMore: (count || 0) > to + 1
  };
};
```

## Security Best Practices

### Row Level Security (RLS)
- RLS is currently disabled for development
- When enabling RLS, ensure proper policies are in place
- Test all operations with different user roles

### Data Sanitization
```typescript
// Sanitize user inputs
const sanitizeInput = (input: string): string => {
  return input.trim().replace(/[<>]/g, '');
};

// Validate file uploads
const validateImageFile = (file: File): boolean => {
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
  const maxSize = 5 * 1024 * 1024; // 5MB
  
  return allowedTypes.includes(file.type) && file.size <= maxSize;
};
```

## Common Patterns

### Profile Fetching
```typescript
// Standard pattern for getting current user's profile
const getCurrentProfile = async (): Promise<Profile | null> => {
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return null;

  const { data: profile, error } = await supabase
    .from('profiles')
    .select('*')
    .eq('user_id', user.id)
    .single();

  if (error) {
    console.error('Profile fetch error:', error);
    return null;
  }

  return profile;
};
```

### Batch Operations
```typescript
// For inserting multiple related items
const addProblemsToSet = async (problemSetId: string, problemIds: string[]) => {
  const items = problemIds.map(problemId => ({
    problem_set_id: problemSetId,
    problem_id: problemId
  }));

  const { data, error } = await supabase
    .from('problem_set_items')
    .insert(items);

  if (error) throw error;
  return data;
};
```

## Debugging Guidelines

### Logging Standards
```typescript
// Use consistent logging format
console.log('API call started:', { function: 'getProblems', teacherId });
console.log('API call completed:', { function: 'getProblems', resultCount: data.length });
console.error('API call failed:', { function: 'getProblems', error: error.message });
```

### Common Issues
1. **Foreign Key Violations**: Always use `profile.id`, never `user.id`
2. **RLS Policy Issues**: Check if RLS is enabled and policies are correct
3. **Type Mismatches**: Ensure data types match database schema
4. **Missing Dependencies**: Include all required fields in insert operations

---
> Source: [hyunwook20230402/math-learning-management](https://github.com/hyunwook20230402/math-learning-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
