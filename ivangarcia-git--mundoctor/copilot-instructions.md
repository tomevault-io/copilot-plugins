## mundoctor

> **MunDoctor** is a comprehensive healthcare platform that connects patients with healthcare professionals across three distinct user roles: **Patients**, **Healthcare Professionals**, and **Administrators**. The application includes:

# MunDoctor Healthcare Platform - AI Assistant Guidelines

## 🏥 Project Overview
**MunDoctor** is a comprehensive healthcare platform that connects patients with healthcare professionals across three distinct user roles: **Patients**, **Healthcare Professionals**, and **Administrators**. The application includes:

- **Frontend**: React 18 + Vite + TypeScript with TailwindCSS and Radix UI
- **Backend**: Node.js + Express API with PostgreSQL database and comprehensive schemas
- **Authentication**: Clerk integration with role-based access control and JWT tokens
- **Real-time Features**: Socket.io WebSocket notifications, live appointment updates
- **Payment Processing**: Stripe integration for professional subscriptions and commission tracking
- **Communication**: Nodemailer for emails, Twilio for SMS, WebSocket for real-time notifications
- **Compliance**: HIPAA-compliant architecture with 6-year audit retention and encryption
- **Security**: Rate limiting, input validation, audit logging, and automated security monitoring

## 🔄 Project Awareness & Context
- **Always understand the healthcare context** - this is a medical platform handling sensitive patient data
- **HIPAA Compliance is mandatory** - all code must maintain privacy and security standards
- **Multi-role system**: Patients, Healthcare Professionals, and Administrators with different permissions
- **Real-time requirements**: Appointments, notifications, and updates must be handled in real-time
- **Security-first approach**: All features must implement proper authentication and authorization

## 🏗️ Application Architecture

### Frontend Structure (`/src`)
- **Pages**: Role-based dashboards (Patient, Professional, Admin)
- **Components**: Reusable UI components with healthcare-specific features
- **Hooks**: Custom React hooks for authentication and data management
- **Services**: API communication and business logic
- **Types**: TypeScript definitions for healthcare entities

### Backend Structure (`/backend/src`)
- **Routes**: RESTful API endpoints organized by feature
- **Services**: Business logic for healthcare operations
- **Models**: Database schemas for medical entities
- **Middleware**: Authentication, authorization, and security
- **Controllers**: Request handling and response formatting

### Database Schema (PostgreSQL)
**Core User Tables:**
- **`users`**: Main user table with Clerk ID integration and role management
- **`patients`**: Patient-specific medical information and preferences
- **`professionals`**: Healthcare professional profiles, licenses, and verification status
- **`user_preferences`**: User settings, notifications, and personalization

**Healthcare Operations:**
- **`appointments`**: Complete appointment lifecycle management with status tracking
- **`appointment_history`**: Change tracking and audit trail for appointments
- **`professional_schedules`**: Weekly availability with exceptions and holidays
- **`specialties`**: Medical specialties catalog for professional classification
- **`professional_services`**: Services offered by professionals with pricing

**Professional Validation System:**
- **`professional_validations`**: Document validation requests and approval workflow
- **`validation_documents`**: Uploaded professional credentials (licenses, certifications)

**Review & Rating System:**
- **`reviews`**: Patient reviews and ratings with content moderation
- **`review_reports`**: Content moderation and abuse reporting
- **`professional_rating_summaries`**: Cached rating statistics for performance

**Business & Admin Tables:**
- **`subscriptions`**: Professional subscription plans and billing
- **`payments`**: Payment processing and commission tracking
- **`support_tickets`**: Customer support system with message threading
- **`admin_actions`**: Audit trail for administrative actions
- **`audit_logs`**: HIPAA-compliant comprehensive logging (6-year retention)
- **`security_events`**: Real-time security monitoring and threat detection

## 🔐 Security & Compliance Requirements

### Authentication (Clerk Integration)
- **All routes must be protected** with Clerk authentication
- **Role-based access control** (RBAC) for different user types
- **JWT tokens** for API authentication
- **Webhook integration** for real-time user synchronization

### Data Protection
- **Encrypt sensitive data** at rest and in transit
- **Audit logging** for all data access and modifications
- **Input validation** using Zod schemas
- **Rate limiting** to prevent abuse

### Healthcare Compliance
- **No logging of PHI** in application logs
- **Secure file upload** for medical documents
- **Data retention policies** for patient records
- **Professional verification** system for healthcare providers

## 🧱 Code Structure & Standards

### File Organization
- **Never create files longer than 500 lines** - refactor into modules
- **Feature-based structure** - group related functionality together
- **Clear naming conventions** - use descriptive names for healthcare context
- **Consistent imports** - prefer relative imports within packages

### Backend Development
- **Use Node.js 20** with ES modules
- **Express.js** for API routes with middleware
- **PostgreSQL** with connection pooling
- **Environment variables** for configuration
- **Comprehensive error handling** with proper HTTP status codes

### Frontend Development
- **React 18** with modern hooks and functional components
- **TypeScript** for type safety
- **Tailwind CSS** for consistent styling
- **Radix UI** components for accessibility
- **React Router** for navigation

