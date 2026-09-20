## incremental-development

> Incremental development approach with always-working builds and controlled changes


# Incremental Development Strategy

## Core Development Principles

### 1. **Always-Working Builds**

- `pnpm run dev` must always work
- Never break the development server
- Incremental changes only
- Test each change before proceeding

### 2. **Controlled Development**

- One feature at a time
- Small, focused commits
- Easy rollback capability
- Clear progress tracking

### 3. **Domain Strategy**

- **Primary Domain**: everprompt.ai
- **Development**: localhost:3000
- **Staging**: staging.everprompt.ai (future)
- **Production**: everprompt.ai

## Development Workflow

### **Git Strategy for Solo Developer**

#### **Primary: Main Branch Development**

```bash
# Work directly on main branch
git checkout main
# Make incremental changes
git add .
git commit -m "feat: add dark/light mode toggle"
git push origin main
```

#### **Rollback Points Strategy**

```bash
# Create rollback points after each working feature
git tag v0.1.0-working-basic-layout
git tag v0.2.0-working-theme-toggle
git tag v0.3.0-working-prompt-editor

# Rollback if needed
git checkout v0.2.0-working-theme-toggle
```

#### **Feature Branches (Only for Major Features)**

```bash
# Only for complex features or experiments
git checkout -b feature/prompt-editor
# Make changes
git add .
git commit -m "feat: implement prompt editor"
git push origin feature/prompt-editor
# Merge when ready
git checkout main
git merge feature/prompt-editor
git push origin main
```

#### **When to Use Feature Branches:**

- **Major experiments** (e.g., trying a new UI approach)
- **Complex features** that might break the app
- **Integration work** (e.g., adding authentication)
- **Refactoring** that affects multiple files
- **When you're unsure** if the approach will work

#### **When to Use Main Branch:**

- **Small, incremental changes**
- **UI tweaks and improvements**
- **Bug fixes**
- **Documentation updates**
- **Configuration changes**
- **Most development work**

### **Phase 1: Foundation (Always Working)**

```bash
# Start with basic Next.js app
pnpm run dev  # Must work immediately

# Incremental changes:
1. Basic layout ✅
2. Dark/light mode toggle ✅
3. Simple prompt editor ✅
4. Basic label system ✅
5. n8n JSON parser ✅
```

### **Phase 2: Core Features (One at a Time)**

```bash
# Each step must work before next:
1. Authentication (Clerk) ✅
2. Database setup (Neon + Prisma) ✅
3. Prompt CRUD operations ✅
4. Label management ✅
5. Workflow collection system ✅
```

### **Phase 3: Advanced Features (Controlled)**

```bash
# Build on working foundation:
1. Search and filtering ✅
2. Public sharing ✅
3. Payment integration (Stripe) ✅
4. API endpoints ✅
5. Mobile optimization ✅
```

## Commit Strategy

### **Commit After Each Working Feature**

```bash
# Example commit pattern:
git add .
git commit -m "feat: add dark/light mode toggle

- Implement theme switcher component
- Add CSS variables for theme colors
- Update layout to support theme switching
- Test: pnpm run dev works ✅"
```

### **Major Milestone Commits**

```bash
# After completing major features:
git commit -m "feat: complete MVP prompt editor

- Working prompt editor with autosave
- Dark/light mode toggle
- Basic label system
- n8n JSON parser integration
- Ready for authentication phase

Test: pnpm run dev works ✅
Next: Add Clerk authentication"
```

## Development Checklist

### **Before Each Change**

- [ ] `pnpm run dev` works
- [ ] No TypeScript errors
- [ ] No console errors
- [ ] Current feature is complete

### **After Each Change**

- [ ] `pnpm run dev` still works
- [ ] Feature works as expected
- [ ] No breaking changes
- [ ] Commit the change

### **Before Major Commits**

- [ ] All features working
- [ ] No linting errors
- [ ] TypeScript compilation successful
- [ ] Test in browser
- [ ] Write descriptive commit message

## File Organization

### **Incremental File Structure**

```
everprompt-n8n/
├── app/
│   ├── page.tsx              # Home page (start here)
│   ├── layout.tsx            # Root layout
│   └── globals.css           # Global styles
├── components/
│   ├── ui/                   # Basic UI components
│   ├── features/             # Feature components
│   └── layout/               # Layout components
├── lib/
│   ├── utils.ts              # Utility functions
│   ├── n8n-parser.ts         # n8n JSON parser
│   └── db.ts                 # Database utilities
└── .cursor/rules/            # Development rules
```

### **Component Development Order (shadcn/ui)**

1. **Basic UI Components** (shadcn/ui Button, Input, Card, etc.)
2. **Layout Components** (Header, Sidebar, etc.)
3. **Feature Components** (PromptEditor with shadcn/ui Card, ArcLabels, etc.)
4. **Page Components** (Home, Dashboard, etc.)
5. **Integration Components** (Auth, Database, etc.)

