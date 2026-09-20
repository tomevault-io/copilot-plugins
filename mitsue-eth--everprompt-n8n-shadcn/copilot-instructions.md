## architecture

> System architecture and extensibility patterns for EverPrompt


# EverPrompt Architecture Guidelines

## Core Architecture Principles

### 1. **Plugin-First Design**

- Every feature should be implementable as a plugin
- Core system handles only: prompts, labels, workspaces, users
- Extensions: tools, categories, attachments, sharing, analytics
- Plugin registry for community contributions

### 2. **API-First Development**

- RESTful API with GraphQL for complex queries
- OpenAPI specification for community integrations
- Webhook system for real-time updates
- Rate limiting and usage tracking

### 3. **Multi-Tenant by Design**

- Workspace isolation at database level
- Resource-based access control (RBAC)
- Tenant-specific customizations
- Cross-workspace sharing capabilities

## System Components

### Core Services

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Auth Service  │    │  Prompt Service │    │  Label Service  │
│   (Clerk)       │    │   (Core)        │    │   (Core)        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │  Workspace      │
                    │  Service        │
                    │  (Core)         │
                    └─────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Plugin         │    │  Sharing        │    │  Analytics      │
│  Registry       │    │  Service        │    │  Service        │
│  (Extensible)   │    │  (Extensible)   │    │  (Extensible)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Database Architecture

- **Primary DB**: Neon PostgreSQL (multi-tenant)
- **Cache**: Redis for session management and frequent queries
- **Search**: PostgreSQL full-text search initially, Elasticsearch later
- **Files**: Vercel Blob for attachments, S3 for production

## Extensibility Patterns

### 1. **Plugin System**

```typescript
interface EverPromptPlugin {
  id: string;
  name: string;
  version: string;
  hooks: {
    onPromptCreate?: (prompt: Prompt) => void;
    onPromptUpdate?: (prompt: Prompt) => void;
    onLabelCreate?: (label: Label) => void;
  };
  components?: {
    promptEditor?: React.ComponentType;
    labelRenderer?: React.ComponentType;
  };
  api?: {
    endpoints: ApiEndpoint[];
  };
}
```

### 2. **Metadata System**

- JSONB fields for extensible data
- Schema validation with Zod
- Type-safe metadata accessors
- Migration system for metadata changes

### 3. **Event System**

```typescript
interface EverPromptEvent {
  type: "prompt.created" | "prompt.updated" | "label.assigned";
  workspaceId: string;
  userId: string;
  payload: Record<string, any>;
  timestamp: Date;
}
```

## n8n Community Focus

### 1. **n8n-Specific Features**

- **Workflow Integration**: Direct n8n workflow import/export
- **Node Templates**: Pre-built prompt templates for common n8n nodes
- **Community Library**: Curated prompts from n8n community
- **Version Control**: Git-like versioning for prompt evolution

### 2. **Integration Points**

- **n8n API**: Direct integration with n8n instances
- **Webhook Triggers**: Auto-sync prompts with n8n workflows
- **Template Marketplace**: Community-driven prompt templates
- **Usage Analytics**: Track prompt effectiveness in n8n workflows

### 3. **Community Features**

- **Prompt Sharing**: Public/private prompt sharing
- **Rating System**: Community-driven prompt quality
- **Comments & Forks**: Collaborative prompt development
- **Collections**: Curated prompt collections by topic

## Performance Considerations

### 1. **Caching Strategy**

- Redis for frequently accessed data
- CDN for static assets
- Database query optimization
- Client-side caching with SWR

### 2. **Scalability**

- Horizontal scaling with read replicas
- Microservices architecture
- Queue system for background jobs
- Rate limiting and resource quotas

### 3. **Security**

- Row-level security (RLS) in PostgreSQL
- API authentication with JWT
- Input validation and sanitization
- Audit logging for compliance

## Development Phases

### Phase 1: Core MVP (Weeks 1-4)

- Basic prompt CRUD
- Label system
- Simple UI with arc navigation
- Authentication with Clerk
- Basic sharing

### Phase 2: n8n Integration (Weeks 5-8)

- n8n workflow import/export
- Community library
- Template system
- Basic analytics

### Phase 3: Extensibility (Weeks 9-12)

- Plugin system
- Advanced sharing
- Search and discovery
- Mobile optimization

### Phase 4: Community (Weeks 13-16)

- Public marketplace
- Collaboration features
- Advanced analytics
- Enterprise features

## Technology Decisions

### Frontend

- **Next.js 15** with App Router for SSR/SSG
- **React 19** with Server Components
- **Tailwind CSS 4** for styling
- **Framer Motion** for animations
- **Zustand** for client state

### Backend

- **Next.js API Routes** for simplicity
- **Prisma** for database ORM
- **Zod** for validation
- **Redis** for caching
- **Vercel** for deployment

### Database

- **Neon PostgreSQL** for primary data
- **Row-level security** for multi-tenancy
- **Full-text search** with PostgreSQL
- **Connection pooling** for performance

## Monitoring & Observability

### 1. **Application Monitoring**

- Vercel Analytics for performance
- Sentry for error tracking
- Custom metrics for business KPIs
- User behavior analytics

### 2. **Database Monitoring**

- Query performance tracking
- Connection pool monitoring
- Slow query alerts
- Storage usage tracking

### 3. **Business Metrics**

- Prompt creation/usage rates
- User engagement metrics
- Community growth tracking
- Revenue metrics (future)

## Security Considerations

### 1. **Data Protection**

- Encryption at rest and in transit
- GDPR compliance
- Data retention policies
- User data export/deletion

### 2. **Access Control**

- Role-based permissions
- Workspace isolation
- API rate limiting
- Audit trail logging

### 3. **Content Security**

- Prompt content validation
- XSS prevention
- CSRF protection
- Content moderation (future)

## Future Extensibility

### 1. **AI Integration**

- Prompt optimization suggestions
- Auto-categorization
- Content generation
- Quality scoring

### 2. **Advanced Features**

- Collaborative editing
- Real-time synchronization
- Advanced search with AI
- Prompt marketplace

### 3. **Enterprise Features**

- SSO integration
- Advanced analytics
- Custom branding
- API access controls

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
