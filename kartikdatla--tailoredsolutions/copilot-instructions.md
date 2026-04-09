## tailoredsolutions

> A comprehensive full-stack platform for a self-employed electrician, carpenter, and home improvement professional. The system includes a public-facing website for showcasing services and a companion mobile app for client management. Features include WhisperAI-powered consultation transcription via Teams meetings, client document management, and portfolio showcase.

# CLAUDE.md - Home Improvements Business Platform

## Project Overview

A comprehensive full-stack platform for a self-employed electrician, carpenter, and home improvement professional. The system includes a public-facing website for showcasing services and a companion mobile app for client management. Features include WhisperAI-powered consultation transcription via Teams meetings, client document management, and portfolio showcase.

**Business Name**: [To be configured]
**Domain**: [To be configured]
**Status**: Development

---

## Core Architecture

### Technology Stack

#### Frontend (Website)
- **Framework**: Next.js 15 (App Router)
- **React**: 19.x
- **TypeScript**: 5.x
- **Styling**: Tailwind CSS 4.x
- **Animations**: Framer Motion
- **State Management**: Zustand
- **Form Handling**: React Hook Form + Zod validation

#### Frontend (Mobile App)
- **Framework**: React Native with Expo
- **Navigation**: React Navigation 6.x
- **State Management**: Zustand (shared logic with web)
- **Push Notifications**: Expo Notifications
- **Offline Storage**: WatermelonDB

#### Backend
- **Runtime**: Node.js 20 LTS
- **Framework**: Express.js 4.x
- **API Style**: RESTful with OpenAPI 3.0 documentation
- **Authentication**: JWT with refresh tokens
- **Rate Limiting**: express-rate-limit

#### Database
- **Primary**: MongoDB Atlas (cloud-hosted)
- **ODM**: Mongoose 8.x
- **File Storage**: Cloudflare R2 (images, documents, recordings)
- **Cache**: Redis (session management, rate limiting)

#### AI/ML Services
- **Transcription**: OpenAI Whisper API
- **Document Generation**: OpenAI GPT-4 for consultation summaries

#### Third-Party Integrations
- **WhatsApp Business**: Twilio API
- **Microsoft Teams**: Microsoft Graph API
- **Email**: SendGrid / Resend
- **SMS**: Twilio
- **Calendar**: Google Calendar API / Microsoft Graph

#### DevOps & Hosting
- **Website**: Vercel (Next.js optimized)
- **Backend API**: Railway or Render
- **Mobile App**: Expo EAS Build
- **CI/CD**: GitHub Actions
- **Monitoring**: Sentry (error tracking), Vercel Analytics

---

## Project Structure

```
home-improvements-business/
├── apps/
│   ├── web/                          # Next.js website
│   │   ├── app/
│   │   │   ├── (public)/             # Public pages
│   │   │   │   ├── page.tsx          # Homepage
│   │   │   │   ├── services/         # Services showcase
│   │   │   │   ├── portfolio/        # Project gallery
│   │   │   │   ├── about/            # About the business
│   │   │   │   ├── testimonials/     # Client reviews
│   │   │   │   ├── contact/          # Contact form
│   │   │   │   └── book/             # Booking system
│   │   │   ├── (client)/             # Client portal (authenticated)
│   │   │   │   ├── dashboard/        # Client dashboard
│   │   │   │   ├── consultations/    # View consultation documents
│   │   │   │   ├── projects/         # Track ongoing projects
│   │   │   │   └── documents/        # Access shared documents
│   │   │   ├── (admin)/              # Admin panel
│   │   │   │   ├── dashboard/        # Business dashboard
│   │   │   │   ├── clients/          # Client management
│   │   │   │   ├── consultations/    # Manage consultations
│   │   │   │   ├── portfolio/        # Manage portfolio
│   │   │   │   ├── services/         # Manage services
│   │   │   │   └── settings/         # Business settings
│   │   │   ├── api/                  # API routes (BFF pattern)
│   │   │   └── layout.tsx
│   │   ├── components/
│   │   │   ├── ui/                   # Reusable UI components
│   │   │   ├── forms/                # Form components
│   │   │   ├── layout/               # Layout components
│   │   │   └── features/             # Feature-specific components
│   │   ├── lib/
│   │   │   ├── api.ts                # API client
│   │   │   ├── auth.ts               # Auth utilities
│   │   │   └── utils.ts              # Helper functions
│   │   └── public/
│   │       └── images/
│   │
│   └── mobile/                       # React Native Expo app
│       ├── app/                      # Expo Router pages
│       ├── components/
│       ├── hooks/
│       ├── lib/
│       └── assets/
│
├── packages/
│   ├── api/                          # Express.js backend
│   │   ├── src/
│   │   │   ├── controllers/
│   │   │   ├── middleware/
│   │   │   ├── models/
│   │   │   ├── routes/
│   │   │   ├── services/
│   │   │   │   ├── whisper.service.ts
│   │   │   │   ├── teams.service.ts
│   │   │   │   ├── whatsapp.service.ts
│   │   │   │   └── document.service.ts
│   │   │   └── utils/
│   │   └── tests/
│   │
│   ├── shared/                       # Shared TypeScript types & utilities
│   │   ├── types/
│   │   ├── constants/
│   │   └── validators/
│   │
│   └── db/                           # Database schemas & migrations
│       ├── schemas/
│       └── seeds/
│
├── docs/                             # Documentation
│   ├── api/                          # API documentation
│   ├── architecture/                 # Architecture decisions
│   └── deployment/                   # Deployment guides
│
├── scripts/                          # Utility scripts
├── .github/
│   └── workflows/                    # CI/CD pipelines
├── docker-compose.yml                # Local development
├── turbo.json                        # Turborepo configuration
└── package.json                      # Monorepo root
```

