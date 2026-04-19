## proj-spbr-booking

> - **Always use PostgreSQL syntax** - this is a Supabase project

# 🎯 Cursor Rules for San Pedro Beach Resort Project

## 🗄️ Database & PostgreSQL/Supabase Rules

### Database Schema & Migration Rules

- **Always use PostgreSQL syntax** - this is a Supabase project
- **Create migrations in `docs/` folder** with descriptive names: `YYYY-MM-DD_description.sql`
- **Use snake_case** for all table names, column names, and function names
- **Always include `id UUID DEFAULT gen_random_uuid() PRIMARY KEY`** for new tables
- **Include `created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()`** and `updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()`** in all tables
- **Use appropriate data types**: 
  - `UUID` for IDs and foreign keys
  - `TEXT` for strings (not VARCHAR)
  - `NUMERIC(10,2)` for currency amounts
  - `BOOLEAN` for true/false values
  - `JSONB` for complex data structures
  - `TIMESTAMP WITH TIME ZONE` for dates

### Row Level Security (RLS) Rules
- **Enable RLS on ALL tables** with `ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;`
- **Create policies for each operation** (SELECT, INSERT, UPDATE, DELETE)
- **Use role-based access control** with these roles: 'admin', 'manager', 'employee', 'guest'
- **Always check user authentication** with `auth.uid()` in policies
- **Create policies in this order**:
  1. SELECT policies (read access)
  2. INSERT policies (create access)
  3. UPDATE policies (modify access)
  4. DELETE policies (remove access)

### Supabase-Specific Rules
- **Use Supabase client** from `@/lib/supabase` for all database operations
- **Never expose service role key** in client-side code
- **Use environment variables** for all Supabase configuration
- **Implement proper error handling** for all Supabase operations
- **Use real-time subscriptions** only when necessary
- **Enable proper CORS settings** in Supabase dashboard

### Database Naming Conventions
- **Tables**: `snake_case` (e.g., `user_profiles`, `rental_units_pricing`)
- **Columns**: `snake_case` (e.g., `first_name`, `check_in_date`)
- **Functions**: `snake_case` (e.g., `get_user_role`, `calculate_total`)
- **Policies**: Descriptive names (e.g., "Users can view own profile")
- **Indexes**: `idx_table_column` (e.g., `idx_user_profiles_role`)

## 🎨 Frontend & React/Next.js Rules

### Component Structure
- **Use TypeScript** for all components and functions
- **Create components in `src/components/`** with PascalCase naming
- **Use functional components** with hooks, not class components
- **Implement proper prop types** with TypeScript interfaces
- **Use `'use client'`** directive for client-side components
- **Keep components small and focused** on single responsibility

### Styling Rules
- **Use Tailwind CSS** for all styling
- **Follow the project's color scheme**: green and yellow theme
- **Use semantic class names** and avoid custom CSS when possible
- **Implement responsive design** with mobile-first approach
- **Use consistent spacing** with Tailwind's spacing scale
- **Apply dark theme** with proper contrast ratios

### State Management
- **Use React hooks** (useState, useEffect, useContext) for state
- **Implement proper loading states** for all async operations
- **Handle errors gracefully** with user-friendly messages
- **Use optimistic updates** where appropriate
- **Implement proper form validation** with error states

## 🔐 Security Rules

### Authentication & Authorization
- **Always check user permissions** before sensitive operations
- **Implement role-based access control** in both frontend and backend
- **Use secure session management** with proper token handling
- **Validate all user inputs** on both client and server side
- **Implement proper logout functionality** with token cleanup

### Data Protection
- **Never store sensitive data** in localStorage or sessionStorage
- **Use environment variables** for all API keys and secrets
- **Implement proper data sanitization** for all user inputs
- **Use HTTPS** for all external communications
- **Implement rate limiting** for API endpoints

## 📁 File Organization Rules

### Project Structure
```
src/
├── app/                 # Next.js app router pages
├── components/          # Reusable React components
├── lib/                 # Utility functions and configurations
├── types/               # TypeScript type definitions
└── hooks/               # Custom React hooks
```

### File Naming Conventions
- **Components**: `PascalCase.tsx` (e.g., `UserProfile.tsx`)
- **Pages**: `page.tsx` (Next.js app router)
- **Utilities**: `camelCase.ts` (e.g., `formatCurrency.ts`)
- **Types**: `camelCase.ts` (e.g., `userTypes.ts`)
- **Constants**: `UPPER_SNAKE_CASE.ts` (e.g., `API_ENDPOINTS.ts`)

## 🧪 Testing & Quality Rules

### Code Quality
- **Use ESLint** for code linting and formatting
- **Follow TypeScript strict mode** guidelines
- **Implement proper error boundaries** for React components
- **Use meaningful variable and function names**
- **Add JSDoc comments** for complex functions
- **Keep functions small** (max 20-30 lines)

