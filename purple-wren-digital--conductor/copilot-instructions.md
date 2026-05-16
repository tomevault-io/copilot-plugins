## conductor

> | Layer | Technology | Purpose |

# Conductor Technical Specification

## Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Frontend | Next.js + TypeScript | SSR + SPA, type safety |
| UI Components | React Query + Tailwind CSS | Data fetching + utility-first styling |
| Backend API | Encore TS | Backend framework with flexible queries |
| Database | PostgreSQL + Encore SQLDatabase | Relational data + raw SQL with repositories |
| Auth & RBAC | Clerk | Authentication + role-based access control |
| Notifications | SendGrid or Resend | Email notifications |
| Hosting | Vercel (Frontend) + AWS (Backend) | Scalable deployment |

## Core Features

### 1. Ticket Management System
- **Create Ticket**: 
  - Fields: title, description, category, urgency (High/Medium/Low)
  - Link to property/transaction (optional)
  - Auto-assign to department based on category
- **Ticket States**: Assigned, Awaiting Response, In Progress, Resolved
- **Ticket List View**:
  - Sortable columns: Date, Title, Status, Assigned To, Urgency, Name, Email
  - Filters: Status, Urgency, Date Range, Assigned To
  - Search functionality
- **Ticket Detail View**:
  - Comments thread
  - Status updates
  - Assignment changes
  - Action items sidebar
  - Due date management

### 2. User Roles & Permissions

```typescript
enum UserRole {
  AGENT = 'AGENT',           // Can create tickets, view own tickets
  STAFF = 'STAFF',           // Can manage assigned tickets, view all tickets
  ADMIN = 'ADMIN'            // Full access, reports, settings
}
```

### 3. Comments System
- Real-time updates using WebSockets or polling
- Markdown support
- @mentions for notifications
- Timestamps and user attribution
- Internal notes (staff-only visibility)

### 4. Notification System
- Email notifications via SendGrid/Resend:
  - Ticket created
  - Ticket assigned
  - Status changed
  - New comment
  - SLA warnings
- In-app notifications
- User notification preferences

### 5. Dashboard Views

#### Agent Dashboard
```typescript
interface AgentDashboard {
  myTickets: Ticket[];
  recentActivity: Activity[];
  quickActions: {
    createTicket: () => void;
    viewAllTickets: () => void;
  };
}
```

#### Staff Dashboard
```typescript
interface StaffDashboard {
  assignedTickets: Ticket[];
  ticketQueue: Ticket[];
  metrics: {
    avgResponseTime: number;
    openTickets: number;
    overdueTickets: number;
  };
}
```

### 6. Database Schema

The database schema is defined via SQL migrations in `backend/ticket/migrations/`. Data access uses the repository pattern with Encore's SQLDatabase.

**Key Tables:**
- `users` - User accounts linked to Clerk via `clerk_id`
- `tickets` - Support tickets with status, urgency, assignee
- `comments` - Ticket comments with internal flag
- `ticket_categories` - Categories per market center
- `ticket_ratings` - Post-resolution surveys
- `subscriptions` - Stripe subscription data

**Repository Pattern:**
```typescript
// Example: ticket/db.ts
import { db } from "../shared/db";

export const ticketRepository = {
  async findById(id: string) {
    return db.queryRow<Ticket>`SELECT * FROM tickets WHERE id = ${id}`;
  },
  async create(data: CreateTicketData) {
    return db.queryRow<Ticket>`
      INSERT INTO tickets (title, description, creator_id, ...)
      VALUES (${data.title}, ${data.description}, ${data.creatorId}, ...)
      RETURNING *
    `;
  }
};
```

**Enums (defined in TypeScript):**
```typescript
type TicketStatus = "ASSIGNED" | "AWAITING_RESPONSE" | "IN_PROGRESS" | "RESOLVED" | "DRAFT" | "CREATED" | "UNASSIGNED";
type Urgency = "HIGH" | "MEDIUM" | "LOW";
type UserRole = "AGENT" | "STAFF" | "STAFF_LEADER" | "ADMIN";
```

### 7. API Endpoints (Encore)

```typescript
// Ticket endpoints
POST   /api/tickets              // Create ticket
GET    /api/tickets              // List tickets (filtered by role)
GET    /api/tickets/:id          // Get ticket details
PUT    /api/tickets/:id          // Update ticket
POST   /api/tickets/:id/assign   // Assign ticket
POST   /api/tickets/:id/comments // Add comment

// User endpoints
GET    /api/users/me             // Get current user
PUT    /api/users/me/preferences // Update preferences

// Dashboard endpoints
GET    /api/dashboard            // Get role-specific dashboard data
GET    /api/metrics              // Get performance metrics (admin only)
```

### 8. Frontend Routes (Next.js)

```
/                        // Dashboard (role-based)
/tickets                 // Ticket list
/tickets/new             // Create ticket
/tickets/:id             // Ticket detail
/settings                // User settings
/admin                   // Admin panel (admin only)
/admin/reports           // Reports (admin only)
```

### 9. UI Components Structure