---

## Feature Specifications

### 1. Public Website Features

#### 1.1 Homepage
- Hero section with business tagline and CTA
- Featured services grid
- Recent portfolio highlights
- Client testimonials carousel
- Contact CTA section

#### 1.2 Services Page
```typescript
interface Service {
  id: string
  slug: string
  name: string                    // e.g., "Electrical Installation"
  category: 'electrical' | 'carpentry' | 'home-improvement'
  shortDescription: string
  fullDescription: string
  features: string[]
  priceRange?: {
    min: number
    max: number
    unit: 'fixed' | 'hourly' | 'per-sqft'
  }
  images: string[]
  faqs: Array<{ question: string; answer: string }>
  relatedServices: string[]       // Service IDs
  isActive: boolean
}
```

**Service Categories**:
- **Electrical**: Rewiring, fuse box upgrades, lighting installation, EV charger installation, fault finding, inspection & testing
- **Carpentry**: Custom furniture, kitchen fitting, door installation, flooring, built-in storage, shelving
- **Home Improvement**: Bathroom renovation, kitchen remodeling, painting & decorating, tiling, general repairs

#### 1.3 Portfolio Gallery
```typescript
interface PortfolioProject {
  id: string
  slug: string
  title: string
  category: 'electrical' | 'carpentry' | 'home-improvement'
  subcategory: string
  description: string
  scope: string                   // Brief scope of work
  duration: string                // e.g., "2 weeks"
  completionDate: Date
  location: string                // General area, not exact address
  images: Array<{
    url: string
    caption?: string
    isBefore: boolean
    isAfter: boolean
    isFeatured: boolean
  }>
  testimonial?: {
    quote: string
    clientName: string
    clientInitials?: string       // For privacy
  }
  tags: string[]
  isPublished: boolean
}
```

#### 1.4 Booking System
```typescript
interface BookingRequest {
  id: string
  clientName: string
  clientEmail: string
  clientPhone: string
  preferredContactMethod: 'email' | 'phone' | 'whatsapp'
  serviceCategory: string
  serviceDescription: string
  propertyType: 'house' | 'flat' | 'commercial' | 'other'
  urgency: 'emergency' | 'urgent' | 'standard' | 'flexible'
  preferredDates: Date[]
  preferredTimeSlots: string[]
  images?: string[]               // Client-uploaded images
  notes?: string
  status: 'pending' | 'contacted' | 'scheduled' | 'completed' | 'cancelled'
  createdAt: Date
  updatedAt: Date
}
```

**Booking Flow**:
1. Client selects service category
2. Fills in contact details and project description
3. Uploads relevant images (optional)
4. Selects preferred contact method (WhatsApp, Phone, Email)
5. Receives confirmation via chosen method
6. Business owner receives notification + email summary

### 2. Client Portal Features