## 🧪 Testing & Quality Assurance

### Testing Requirements
- **Unit tests** for all new business logic
- **Integration tests** for API endpoints
- **E2E tests** for critical user flows
- **Security testing** for authentication and authorization

### Code Quality
- **ESLint** for code linting
- **Prettier** for code formatting
- **Type checking** with TypeScript
- **Security scanning** for vulnerabilities

## 📋 Healthcare-Specific Features & User Roles

### 🩺 Healthcare Professionals (`/profesionales/dashboard`)
**Core Features:**
- **Profile Management**: Complete professional profiles with education, experience, specializations
- **Appointment Management**: Schedule management, calendar view, appointment booking acceptance/rejection
- **Patient Management**: Patient history, medical records, treatment plans and notes
- **Service Management**: Define services, pricing, duration, and consultation types
- **Schedule Management**: Set working hours, breaks, holidays, and availability exceptions
- **Analytics Dashboard**: Income tracking, appointment statistics, patient analytics with charts
- **Reviews Management**: View and respond to patient reviews, rating summaries
- **Subscription Management**: Plan upgrades, billing management, feature access control
- **Validation System**: Submit documents for professional verification (license, certifications)
- **Earnings Tracking**: Commission calculations, payment history, financial reporting

### 🏥 Patients (`/paciente/dashboard`)
**Core Features:**
- **Professional Search**: Advanced search with filters (specialty, location, rating, price, availability)
- **Appointment Booking**: Schedule appointments with preferred professionals, real-time availability
- **Medical Records**: Personal health information, medical history, allergies, medications
- **Review System**: Rate and review healthcare professionals, helpful voting on reviews
- **Favorites**: Save preferred professionals for quick access
- **Appointment History**: Track past and upcoming appointments with status updates
- **Profile Management**: Personal information, emergency contacts, insurance details
- **Notifications**: Appointment reminders, confirmations, professional responses

### ⚙️ Administrators (`/admin/dashboard`)
**Core Features:**
- **User Management**: Complete CRUD operations for all users, role assignments
- **Professional Validation**: Review and approve professional credentials, document verification
- **System Analytics**: Platform-wide statistics, user growth, revenue metrics, performance monitoring
- **Support Ticket Management**: Customer support resolution system with priority and assignment
- **Content Moderation**: Review management, report handling, abuse prevention
- **Subscription Management**: Plan management, billing oversight, commission tracking
- **System Configuration**: Platform settings, feature toggles, notification templates
- **Security Monitoring**: Audit logs, security events tracking, threat detection
- **Compliance Management**: HIPAA audit trails, data retention policies, regulatory reporting
- **Financial Oversight**: Payment processing monitoring, dispute handling, refund management

## 💻 Development Environment

### Local Development
- **Docker support** for consistent environments
- **Environment configuration** for different stages
- **Database migrations** for schema management
- **Real-time development** with hot reload

### Third-Party Integrations
- **Clerk**: Authentication and user management
- **Stripe**: Payment processing and subscriptions
- **Twilio**: SMS notifications and communications
- **Nodemailer**: Email notifications and reminders
- **Socket.io**: Real-time features and notifications

## 🚀 Deployment & Operations

### Production Deployment
- **Docker containers** for consistent deployment
- **Environment-specific configurations**
- **Database connection pooling** for performance
- **Monitoring and logging** for system health

### Performance Optimization
- **Redis caching** for frequently accessed data
- **Database query optimization** with proper indexing
- **Image optimization** for profile pictures and documents
- **API rate limiting** for resource protection

## 🧠 AI Assistant Behavior

### Healthcare Context Awareness
- **Always consider patient privacy** when suggesting features
- **Verify compliance requirements** before implementing solutions
- **Understand medical workflows** when designing features
- **Prioritize security** in all code recommendations

### Development Guidelines
- **Ask clarifying questions** about healthcare requirements
- **Verify existing patterns** before creating new implementations
- **Consider real-time requirements** for appointment-related features
- **Ensure proper error handling** for critical healthcare operations

### Code Quality Standards
- **Follow established patterns** in the existing codebase
- **Maintain type safety** with TypeScript
- **Implement proper validation** for all inputs
- **Add comprehensive error handling** for edge cases

## 🛣️ API Routes & Endpoints

### Public Endpoints
- `GET /api/professionals/search` - Professional search with geolocation and filters
- `GET /api/professionals/featured` - Featured professionals listing
- `GET /api/professionals/specialties` - Medical specialties catalog
- `GET /api/professionals/:id` - Professional profile details and reviews

### Authentication Required
- `POST /api/appointments` - Create appointment with availability checking
- `GET /api/appointments` - Get user appointments with filtering
- `PUT /api/appointments/:id` - Update appointment status and details
- `DELETE /api/appointments/:id` - Cancel appointment with reason tracking
- `GET /api/users/profile` - Get current user profile with role-specific data
- `PUT /api/users/profile` - Update user profile and preferences