```
components/
├── layout/
│   ├── Header.tsx
│   ├── Sidebar.tsx
│   └── Layout.tsx
├── tickets/
│   ├── TicketList.tsx
│   ├── TicketCard.tsx
│   ├── TicketDetail.tsx
│   ├── TicketForm.tsx
│   └── TicketFilters.tsx
├── comments/
│   ├── CommentList.tsx
│   ├── CommentForm.tsx
│   └── Comment.tsx
├── common/
│   ├── Button.tsx
│   ├── Input.tsx
│   ├── Select.tsx
│   ├── Badge.tsx
│   └── Modal.tsx
└── dashboard/
    ├── AgentDashboard.tsx
    ├── StaffDashboard.tsx
    └── MetricCard.tsx
```

### 10. State Management

```typescript
// Using React Query for server state
const useTickets = (filters?: TicketFilters) => {
  return useQuery(['tickets', filters], () => fetchTickets(filters));
};

const useCreateTicket = () => {
  return useMutation(createTicket, {
    onSuccess: () => {
      queryClient.invalidateQueries(['tickets']);
    },
  });
};
```

### 11. Authentication Flow

1. User visits site → Redirect to Clerk
2. User logs in → Clerk returns session token
3. Frontend uses Clerk session token
4. Include Clerk user ID in API requests via Authorization header
5. Backend validates Clerk user ID via Clerk API
6. Check user role for authorization

### 12. Real-time Updates

**Encore Streaming API Implementation:**

```typescript
// Notification Streaming (backend/notifications/stream.ts)
export const notificationStream = api.streamOut<Notification>({
  expose: true,
  auth: true,
  path: "/notifications/stream",
  method: "GET",
})

// Comment Event Streaming (backend/comment/stream.ts)
export const commentStream = api.streamOut<CommentStreamHandshake, CommentStreamMessage>({
  expose: true,
  auth: true,
  path: "/comments/stream/:ticketId",
  method: "GET",
})

// Frontend usage with Encore client
const stream = await client.notifications.notificationStream();
for await (const notification of stream) {
  // Handle real-time notifications
}

// Fallback polling for reliability
useEffect(() => {
  const interval = setInterval(() => {
    refetchTicket();
  }, 30000); // Poll every 30 seconds as fallback

  return () => clearInterval(interval);
}, [ticketId]);
```

### 13. Email Templates

- **Ticket Created**: "Your ticket #[ID] has been received"
- **Ticket Assigned**: "Ticket #[ID] has been assigned to [STAFF]"
- **Status Update**: "Ticket #[ID] status changed to [STATUS]"
- **New Comment**: "[USER] commented on ticket #[ID]"

### 14. Environment Variables

```env
# .env.local
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
DATABASE_URL=postgresql://...
SENDGRID_API_KEY=
```

### 15. Development Priorities

1. **Sprint 1**: Auth setup, database schema, basic ticket CRUD
2. **Sprint 2**: Ticket list/detail views, assignment flow
3. **Sprint 3**: Comments, notifications, real-time updates
4. **Sprint 4**: Dashboards, metrics, reporting
5. **Sprint 5**: Polish, testing, performance optimization
6. **Sprint 6**: User settings, preferences, final touches

## Key Technical Decisions

- **Monorepo**: Single repository for frontend and backend
- **Type Safety**: TypeScript everywhere, raw SQL with typed repositories
- **API Design**: RESTful with Encore TS
- **Database**: Encore SQLDatabase with repository pattern
- **Styling**: Tailwind CSS with custom design tokens
- **Data Fetching**: React Query for caching and synchronization
- **Forms**: React Hook Form with Zod validation
- **Testing**: Vitest for backend unit tests

# Conductor Project Guide

## Project Overview
Conductor is a [brief description of what your app does]. Built with Next.js frontend and Encore backend.

## Architecture
- **Frontend**: Next.js 14+ with Clerk authentication
- **Backend**: Encore framework with TypeScript
- **Auth**: Clerk for user management and authentication
- **Database**: PostgreSQL with Encore SQLDatabase (repository pattern)

## Development Setup
```bash
# Frontend
cd frontend && npm run dev

# Backend  
cd backend && encore run

# Tests
cd backend && npm test
```

## Common Commands
- `npm run lint` - Lint frontend code
- `npm run typecheck` - TypeScript checking
- [Add other common commands you use]

## Authentication Flow
- Frontend uses Clerk React SDK
- Backend validates Clerk user ID via Clerk API (@clerk/backend)
- Clerk user ID passed via `Authorization: Bearer <clerk_user_id>` header
- Users are automatically created in database on first login

## Project Structure
```
conductor/
├── frontend/          # Next.js app
├── backend/           # Encore services
│   ├── admin/         # Admin dashboard APIs
│   ├── auth/          # Authentication handling
│   └── subscription/  # Stripe integration (commented out)
```

## Environment Variables
[List key environment variables and their purpose]

## Known Issues
- [Any current issues or quirks]

## Notes for Claude
- Always run lint/typecheck after code changes
- Use existing error handling patterns (APIError class)
- Follow Clerk authentication patterns (@clerk/backend for user validation)
- Test endpoints with `/health` for connectivity
- User database records use `clerkId` field to link to Clerk users
- Real-time features use Encore's Streaming API (not WebSocket)
  - Notifications stream: `/notifications/stream`
  - Comment events stream: `/comments/stream/:ticketId`
  - Frontend uses generated Encore client for streaming
  - Polling remains as fallback mechanism (30 second intervals)

---
> Source: [Purple-Wren-Digital/conductor](https://github.com/Purple-Wren-Digital/conductor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