#### 2.1 Consultation Documents
```typescript
interface ConsultationDocument {
  id: string
  consultationId: string
  clientId: string
  title: string
  type: 'initial-consultation' | 'quote' | 'project-update' | 'completion'
  content: {
    summary: string               // AI-generated summary
    keyPoints: string[]           // Main discussion points
    recommendations: string[]     // Professional recommendations
    nextSteps: string[]           // Action items
    estimatedCost?: {
      min: number
      max: number
      breakdown?: Array<{
        item: string
        cost: number
      }>
    }
  }
  attachments: Array<{
    type: 'client-image' | 'example-work' | 'diagram' | 'quote-pdf'
    url: string
    caption?: string
    uploadedBy: 'client' | 'business'
  }>
  transcription?: {
    rawText: string
    processedAt: Date
    meetingDuration: number       // minutes
    meetingDate: Date
  }
  status: 'draft' | 'pending-review' | 'published' | 'archived'
  createdAt: Date
  updatedAt: Date
  publishedAt?: Date
}
```

#### 2.2 Teams Meeting Integration & WhisperAI Transcription

**Flow**:
1. Business owner schedules Teams meeting via booking system
2. Meeting is recorded with client consent
3. After meeting ends, recording is sent to WhisperAI for transcription
4. GPT-4 processes transcription to create structured consultation document
5. Business owner reviews and edits document
6. Document is published to client portal
7. Client receives notification to view consultation summary

```typescript
// WhisperAI Service
interface WhisperTranscriptionService {
  transcribeAudio(audioUrl: string): Promise<TranscriptionResult>
  processConsultation(transcription: string): Promise<ConsultationSummary>
}

interface TranscriptionResult {
  text: string
  segments: Array<{
    start: number
    end: number
    text: string
    speaker?: string
  }>
  language: string
  duration: number
}

interface ConsultationSummary {
  title: string
  summary: string
  keyPoints: string[]
  recommendations: string[]
  nextSteps: string[]
  estimatedCost?: {
    min: number
    max: number
    notes: string
  }
  questionsFromClient: string[]
  materialsDiscussed: string[]
}
```

### 3. Admin Panel Features

#### 3.1 Dashboard
- Pending booking requests
- Upcoming consultations
- Recent client activity
- Revenue overview (optional)
- Quick actions

#### 3.2 Consultation Management
- View scheduled meetings
- Process transcriptions
- Edit consultation documents
- Add example work images
- Publish to client portal

#### 3.3 Client Management
- Client list with search/filter
- Client profile with project history
- Communication log
- Document sharing

#### 3.4 Portfolio Management
- Add/edit portfolio projects
- Upload before/after images
- Manage testimonials
- Category organization

### 4. Mobile App Features

#### 4.1 Client App Features
- View portfolio and services
- Submit booking requests
- Push notifications for updates
- Access consultation documents
- Direct contact buttons (WhatsApp, Phone)
- Offline access to saved documents

#### 4.2 Business Owner App Features (Phase 2)
- Receive booking notifications
- Quick response to inquiries
- Photo capture for portfolio
- Basic project tracking
- Client contact management

---

## Database Schemas

### MongoDB Collections

