## radroster

> You are developing a mobile-first radiation dose tracking application for nuclear industry supervisors and RPOs. This is a safety-critical application with strict regulatory compliance requirements.


# Radiation Dose Tracking App - AI Development Rules

## Project Context
You are developing a mobile-first radiation dose tracking application for nuclear industry supervisors and RPOs. This is a safety-critical application with strict regulatory compliance requirements.

## Technology Stack
- **Frontend**: React Native (iOS/Android)
- **Backend**: Node.js + Express
- **Database**: PostgreSQL with Sequelize ORM
- **Auth**: JWT with refresh tokens
- **Offline**: SQLite with sync capabilities
- **Notifications**: FCM (push) + Twilio (SMS)

## Code Standards

### File Structure
```
/project-root
├── /mobile-app/src/{components,screens,services,utils,hooks}
├── /backend/src/{controllers,models,routes,services,middleware,utils}
├── /migrations
└── /docs
```

### Naming Conventions
- **Components**: PascalCase (`DoseEntryForm.tsx`)
- **Functions/Variables**: camelCase (`calculateDoseExtrapolation()`)
- **Constants**: UPPERCASE_SNAKE_CASE (`ANNUAL_DOSE_LIMIT`)
- **Database**: snake_case (`dose_entries`)
- **Files**: kebab-case for utilities (`dose-calculator.ts`)

### Code Style
- **Indentation**: 2 spaces
- **Quotes**: Single quotes
- **Semicolons**: Required
- **Line Length**: 80 characters max
- **TypeScript**: Use for all new code

## Development Priorities

### Safety & Compliance First
- All dose calculations must be validated
- Implement proper error handling for critical operations
- Log all dose entries for audit trail
- Follow nuclear industry security standards

### Offline-First Architecture
- Design for offline operation
- Implement robust sync with conflict resolution
- Show sync status to users
- Handle network failures gracefully

### Performance Requirements
- API responses < 200ms
- App launch < 3 seconds
- Smooth 60fps animations
- Efficient database queries

## Database Schema Rules

### Core Tables
```sql
-- Workers: id, name, role, annual_limit_mSv, last_bioassay
-- Jobs: id, site, start_time, planned_duration_min, supervisor_id
-- DoseEntries: id, worker_id, job_id, timestamp, dose_mSv, source_instrument
-- TrainingRecords: id, worker_id, course_name, completion_date, expiry_date
-- Alerts: id, worker_id, type, triggered_at, resolved_at
```

### Data Types
- Use DECIMAL(8,4) for dose values (precision critical)
- Use UUID for all primary keys
- Include created_at/updated_at timestamps
- Use proper foreign key constraints

## API Design Patterns

### Authentication
```javascript
// JWT with refresh tokens
// Include role in claims for RBAC
// 15min access token, 7day refresh token
```

### Response Format
```javascript
// Success
{ "success": true, "data": {...}, "message": "..." }

// Error
{ "success": false, "error": { "code": "...", "message": "...", "details": [...] } }
```

### Endpoint Patterns
```javascript
GET    /api/{resource}           // List with pagination
GET    /api/{resource}/{id}      // Get single
POST   /api/{resource}           // Create
PUT    /api/{resource}/{id}      // Update
DELETE /api/{resource}/{id}      // Delete

// Custom endpoints
GET    /api/jobs/{id}/dose-summary
POST   /api/dose
GET    /api/alerts
PUT    /api/alerts/{id}/resolve
```

## React Native Guidelines

### Component Structure
```typescript
// Functional components with hooks
const DoseEntryForm: React.FC<DoseEntryFormProps> = ({ workerId, jobId }) => {
  const [dose, setDose] = useState<number>(0);
  const [loading, setLoading] = useState<boolean>(false);
  
  // Implementation
};
```

### Navigation
```typescript
// Use React Navigation v6
// Stack-based navigation
// Tab navigation for main sections
```

### State Management
```typescript
// Use Context API for global state
// Local state for component-specific data
// AsyncStorage for persistent data
```

### Offline Sync
```typescript
// SQLite for local storage
// Queue changes when offline
// Sync on connectivity restore
// Handle conflicts gracefully
```

## Backend Guidelines

### Express Structure
```typescript
// Controllers handle HTTP logic
// Services handle business logic
// Models handle data access
// Middleware for auth, validation, logging
```

### Validation
```typescript
// Use Joi or Zod for request validation
// Validate all inputs, especially dose values
// Return detailed error messages
```

### Error Handling
```typescript
// Use try-catch blocks
// Log errors appropriately
// Return user-friendly error messages
// Don't expose internal errors
```

## Testing Requirements

### Unit Tests
```typescript
// Test all service functions
// Mock external dependencies
// Test dose calculation logic thoroughly
// Aim for 80%+ coverage
```

### Integration Tests
```typescript
// Test API endpoints with real DB
// Test offline sync scenarios
// Test conflict resolution
```

## Security Guidelines

### Data Protection
- Encrypt sensitive health data at rest
- Use TLS for all communications
- Implement proper input sanitization
- Log all data modifications

### Authentication
- Strong password requirements
- Optional 2FA for supervisors
- Role-based access control
- Secure token storage

## Performance Guidelines

### Database
- Use indexes on frequently queried columns
- Implement connection pooling
- Optimize queries for dose calculations
- Use read replicas for reporting

### Mobile
- Implement lazy loading
- Use memoization for expensive calculations
- Optimize bundle size
- Cache frequently accessed data

## Development Workflow

### Git Strategy
- Feature branches from main
- Pull request reviews required
- Automated testing on all PRs
- Semantic versioning

### Code Quality
- ESLint with strict rules
- Prettier for formatting
- TypeScript for type safety
- Regular dependency updates

## When Writing Code

### Always Consider
1. **Safety**: Is this safe for nuclear industry use?
2. **Compliance**: Does this meet regulatory requirements?
3. **Offline**: Will this work without internet?
4. **Performance**: Is this optimized for field use?
5. **Usability**: Is this easy to use in field conditions?

### Code Examples
```typescript
// Good: Proper error handling
const logDose = async (doseData: DoseEntry): Promise<DoseEntry> => {
  try {
    // Validate dose value
    if (doseData.dose_mSv < 0 || doseData.dose_mSv > 1000) {
      throw new Error('Invalid dose value');
    }
    
    // Log for audit trail
    logger.info('Dose entry created', { workerId: doseData.worker_id, dose: doseData.dose_mSv });
    
    return await doseService.create(doseData);
  } catch (error) {
    logger.error('Failed to log dose', error);
    throw new Error('Failed to log dose entry');
  }
};

// Good: Offline-first component
const DoseEntryScreen: React.FC = () => {
  const [isOnline, setIsOnline] = useState(true);
  const [syncStatus, setSyncStatus] = useState<'synced' | 'pending' | 'failed'>('synced');
  
  // Implementation with offline handling
};
```

## Common Patterns

### Dose Calculation
```typescript
// Always validate inputs
// Use precise decimal arithmetic
// Log all calculations for audit
// Handle edge cases (zero time, negative values)
```

### Alert Generation
```typescript
// Check thresholds against limits
// Generate appropriate alert types
// Include context in alert messages
// Allow manual resolution
```

### Sync Logic
```typescript
// Queue changes when offline
// Resolve conflicts by timestamp
// Show sync status to user
// Handle partial sync failures
```

## Remember
- This is a safety-critical application
- Nuclear industry compliance is mandatory
- Offline operation is essential
- Performance matters in field conditions
- User experience should be simple and reliable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/boilerrat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