## Testing Strategy

### **Development Testing**

```bash
# Always test these before committing:
pnpm run dev          # Must work
pnpm run build        # Must build successfully
pnpm run lint         # Must pass linting
pnpm run type-check   # Must pass TypeScript checks
```

### **Feature Testing**

- **Manual testing** in browser
- **Console error checking**
- **Responsive design testing**
- **Cross-browser compatibility**

## Rollback Strategy

### **Git Rollback Points**

```bash
# Create rollback points:
git tag v0.1.0-working-basic-layout
git tag v0.2.0-working-theme-toggle
git tag v0.3.0-working-prompt-editor

# Rollback if needed:
git checkout v0.2.0-working-theme-toggle
```

### **Feature Flags**

```typescript
// Use feature flags for gradual rollout
const FEATURES = {
  AUTHENTICATION: process.env.NODE_ENV === "development",
  PAYMENT: false, // Enable when ready
  API: false, // Enable when ready
  SHADCN_UI: true, // Always enabled - core UI library
} as const;

// shadcn/ui Component Configuration
const SHADCN_CONFIG = {
  theme: "light" | "dark" | "system",
  components: {
    button: "default",
    card: "default",
    input: "default",
    sheet: "default",
  },
} as const;
```

## Domain Configuration

### **everprompt.ai Setup**

```typescript
// next.config.ts
const nextConfig = {
  async redirects() {
    return [
      {
        source: "/",
        destination: "/dashboard",
        permanent: false,
      },
    ];
  },
  async rewrites() {
    return [
      {
        source: "/api/:path*",
        destination: "/api/:path*",
      },
    ];
  },
};
```

### **Environment Configuration**

```bash
# .env.local
NEXT_PUBLIC_APP_URL=https://everprompt.ai
NEXT_PUBLIC_APP_NAME=EverPrompt
NEXT_PUBLIC_APP_DESCRIPTION="Prompt management for n8n workflows"
```

## Development Phases

### **Phase 1: Basic Foundation (Week 1)**

- [ ] Next.js 15 setup with App Router
- [ ] Tailwind CSS 4 configuration
- [ ] shadcn/ui components setup and configuration
- [ ] Basic layout and components (shadcn/ui)
- [ ] Dark/light mode toggle (shadcn/ui Switch)
- [ ] TypeScript configuration

### **Phase 2: Core Features (Week 2)**

- [ ] Prompt editor component (shadcn/ui Card + Textarea)
- [ ] Label system (arc navigation with shadcn/ui styling)
- [ ] n8n JSON parser
- [ ] Basic data management
- [ ] Responsive design (shadcn/ui responsive utilities)

### **Phase 3: Authentication (Week 3)**

- [ ] Clerk integration
- [ ] User authentication
- [ ] Protected routes
- [ ] User management
- [ ] Workspace setup

### **Phase 4: Database (Week 4)**

- [ ] Neon PostgreSQL setup
- [ ] Prisma ORM configuration
- [ ] Database schema
- [ ] CRUD operations
- [ ] Data validation

## Quality Assurance

### **Code Quality**

- **TypeScript strict mode** enabled
- **ESLint rules** enforced
- **Prettier formatting** applied
- **Consistent naming** conventions

### **Performance**

- **Bundle size** monitoring
- **Loading time** optimization
- **Image optimization** (Next.js)
- **Code splitting** (lazy loading)

### **Security**

- **Input validation** (Zod)
- **SQL injection** prevention (Prisma)
- **XSS protection** (React)
- **CSRF protection** (Next.js)

## Deployment Strategy

### **Development**

- **Local development** with `pnpm run dev`
- **Hot reload** for instant feedback
- **Error overlay** for debugging
- **TypeScript checking** in real-time

### **Staging (Future)**

- **staging.everprompt.ai** for testing
- **Preview deployments** for each PR
- **Automated testing** before merge
- **Performance monitoring**

### **Production**

- **everprompt.ai** for live users
- **Automated deployments** from main branch
- **Error monitoring** (Vercel Analytics)
- **Performance tracking**

## Success Metrics

### **Development Success**

- **Build time** < 30 seconds
- **Hot reload** < 2 seconds
- **TypeScript errors** = 0
- **Linting errors** = 0

### **Feature Success**

- **User can create** a prompt
- **User can organize** with labels
- **User can upload** n8n workflow
- **User can share** prompts

### **Business Success**

- **Break-even** at 9 paid users
- **User retention** > 70%
- **Community engagement** > 30%
- **Revenue growth** month-over-month

This incremental approach ensures you always have a working project while building towards your vision of the perfect n8n prompt management platform! 🚀

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