```typescript
// clients collection
interface ClientDocument {
  _id: ObjectId
  email: string
  phone: string
  name: {
    first: string
    last: string
  }
  preferredContactMethod: 'email' | 'phone' | 'whatsapp'
  address?: {
    line1: string
    line2?: string
    city: string
    postcode: string
    country: string
  }
  projects: ObjectId[]            // Reference to projects
  consultations: ObjectId[]       // Reference to consultations
  notes: string
  tags: string[]
  createdAt: Date
  updatedAt: Date
}

// bookings collection
interface BookingDocument {
  _id: ObjectId
  clientId?: ObjectId             // Linked after client created
  contactInfo: {
    name: string
    email: string
    phone: string
  }
  service: {
    category: string
    description: string
  }
  property: {
    type: string
    location?: string
  }
  urgency: string
  preferredDates: Date[]
  preferredTimes: string[]
  images: string[]
  notes: string
  status: string
  statusHistory: Array<{
    status: string
    changedAt: Date
    changedBy?: ObjectId
    notes?: string
  }>
  createdAt: Date
  updatedAt: Date
}

// consultations collection
interface ConsultationDocument {
  _id: ObjectId
  clientId: ObjectId
  bookingId?: ObjectId
  meetingDetails: {
    platform: 'teams' | 'whatsapp' | 'phone' | 'in-person'
    scheduledAt: Date
    duration?: number
    recordingUrl?: string
    teamsEventId?: string
  }
  transcription?: {
    status: 'pending' | 'processing' | 'completed' | 'failed'
    rawText?: string
    processedAt?: Date
    whisperJobId?: string
  }
  document?: {
    title: string
    summary: string
    keyPoints: string[]
    recommendations: string[]
    nextSteps: string[]
    estimatedCost?: object
    attachments: object[]
    status: string
    publishedAt?: Date
  }
  createdAt: Date
  updatedAt: Date
}

// projects collection
interface ProjectDocument {
  _id: ObjectId
  clientId: ObjectId
  consultationId?: ObjectId
  title: string
  category: string
  subcategory: string
  description: string
  status: 'quoted' | 'approved' | 'in-progress' | 'completed' | 'on-hold' | 'cancelled'
  timeline: {
    estimatedStart?: Date
    actualStart?: Date
    estimatedCompletion?: Date
    actualCompletion?: Date
  }
  pricing: {
    quoted: number
    final?: number
    invoices: ObjectId[]
  }
  updates: Array<{
    date: Date
    content: string
    images?: string[]
    isPublic: boolean
  }>
  portfolio?: {
    isPublished: boolean
    publishedAt?: Date
    images: object[]
    testimonial?: object
  }
  createdAt: Date
  updatedAt: Date
}

// services collection
interface ServiceDocument {
  _id: ObjectId
  slug: string
  name: string
  category: string
  shortDescription: string
  fullDescription: string
  features: string[]
  priceRange?: object
  images: string[]
  faqs: object[]
  relatedServices: ObjectId[]
  displayOrder: number
  isActive: boolean
  createdAt: Date
  updatedAt: Date
}

// portfolio collection
interface PortfolioDocument {
  _id: ObjectId
  projectId?: ObjectId
  slug: string
  title: string
  category: string
  subcategory: string
  description: string
  scope: string
  duration: string
  completionDate: Date
  location: string
  images: object[]
  testimonial?: object
  tags: string[]
  isPublished: boolean
  isFeatured: boolean
  displayOrder: number
  createdAt: Date
  updatedAt: Date
}

// users collection (admin/business owner)
interface UserDocument {
  _id: ObjectId
  email: string
  passwordHash: string
  name: string
  role: 'admin' | 'owner'
  permissions: string[]
  lastLoginAt: Date
  createdAt: Date
  updatedAt: Date
}
```

---

## API Routes

### Public Routes
```
GET    /api/services              # List all active services
GET    /api/services/:slug        # Get service details
GET    /api/portfolio             # List published portfolio items
GET    /api/portfolio/:slug       # Get portfolio item details
GET    /api/testimonials          # List approved testimonials
POST   /api/bookings              # Submit booking request
POST   /api/contact               # Submit contact form
```

### Client Routes (Authenticated)
```
GET    /api/client/profile        # Get client profile
PUT    /api/client/profile        # Update client profile
GET    /api/client/consultations  # List client's consultations
GET    /api/client/consultations/:id  # Get consultation details
GET    /api/client/documents      # List client's documents
GET    /api/client/documents/:id  # Get document details
GET    /api/client/projects       # List client's projects
```

### Admin Routes (Authenticated)
```
# Clients
GET    /api/admin/clients         # List all clients
POST   /api/admin/clients         # Create client
GET    /api/admin/clients/:id     # Get client details
PUT    /api/admin/clients/:id     # Update client
DELETE /api/admin/clients/:id     # Delete client

# Bookings
GET    /api/admin/bookings        # List all bookings
GET    /api/admin/bookings/:id    # Get booking details
PUT    /api/admin/bookings/:id    # Update booking status
POST   /api/admin/bookings/:id/convert  # Convert to client/consultation

# Consultations
GET    /api/admin/consultations   # List all consultations
POST   /api/admin/consultations   # Create consultation
GET    /api/admin/consultations/:id  # Get consultation details
PUT    /api/admin/consultations/:id  # Update consultation
POST   /api/admin/consultations/:id/transcribe  # Trigger transcription
POST   /api/admin/consultations/:id/publish     # Publish to client

# Services
GET    /api/admin/services        # List all services
POST   /api/admin/services        # Create service
PUT    /api/admin/services/:id    # Update service
DELETE /api/admin/services/:id    # Delete service

# Portfolio
GET    /api/admin/portfolio       # List all portfolio items
POST   /api/admin/portfolio       # Create portfolio item
PUT    /api/admin/portfolio/:id   # Update portfolio item
DELETE /api/admin/portfolio/:id   # Delete portfolio item

# Upload
POST   /api/admin/upload          # Upload file to R2
DELETE /api/admin/upload/:id      # Delete uploaded file
```

