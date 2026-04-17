## learnfunsl

> LearnFun SL is a multilingual educational platform for Sri Lankan students (Grades 1-13) built with modern web technologies focusing on performance, accessibility, and scalability.


# LearnFun SL - Tech Stack

## Overview

LearnFun SL is a multilingual educational platform for Sri Lankan students (Grades 1-13) built with modern web technologies focusing on performance, accessibility, and scalability.

## System Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Client Layer  │    │    API Layer     │    │   Data Layer    │
│ Next.js PWA     │◄──►│ Serverless       │◄──►│ Supabase        │
│ React+TypeScript│    │ Functions        │    │ PostgreSQL      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   AI Layer      │    │ External Services│    │ Infrastructure  │
│ Gemini 2.5 Flash│    │ YouTube + Clerk  │    │ Vercel + CDN    │
│ Pinecone Vector │    │ PostHog Analytics│    │  Google Drive   │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## Frontend Stack

### Core Framework

- **Next.js 15+** - React framework with SSR and API routes
- **React 19+** - Component-based UI library
- **TypeScript** - Type-safe development

### Styling & UI

- **Tailwind CSS** - Utility-first styling
- **Framer Motion** - Smooth animations
- **Lucide React** - Icon library
- **next-themes** - Dark mode support

### Internationalization

- **next-i18next** - Multi-language support (English, Sinhala, Tamil)
- **Custom font loading** - Multi-script rendering

### Performance

- **next-pwa** - Progressive Web App
- **Sharp** - Image optimization
- **Bundle analyzer** - Performance monitoring

## Backend Stack

### Runtime

- **Node.js 22.1+** - JavaScript runtime
- **Next.js API Routes** - Serverless endpoints
- **Express.js** - Web framework integration

### Database & Storage

- **Supabase PostgreSQL** - Primary database
- **Pinecone** - Vector database for AI
- **Google Drive** - File storage and CDN
- **Supabase Storage** - User file management

### Authentication & Security

- **Clerk** - User authentication
- **Rate limiting** - 20 queries/day per user
- **CORS & Helmet.js** - Security headers

## AI & Machine Learning

### Core AI

- **Google Gemini 2.5 Flash** - AI responses
- **RAG System** - Enhanced context retrieval
- **Vector embeddings** - Content similarity
- **Prompt engineering** - Optimized interactions

### Features

- Subject-specific help
- Career guidance and motivation
- Multilingual responses
- Content safety filtering

## External Services

### Content & Media

- **YouTube Data API** - Video integration
- **PDF.js** - Client-side PDF rendering

### Analytics & Monitoring

- **PostHog** - User analytics
- **Vercel Analytics** - Performance metrics
- **Supabase Dashboard** - Database monitoring

## Development Tools

### Code Quality

- **ESLint + Prettier** - Code formatting
- **Husky** - Git hooks
- **TypeScript** - Static typing

### Testing

- **Jest** - Unit testing
- **React Testing Library** - Component testing
- **Playwright** - End-to-end testing

## Data Models

```typescript
interface User {
  id: string;
  email: string;
  name: string;
  grade: number;
  preferred_language: "en" | "si" | "ta";
  created_at: Date;
  last_active: Date;
}

interface Content {
  id: string;
  title: string;
  type: "pastpaper" | "textbook" | "other";
  subject: string;
  grade: number;
  year?: number;
  term: number;
  medium: "english" | "sinhala" | "tamil";
  file_url: string;
  metadata: Record<string, any>;
  created_at: Date;
}

interface ChatMessage {
  id: string;
  user_id: string;
  message: string;
  response: string;
  language: string;
  created_at: Date;
}
```

## Environment Variables

```bash
# Database
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Authentication
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=

# AI Services
GEMINI_API_KEY=
PINECONE_API_KEY=
PINECONE_ENVIRONMENT=

# Storage
GOOGLE_DRIVE_API=
GOOGLE_DRIVE_FOLDER_ID

# Analytics
NEXT_PUBLIC_POSTHOG_KEY=
NEXT_PUBLIC_POSTHOG_HOST=
```

## Performance Requirements

### Technical Targets

- **Page Load:** <2 seconds
- **API Response:** <2 seconds
- **AI Response:** <3 seconds
- **File Download:** <5 seconds
- **Uptime:** 95%

### Mobile Optimization

- **Lighthouse Score:** 90+
- **Core Web Vitals:** Green metrics
- **Touch-friendly** interface
- **Low bandwidth** support

## Security Implementation

### Data Protection

- **HTTPS everywhere** - SSL/TLS encryption
- **Input validation** - Server-side checks
- **SQL injection prevention** - Parameterized queries
- **XSS protection** - Content Security Policy

### API Security

- **Rate limiting** - Request throttling
- **CORS configuration** - Origin restrictions
- **Environment secrets** - Secure key management
- **User authentication** - Clerk integration

## Deployment & Infrastructure

### Hosting

- **Vercel Platform** - Application hosting
- **Serverless Functions** - Auto-scaling
- **Edge Network** - Global CDN
- **Automatic HTTPS** - SSL management

### CI/CD

- **GitHub Integration** - Auto deployments
- **Preview deployments** - Branch testing
- **Environment promotion** - Dev → Production
- **Rollback capability** - Quick reversion

## Monitoring & Analytics

### Performance

- **Vercel Analytics** - Real-time metrics
- **Error tracking** - Exception monitoring
- **API latency** - Response analysis

### User Analytics

- **PostHog Events** - Behavior tracking
- **Conversion funnels** - User journeys
- **Retention metrics** - Engagement analysis

## Scalability

### Current Capacity

- **Concurrent Users:** 100+
- **Database:** 1000+ content items
- **Storage:** 10GB+ content
- **API Calls:** 100+ daily

### Scaling Strategy

- **Serverless auto-scaling**
- **Database optimization**
- **CDN utilization**
- **Caching with Redis** (future)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LearnFunSL) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