### Performance Rules
- **Use React.memo()** for expensive components
- **Implement proper loading states** and skeleton screens
- **Use Next.js Image component** for optimized images
- **Implement proper caching strategies** for API calls
- **Use code splitting** for large components

## 📊 Data Import & CSV Rules

### CSV File Handling
- **Store CSV files in `public/csv-imports/`** directory
- **Use descriptive filenames** (e.g., `employees_2025.csv`)
- **Include headers** in all CSV files
- **Use UTF-8 encoding** for international characters
- **Validate CSV data** before importing to database
- **Create import scripts** in `scripts/` directory

### Data Validation Rules
- **Validate all required fields** before database insertion
- **Check data types** match database schema
- **Handle missing or null values** appropriately
- **Implement data transformation** for format consistency
- **Log all import operations** for audit purposes

## 🚀 Deployment & Environment Rules

### Environment Configuration
- **Use `.env.local`** for local development
- **Set up Vercel environment variables** for production
- **Never commit sensitive data** to version control
- **Use different configurations** for dev/staging/prod
- **Implement proper logging** for debugging

### Build & Deployment
- **Test builds locally** before deployment
- **Use proper Node.js version** (>=20 for this project)
- **Optimize bundle size** with proper imports
- **Implement proper error handling** for production
- **Use Vercel for deployment** with automatic builds

## 📝 Documentation Rules

### Code Documentation
- **Add README.md** for all major components
- **Document API endpoints** with proper examples
- **Include setup instructions** for new developers
- **Document database schema** with ERD diagrams
- **Maintain changelog** for version updates

### Database Documentation
- **Document all tables** with purpose and relationships
- **Include sample data** for testing
- **Document RLS policies** with access rules
- **Maintain data dictionary** for all fields
- **Document migration procedures**

## 🔧 Development Workflow Rules

### Git & Version Control
- **Use descriptive commit messages** with conventional commits
- **Create feature branches** for new development
- **Review code** before merging to main
- **Keep commits atomic** and focused
- **Use proper branch naming** (feature/, bugfix/, hotfix/)

### Code Review Guidelines
- **Check for security vulnerabilities** in all code
- **Verify database queries** are optimized
- **Test responsive design** on multiple devices
- **Validate accessibility** standards
- **Ensure proper error handling** is implemented

## 🎯 Project-Specific Rules

### San Pedro Beach Resort Features
- **Implement booking system** with proper validation
- **Create inventory management** with stock tracking
- **Build payment processing** with secure handling
- **Design user-friendly interfaces** for staff
- **Implement reporting features** for management

### Business Logic Rules
- **Validate booking dates** (no past dates, proper duration)
- **Calculate pricing** based on room type and duration
- **Track inventory levels** with automatic alerts
- **Handle payment processing** with proper receipts
- **Implement guest management** with proper data protection

## 🚨 Error Handling Rules

### Database Errors
- **Handle connection errors** gracefully
- **Implement retry logic** for transient failures
- **Log all database errors** with context
- **Provide user-friendly error messages**
- **Implement fallback mechanisms** where possible

### Frontend Errors
- **Use error boundaries** for React components
- **Implement proper loading states** during errors
- **Show user-friendly error messages**
- **Log errors** for debugging purposes
- **Implement offline handling** where appropriate

## 📱 Responsive Design Rules

### Mobile-First Approach
- **Design for mobile** first, then enhance for desktop
- **Use Tailwind responsive classes** consistently
- **Test on multiple screen sizes** during development
- **Implement touch-friendly** interface elements
- **Optimize images** for different screen densities

### Accessibility Rules
- **Use semantic HTML** elements
- **Implement proper ARIA labels** for screen readers
- **Ensure proper color contrast** ratios
- **Add keyboard navigation** support
- **Test with accessibility tools**

## 🔄 API & Integration Rules

### REST API Design
- **Use consistent URL patterns** (/api/resource)
- **Implement proper HTTP status codes**
- **Use JSON for all responses**
- **Include proper error responses**
- **Implement rate limiting** and pagination

### External Integrations
- **Use environment variables** for API keys
- **Implement proper error handling** for external APIs
- **Add retry logic** for failed requests
- **Cache responses** where appropriate
- **Monitor API usage** and costs

---

## 🎯 When Creating New Rules

### Rule Creation Guidelines
- **Be specific and actionable** - avoid vague instructions
- **Include examples** where helpful
- **Consider security implications** of all rules
- **Test rules** before implementing
- **Update rules** as project evolves
- **Document rule rationale** for team understanding

### Rule Maintenance
- **Review rules regularly** for relevance
- **Update rules** based on project changes
- **Remove outdated rules** to avoid confusion
- **Add new rules** as patterns emerge
- **Share rule updates** with the team

---

**Remember**: These rules are designed to ensure consistency, security, and maintainability across the San Pedro Beach Resort project. Always prioritize user experience, data security, and code quality in all development decisions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nxgdevllc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