### Webhook Routes
```
POST   /api/webhooks/teams        # Teams meeting events
POST   /api/webhooks/twilio       # WhatsApp/SMS events
POST   /api/webhooks/whisper      # Transcription completion
```

---

## Integration Details

### Microsoft Teams Integration

**Purpose**: Schedule video consultations, record meetings, extract audio for transcription

**Setup Requirements**:
1. Azure AD App Registration
2. Microsoft Graph API permissions:
   - `Calendars.ReadWrite`
   - `OnlineMeetings.ReadWrite`
   - `OnlineMeetingRecording.Read.All`

**Flow**:
```typescript
// teams.service.ts
interface TeamsService {
  // Create Teams meeting for consultation
  createMeeting(consultation: {
    clientEmail: string
    clientName: string
    scheduledAt: Date
    duration: number
    subject: string
  }): Promise<{
    meetingUrl: string
    meetingId: string
  }>

  // Get meeting recording after completion
  getMeetingRecording(meetingId: string): Promise<{
    recordingUrl: string
    duration: number
  }>

  // Subscribe to meeting events
  subscribeToMeetingEvents(meetingId: string): Promise<void>
}
```

### WhisperAI Integration

**Purpose**: Transcribe meeting recordings into text

**Implementation Options**:
1. **OpenAI Whisper API** (Recommended) - Cloud-hosted, simple integration
2. **Self-hosted Whisper** - More control, higher setup complexity

```typescript
// whisper.service.ts
interface WhisperService {
  // Transcribe audio file
  transcribe(audioUrl: string, options?: {
    language?: string
    prompt?: string
  }): Promise<{
    text: string
    segments: TranscriptSegment[]
    duration: number
  }>

  // Process transcription into consultation summary
  generateConsultationSummary(transcription: string): Promise<{
    summary: string
    keyPoints: string[]
    recommendations: string[]
    nextSteps: string[]
    estimatedCost?: {
      min: number
      max: number
      notes: string
    }
  }>
}
```

**GPT-4 Prompt for Consultation Processing**:
```
You are an assistant helping a professional electrician, carpenter, and home improvement specialist create consultation summaries for clients.

Given the following transcription of a consultation meeting, create a structured summary including:

1. **Summary**: A clear, professional 2-3 paragraph summary of the discussion
2. **Key Points**: The main topics discussed (5-8 bullet points)
3. **Recommendations**: Professional recommendations made during the call
4. **Next Steps**: Action items for both the client and the professional
5. **Cost Estimate**: If pricing was discussed, provide the range mentioned

Format the output as JSON. Use professional, clear language appropriate for a client document.

Transcription:
{transcription}
```

### WhatsApp Business Integration (Twilio)

**Purpose**: Alternative contact method, booking confirmations, status updates

```typescript
// whatsapp.service.ts
interface WhatsAppService {
  // Send booking confirmation
  sendBookingConfirmation(phone: string, booking: {
    name: string
    service: string
    date: string
  }): Promise<void>

  // Send consultation document notification
  sendDocumentNotification(phone: string, documentUrl: string): Promise<void>

  // Handle incoming messages
  handleIncomingMessage(message: TwilioMessage): Promise<void>
}
```

---

## Environment Variables

```env
# Database
MONGODB_URI=mongodb+srv://...
REDIS_URL=redis://...

# Authentication
JWT_SECRET=...
JWT_REFRESH_SECRET=...

# Cloudflare R2
R2_ACCOUNT_ID=...
R2_ACCESS_KEY_ID=...
R2_SECRET_ACCESS_KEY=...
R2_BUCKET_NAME=...
R2_PUBLIC_URL=...

# OpenAI (Whisper & GPT-4)
OPENAI_API_KEY=...

# Microsoft Graph (Teams)
AZURE_CLIENT_ID=...
AZURE_CLIENT_SECRET=...
AZURE_TENANT_ID=...

# Twilio (WhatsApp/SMS)
TWILIO_ACCOUNT_SID=...
TWILIO_AUTH_TOKEN=...
TWILIO_WHATSAPP_NUMBER=...

# Email (SendGrid)
SENDGRID_API_KEY=...
EMAIL_FROM=...

# App URLs
NEXT_PUBLIC_APP_URL=https://...
API_URL=https://...

# Admin
ADMIN_EMAIL=...
ADMIN_PASSWORD=...
```

