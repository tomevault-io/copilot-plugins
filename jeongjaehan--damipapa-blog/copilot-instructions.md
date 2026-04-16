## damipapa-blog

> This is a full-stack blog platform with:

# Cursor Rules for Blog Platform Project

## Project Context
This is a full-stack blog platform with:
- Backend: Spring Boot 3.2.1 (Java 17) + H2 Database
- Frontend: Next.js 15.5.4 (TypeScript) + Tailwind CSS
- Auth: JWT token-based authentication
- Features: Blog posts, comments, tags, categories, image upload, view count

## General Rules

### Code Style
- Follow Google Java Style Guide for backend
- Follow Airbnb TypeScript Style Guide for frontend
- Use meaningful variable and function names in Korean or English
- Prefer composition over inheritance
- Keep functions small and focused (max 50 lines)

### TypeScript/JavaScript
- Always use TypeScript strict mode
- Prefer `const` over `let`, avoid `var`
- Use async/await instead of Promise chains
- Always handle errors with try-catch
- Use optional chaining (?.) and nullish coalescing (??)
- Client components must use 'use client' directive
- Wrap useSearchParams in Suspense boundary

### Java/Spring Boot
- Use Lombok annotations (@Data, @Builder, @Slf4j)
- Always use @Transactional for service methods
- Prefer constructor injection over field injection
- Use Optional for nullable returns
- Log important operations with SLF4J
- Follow REST API naming conventions

### React/Next.js Specific
- All API-calling pages should be client components ('use client')
- Use Suspense for components using useSearchParams
- Params in Next.js 15 should be awaited (Promise type)
- Use useParams hook for dynamic routes in client components
- Always add loading states for async operations

### Database
- Use H2 in-memory database for development
- Always use JPA annotations correctly
- Add @CreatedDate and @LastModifiedDate for audit fields
- Use @Builder.Default for default values
- Create indexes for frequently queried columns

### Security
- Never expose sensitive data in responses (passwords, secrets)
- Always validate user input with Bean Validation
- Use @PreAuthorize for role-based access control
- Sanitize HTML content before rendering
- Check IP address for unique view counting

### API Design
- Use RESTful conventions (GET, POST, PUT, DELETE)
- Return appropriate HTTP status codes
- Use PageResponse for paginated data
- Include error codes and messages in error responses
- Support CORS for localhost:3000 and localhost:3001

### UI/UX
- All input/textarea must have visible text (color: #111827, background: #fff)
- Use Tailwind CSS classes instead of custom CSS when possible
- Ensure responsive design (mobile-first)
- Show loading indicators for async operations
- Provide clear error messages to users

### File Upload
- Maximum file size: 5MB
- Allowed formats: jpg, jpeg, png, gif, webp
- Use UUID for filenames to avoid conflicts
- Store in ./uploads directory (volume-mounted in Docker)

### Authentication
- JWT token expires in 30 minutes
- Store token in localStorage (client-side)
- Auto-redirect to login on 401 errors
- Check admin role for protected routes

## Project-Specific Patterns

### Backend Service Pattern
```java
@Service
@Transactional
@Slf4j
public class XxxService {
    private final XxxRepository repository;
    
    public XxxService(XxxRepository repository) {
        this.repository = repository;
    }
    
    @Transactional(readOnly = true)
    public XxxResponse getXxx(Long id) {
        log.info("Getting xxx: {}", id);
        Xxx xxx = repository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("Xxx", id));
        return XxxResponse.fromEntity(xxx);
    }
}
```

### Frontend API Pattern
```typescript
export const getXxx = async (id: number): Promise<Xxx> => {
  const response = await api.get<Xxx>(`/xxx/${id}`)
  return response.data
}
```

### Frontend Page Pattern (Client Component)
```typescript
'use client'

import { useEffect, useState } from 'react'
import { getXxx } from '@/services/api'
import Loading from '@/components/common/Loading'

export default function XxxPage() {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)
  
  useEffect(() => {
    const loadData = async () => {
      try {
        const result = await getXxx()
        setData(result)
      } catch (error) {
        console.error('Error:', error)
      } finally {
        setLoading(false)
      }
    }
    loadData()
  }, [])
  
  if (loading) return <Loading />
  
  return <div>{/* content */}</div>
}
```

## Docker Commands

### Development
```bash
# Start all services
docker-compose up -d

# Rebuild and start
docker-compose up --build -d

# View logs
docker logs blog-backend -f
docker logs blog-frontend -f

# Stop services
docker-compose down

# Clean rebuild (removes volumes)
docker-compose down -v && docker-compose up --build -d
```

## Common Issues and Solutions

### Issue: Input text not visible
**Solution**: Add explicit color styles to all input/textarea elements
```css
input, textarea {
  color: #111827 !important;
  background-color: #ffffff !important;
}
```

### Issue: Next.js 15 params error
**Solution**: Use Promise type for params in server components, or use useParams in client components
```typescript
// Server component
async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
}

// Client component
'use client'
function Page() {
  const params = useParams()
  const id = params.id
}
```

### Issue: useSearchParams error
**Solution**: Wrap in Suspense
```typescript
function Content() {
  const searchParams = useSearchParams()
  // ...
}

export default function Page() {
  return (
    <Suspense fallback={<Loading />}>
      <Content />
    </Suspense>
  )
}
```

### Issue: Image upload button not working
**Solution**: Use useRef instead of label wrapper
```typescript
const fileInputRef = useRef<HTMLInputElement>(null)

<input ref={fileInputRef} type="file" className="hidden" />
<Button onClick={() => fileInputRef.current?.click()}>Upload</Button>
```

## Testing Checklist

Before considering a feature complete:
- [ ] API endpoint responds correctly
- [ ] Frontend displays data properly
- [ ] Error cases are handled
- [ ] Loading states are shown
- [ ] Input fields are visible
- [ ] Mobile responsive
- [ ] Admin permissions checked
- [ ] Logs are added for debugging

## Notes
- H2 database resets on container restart - this is intentional for development
- View count uses IP address for uniqueness - may not work perfectly behind proxies
- Image uploads are stored in container volume - will be lost on volume deletion
- All external API calls (spell check) were removed due to CORS/403 issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeongjaehan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