### Professional Only Endpoints
- `GET /api/professionals/dashboard/stats` - Professional dashboard analytics
- `PUT /api/professionals/profile` - Update professional profile and services
- `POST /api/services` - Create professional service offerings
- `GET /api/schedules` - Get schedule configuration and availability
- `PUT /api/schedules` - Update working hours and availability
- `POST /api/validation/submit` - Submit professional validation documents

### Admin Only Endpoints
- `GET /api/admin/users` - User management with filtering and pagination
- `GET /api/admin/stats` - Platform analytics and performance metrics
- `PUT /api/validation/:id/status` - Approve/reject professional validation
- `GET /api/admin/tickets` - Support ticket management and assignment
- `GET /api/admin/audit-logs` - HIPAA-compliant audit trail access
- `POST /api/admin/security-events` - Security monitoring and threat response

## 🎯 Common Tasks & Patterns

### Authentication & Authorization
- **Clerk Integration**: JWT tokens, role-based middleware, webhook synchronization
- **Route Protection**: Multi-level access control (public, authenticated, role-specific)
- **Session Management**: Secure session handling with automatic timeout
- **User Synchronization**: Real-time sync between Clerk and local database

### Healthcare Data Operations
- **CRUD Operations**: Medical entities with audit logging and validation
- **Transaction Management**: Critical operations with rollback capabilities
- **Search & Filtering**: Professional discovery with geolocation and multi-criteria filtering
- **Appointment Logic**: Availability checking, conflict resolution, status management

### Real-time & Communication Features
- **WebSocket Events**: Live appointment updates, dashboard notifications
- **Email Notifications**: Appointment confirmations, reminders, validation updates
- **SMS Integration**: Twilio for appointment reminders and emergency notifications
- **Push Notifications**: Real-time browser notifications for critical updates

### Professional Validation Workflow
- **Document Upload**: Secure file handling for medical licenses and certifications
- **Multi-step Approval**: Pending → Under Review → Approved/Rejected workflow
- **Admin Review Interface**: Batch processing and detailed document verification
- **Status Notifications**: Real-time updates to professionals throughout the process

### Payment & Subscription System
- **Stripe Integration**: Secure payment processing with webhook handling
- **Subscription Management**: Tiered plans (Free, Basic, Premium, Enterprise)
- **Commission Tracking**: Platform fee calculation and professional payouts
- **Billing Automation**: Recurring payments and invoice generation

## 📚 Key Resources

### Documentation
- **Clerk Documentation**: Authentication patterns and best practices
- **PostgreSQL**: Database optimization and security
- **React Documentation**: Modern React patterns and hooks
- **Express.js**: API development and middleware

### Healthcare Standards
- **HIPAA Compliance**: Privacy and security requirements
- **Medical Data Standards**: Proper handling of health information
- **Accessibility Standards**: Ensuring medical applications are accessible

### Development Tools
- **Docker**: Containerization and deployment
- **Vite**: Frontend build tool and development server
- **Node.js**: Backend runtime and package management
- **PostgreSQL**: Database management and optimization

---

# MunDoctor Development Guidelines

## 🎯 Task Execution Principles
- **Focus on healthcare requirements** - consider patient safety and data security
- **Maintain compliance standards** - ensure HIPAA compliance in all implementations
- **Preserve existing functionality** - avoid breaking changes to critical healthcare features
- **Follow established patterns** - use existing code patterns for consistency

## 📁 File Management Rules
- **Edit existing files** rather than creating new ones unless absolutely necessary
- **Maintain file organization** - keep healthcare-related files properly grouped
- **Update documentation** only when explicitly requested
- **Preserve audit trails** - maintain existing logging and monitoring code

## 🔒 Security & Privacy Rules
- **Never log PHI** (Protected Health Information) in application logs
- **Validate all inputs** using Zod schemas or similar validation
- **Implement proper authorization** for all healthcare data access
- **Use secure communication** for all patient-professional interactions

## 🏥 Healthcare-Specific Considerations
- **Patient privacy first** - always consider privacy implications
- **Professional verification** - maintain strict verification processes
- **Appointment integrity** - ensure appointment data consistency
- **Emergency handling** - consider urgent care scenarios in feature design

## 🚀 Development Best Practices
- **Test thoroughly** - especially for critical healthcare workflows
- **Handle errors gracefully** - provide meaningful error messages
- **Monitor performance** - healthcare applications require high availability
- **Document decisions** - maintain clear reasoning for healthcare-related choices

## 🎯 MunDoctor Mission
**Connecting patients with healthcare professionals through a secure, compliant, and user-friendly platform that prioritizes patient privacy and professional excellence.**

### Core Values
- **Patient Privacy**: All features must protect patient data according to HIPAA standards
- **Professional Excellence**: Support healthcare professionals with tools that enhance patient care
- **Accessibility**: Ensure the platform is accessible to users with diverse needs
- **Reliability**: Maintain high availability for critical healthcare operations
- **Innovation**: Continuously improve healthcare delivery through technology

---

*This context is highly relevant to all MunDoctor development tasks. Consider these guidelines and requirements when working on any aspect of the healthcare platform.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IvanGarcia-Git)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IvanGarcia-Git)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