---

## Development Phases

### Phase 1: Foundation (Weeks 1-4)
- [ ] Set up monorepo with Turborepo
- [ ] Initialize Next.js website with basic pages
- [ ] Set up Express.js API with MongoDB
- [ ] Implement authentication (JWT)
- [ ] Create admin panel foundation
- [ ] Deploy to Vercel/Railway

### Phase 2: Core Features (Weeks 5-8)
- [ ] Complete public website (Services, Portfolio, Contact)
- [ ] Implement booking system
- [ ] Set up Cloudflare R2 for image storage
- [ ] Build client portal authentication
- [ ] Create consultation document system
- [ ] Admin dashboard with CRUD operations

### Phase 3: Integrations (Weeks 9-12)
- [ ] Microsoft Teams meeting integration
- [ ] WhisperAI transcription pipeline
- [ ] GPT-4 consultation summary generation
- [ ] WhatsApp Business integration (Twilio)
- [ ] Email notifications (SendGrid)
- [ ] Push notifications setup

### Phase 4: Mobile App (Weeks 13-16)
- [ ] Initialize React Native Expo project
- [ ] Implement core navigation
- [ ] Portfolio and services browsing
- [ ] Booking submission
- [ ] Push notifications
- [ ] Offline document access

### Phase 5: Polish & Launch (Weeks 17-20)
- [ ] SEO optimization
- [ ] Performance optimization
- [ ] Security audit
- [ ] User testing
- [ ] Documentation
- [ ] Production deployment
- [ ] App Store submission

---

## Cost Estimates

### Development Costs (Self-Development)
- **Time Investment**: 400-500 hours total
- **External Services** (Monthly):
  - MongoDB Atlas: $0-57/month (M0 free tier → M10)
  - Vercel: $0-20/month (Hobby → Pro)
  - Railway: $5-20/month
  - Cloudflare R2: ~$5-15/month (storage + egress)
  - OpenAI API: ~$20-50/month (Whisper + GPT-4)
  - Twilio: ~$15-30/month (WhatsApp Business)
  - SendGrid: $0-20/month (free tier → Essentials)
  - Domain: ~$12/year

### Estimated Monthly Running Costs
- **Minimal (Low Usage)**: $20-40/month
- **Standard (Regular Usage)**: $60-100/month
- **Growth (High Usage)**: $150-250/month

### If Hiring Developer (Estimates)
- **Full Project**: £15,000-30,000
- **Website Only**: £8,000-15,000
- **Mobile App Only**: £10,000-20,000

---

## Security Considerations

### Authentication
- JWT with short expiry (15min access, 7d refresh)
- Secure password hashing (bcrypt, 12 rounds)
- Rate limiting on auth endpoints
- HTTPS only

### Data Protection
- Client data encryption at rest
- PII handling compliance (UK GDPR)
- Secure file uploads (type validation, size limits)
- Input sanitization

### API Security
- CORS configuration
- Request validation (Zod schemas)
- SQL/NoSQL injection prevention
- XSS protection

---

## Troubleshooting

### Common Issues

**Teams Recording Not Available**
- Ensure recording was enabled before meeting
- Check Azure AD permissions
- Verify webhook subscription is active

**Whisper Transcription Fails**
- Check audio file format (MP3, WAV, M4A supported)
- Verify file size under 25MB limit
- Check OpenAI API key validity

**WhatsApp Messages Not Sending**
- Verify Twilio credentials
- Check WhatsApp Business number is approved
- Ensure template messages are approved for marketing

**Client Portal Access Issues**
- Clear browser cache
- Verify client email matches record
- Check JWT token expiry

---

## Maintenance Tasks

### Weekly
- Review pending booking requests
- Check for failed transcription jobs
- Monitor API error logs

### Monthly
- Update dependencies
- Review and optimize database indexes
- Backup verification
- Usage analytics review

### Quarterly
- Security dependency audit
- Performance benchmarking
- Feature usage analysis
- Cost optimization review

---

## Contact & Support

**Developer**: [Your Name]
**Email**: [Your Email]
**Repository**: [GitHub URL]

---

*Document Version: 1.0*
*Created: January 2026*
*Project Status: Planning*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kartikdatla)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kartikdatla)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
