## startechindiacrm

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->
# AGENTS.md — Star Tech India CRM

## Final Technical Product & Implementation Specification

> **Status:** Final implementation contract
> **Product:** Star Tech India CRM
> **Application type:** Internal responsive web application
> **Primary users:** Owner, Admin, Sales, Project, Delivery and Accounts teams
> **Deployment:** Vercel + Supabase
> **Mobile application:** Not in current scope
> **UI library:** shadcn/ui only
> **Chart library:** Apache ECharts only

---

# 1. Purpose of This File

This file is the implementation source of truth for any coding agent working on the Star Tech India CRM.

The implementing agent must follow this document before introducing:

* new packages
* new UI patterns
* new database tables
* new status values
* new roles
* new permissions
* new integrations
* new infrastructure
* new business workflows

Do not invent architecture because another approach is personally preferred.

If requirements are ambiguous, preserve the architecture and business rules defined here rather than introducing a conflicting pattern.

---

# 2. Product Objective

Star Tech India CRM manages the complete business lifecycle:

```text
MARKETING
    ↓
LEAD
    ↓
SALES
    ↓
QUALIFICATION
    ↓
MEETING / DEMO
    ↓
REQUIREMENT
    ↓
ESTIMATE / QUOTATION
    ↓
NEGOTIATION
    ↓
WON / LOST
    ↓
CLIENT
    ↓
PROJECT
    ↓
TEAM EXECUTION
    ↓
DELIVERY
    ↓
INVOICE
    ↓
PAYMENT
    ↓
SUPPORT / CLOSURE
```

The CRM must connect these stages instead of creating unrelated modules.

The Owner must be able to understand the complete story of a customer from initial marketing source through payment and project completion.

---

# 3. Non-Negotiable Technology Decisions

## Application

* Next.js App Router
* React
* TypeScript with strict mode
* pnpm package manager

## Hosting

* Vercel

## Backend

* Supabase PostgreSQL
* Supabase Auth
* Supabase Row Level Security
* Supabase Storage
* Supabase Edge Functions
* PostgreSQL functions/RPC
* Supabase Cron for scheduled work
* Supabase Realtime only where justified

## UI

**shadcn/ui only.**

Use official shadcn components and current official shadcn dashboard/sidebar block patterns wherever appropriate.

Do not introduce another general-purpose component library.

## Charts

**Apache ECharts only.**

Do not use:

* Recharts
* Chart.js
* ApexCharts
* Highcharts
* Nivo
* Victory
* Plotly
* Tremor charts
* shadcn Chart/Recharts implementation

Important:

shadcn's normal Chart abstraction may use Recharts. **Do not use it.**

Charts should instead use the `echarts` package directly inside a small reusable internal React component.

Example internal component:

```text
components/charts/e-chart.tsx
```

The surrounding cards, dropdowns, filters and layout must still use shadcn/ui.

## Server State

* TanStack Query

## Tables

* TanStack Table
* shadcn Table

## Forms

* React Hook Form
* Zod
* shadcn Form

## Small Client UI State

* Zustand only when genuinely useful

Do not use Zustand as a replacement for server state.

## Icons

* Lucide React only

## Email

* Brevo Transactional Email API

## Integrations

* Star Tech India website lead forms
* Meta/Facebook Lead Ads
* Instagram Lead Ads where supported through Meta

## Redis

**Do not add Redis for MVP.**

---

# 4. Explicitly Out of Scope

Do not build these in the current version:

* Android application
* iOS application
* desktop native application
* multi-tenant SaaS architecture
* subscription billing
* payroll
* HRMS
* complete ERP
* complete procurement ERP
* complex inventory system
* customer mobile application
* WhatsApp CRM inbox
* AI agents
* AI lead scoring
* Redis infrastructure
* Trigger.dev
* Kafka
* RabbitMQ
* microservices
* Kubernetes
* custom workflow builder

The system is an internal Star Tech India web CRM.

---

# 5. UI Architecture Rules

All non-chart UI must use shadcn components where a suitable component exists.

Preferred components include:

* Sidebar
* Breadcrumb
* Button
* Card
* Badge
* Avatar
* Input
* Textarea
* Checkbox
* Radio Group
* Switch
* Select
* Command
* Popover
* Calendar
* Date Picker patterns
* Form
* Table
* Tabs
* Dialog
* AlertDialog
* Sheet
* Drawer
* DropdownMenu
* ContextMenu
* Tooltip
* HoverCard
* Separator
* Skeleton
* ScrollArea
* Collapsible
* Accordion
* Progress
* Pagination
* Sonner
* Empty state patterns
* Command palette patterns

Do not manually recreate a component already available through shadcn.

---

# 6. Dashboard Shell

Start from the current official shadcn dashboard/sidebar block architecture rather than hand-building a completely different shell.

The application should use:

```text
AppSidebar
PageHeader
Breadcrumbs
MainContent
Cards
Filters
Tables / ECharts
```

Desktop:

```text
┌──────────────────┬───────────────────────────────────┐
│ Sidebar          │ Header / Breadcrumb               │
│                  ├───────────────────────────────────┤
│ Dashboard        │                                   │
│ CRM              │ Main page                         │
│ Sales            │                                   │
│ Projects         │                                   │
│ Accounts         │                                   │
│ Reports          │                                   │
│ Administration   │                                   │
└──────────────────┴───────────────────────────────────┘
```

On smaller devices use the responsive sidebar/sheet behavior from shadcn patterns.

---

# 7. Visual Design Principles

The CRM should look professional and operational rather than decorative.

Use:

* strong spacing
* clear hierarchy
* neutral backgrounds
* restrained borders
* readable tables
* status badges
* compact but not crowded forms
* consistent empty states
* skeleton loading
* accessible contrast
* predictable navigation

Do not create:

* unnecessary gradients
* excessive animations
* 3D effects
* glassmorphism everywhere
* oversized cards
* decorative charts with no business purpose
* excessive shadows
* random custom color systems

---

# 8. Chart Rules

Apache ECharts is the only chart system.

Create reusable primitives such as:

```text
EChart
SalesFunnelChart
LeadSourceBarChart
SalesTrendChart
ProjectHealthDonut
OutstandingTrendChart
TeamPerformanceBarChart
```

These are internal domain components that use Apache ECharts.

Use charts only when they improve comprehension.

Preferred chart types:

* Funnel
* Bar
* Horizontal Bar
* Line
* Area where justified
* Donut
* Pie only for a small meaningful whole
* Stacked Bar where useful

Do not use 3D charts.

Every chart must:

* handle loading
* handle empty data
* resize responsively
* dispose its ECharts instance correctly
* support light/dark theme if application theme supports both
* use accessible labels/tooltips
* avoid querying data independently when dashboard aggregation can provide it

---

# 9. Business Roles

There are three account concepts:

```text
OWNER
ADMIN
STAFF
```

Operational capabilities are separate from the account type.

Operational role bundles:

```text
SALES_MANAGER
SALES_EXECUTIVE
PROJECT_MANAGER
DELIVERY
ACCOUNTS
```

---

# 10. Owner

The Owner has unrestricted access to the Star Tech India CRM.

The Owner can:

* access every record
* access every module
* create Admins
* manage Admins
* assign role bundles to Admins
* remove role bundles from Admins
* create staff users
* assign standard staff roles
* manage integrations
* manage services
* manage teams
* view all financial information
* view reports
* access audit logs
* manage application settings
* archive records
* perform authorized overrides

Owner permissions cannot be reduced by another CRM user.

Owner should be handled as an explicit server-side authorization override.

---

# 11. Admin Model

Admin is a privileged account capable of holding multiple operational role bundles.

Example:

```text
Admin Raj
├── SALES_MANAGER
├── PROJECT_MANAGER
└── ACCOUNTS
```

Another Admin:

```text
Admin Priya
├── SALES_MANAGER
└── SALES_EXECUTIVE
```

The system calculates the Admin's effective permissions as the union of their assigned bundles.

Admin account type does not mean "automatically pretend to be Owner."

Owner-only actions remain Owner-only.

Examples of Owner-only actions:

* create/remove another Admin
* change Admin role bundles
* transfer ownership
* change highly sensitive security settings
* destructive retention/purge controls

Admins can receive appropriate administration capabilities required for daily operation but cannot override the Owner.

---

# 12. Standard Staff

Normal staff users generally receive one operational role.

Examples:

```text
Rahul
└── SALES_EXECUTIVE
```

```text
Anjali
└── PROJECT_MANAGER
```

```text
Arun
└── DELIVERY
```

The database may technically use the same role-assignment model for every user, but the application business rule should restrict standard staff to their intended operational role unless the Owner explicitly changes their account structure.

---

# 13. Sales Manager Responsibilities

Can access:

* Sales Dashboard
* Team Leads
* Unassigned Leads
* Lead Assignment
* Follow-ups
* Meetings
* Opportunities
* Quotations
* Pipeline
* Sales Reports
* Lost Leads
* Sales team performance

Can:

* assign/reassign leads
* monitor SLA
* monitor Sales Executives
* view team pipeline
* review quotations when policy requires
* manage sales follow-ups
* analyze conversions

---

# 14. Sales Executive / BDM Responsibilities

Can access appropriate assigned records:

* My Dashboard
* My Leads
* My Opportunities
* Follow-ups
* Meetings
* Requirements
* Quotations
* Clients related to their work
* Personal activities

Core workflow:

```text
Assigned Lead
    ↓
Contact
    ↓
Follow-up
    ↓
Qualification
    ↓
Meeting / Demo
    ↓
Requirement
    ↓
Quotation
    ↓
Negotiation
    ↓
Won / Lost
```

---

# 15. Project Manager Responsibilities

Can manage:

* assigned projects
* project members
* milestones
* project tasks
* task assignment
* project deadlines
* requirements
* deliverables
* project files
* blockers
* reviews
* delivery status
* project progress
* project timeline

---

# 16. Delivery Role

Delivery is used for:

* Developers
* Designers
* Technical employees
* Other execution staff

Do not create separate authorization roles for every job title.

Use employee metadata such as:

```text
role = DELIVERY
department = DEVELOPMENT
```

or:

```text
role = DELIVERY
department = DESIGN
```

Delivery staff primarily see:

* assigned projects
* assigned tasks
* task details
* requirements
* files
* comments
* due dates
* deliverables

They must not automatically see unrelated sales or financial records.

---

# 17. Accounts Responsibilities

Can manage:

* invoices
* invoice line items
* payments
* outstanding balances
* overdue invoices
* payment references
* receipts
* accounts reports

Accounts should see the minimum client/project information necessary to perform financial work.

---

# 18. Permission Architecture

Do not use only a single `role` column on `profiles`.

Recommended structure:

```text
profiles
roles
permissions
role_permissions
user_roles
```

Suggested `profiles` fields:

```text
id
auth_user_id
account_type
full_name
email
phone
department
is_active
created_at
updated_at
```

`account_type`:

```text
OWNER
ADMIN
STAFF
```

`roles`:

```text
SALES_MANAGER
SALES_EXECUTIVE
PROJECT_MANAGER
DELIVERY
ACCOUNTS
```

Effective permissions must be resolved server-side.

Frontend route hiding is UX only.

**Frontend hiding is never authorization.**

---

# 19. Supabase RLS

RLS must be enabled on all relevant business tables.

Authorization must be enforced through:

1. authenticated Supabase session
2. RLS
3. server-side permission checks for important mutations

Do not depend on client-side code to protect data.

Use reusable SQL helper functions for common authorization checks.

Examples:

```text
is_owner()
is_admin()
has_role(role_code)
has_permission(permission_code)
can_access_lead(lead_id)
can_access_project(project_id)
```

Security-definer functions must:

* be narrowly scoped
* explicitly control `search_path`
* not expose service-role privileges broadly
* return only what the application needs

---

# 20. Core Data Principle

Separate:

```text
CONTACT
LEAD
OPPORTUNITY
PROJECT
```

Do not treat them as the same entity.

Relationship:

```text
Contact
   │
   ├── Lead A
   │      ↓
   │  Opportunity A
   │      ↓
   │   Project A
   │
   └── Lead B
          ↓
      Opportunity B
```

One customer can have multiple enquiries and projects.

---

# 21. Customer Identification

Never use phone number as the primary customer identifier.

Use UUID primary keys.

Phone and email are matching/search fields.

This supports:

* phone changes
* multiple phone numbers later
* multiple enquiries
* repeated clients
* multiple projects
* duplicate review

---

# 22. Duplicate Handling

Normalize before matching.

Phone:

* strip spaces
* normalize country code
* store normalized form

Email:

* trim
* lowercase

When a potential contact match exists, show:

```text
Possible Existing Contact

Name
Phone
Email
Previous Leads / Projects

[Link Existing]
[Create New Contact]
```

Do not automatically merge two contacts solely because their phone numbers match.

External integration idempotency is separate from customer duplicate detection.

---

# 23. Canonical Lead Sources

Initial sources:

```text
WEBSITE
META_FACEBOOK
META_INSTAGRAM
REFERRAL
PHONE
WHATSAPP
WALK_IN
MANUAL
OTHER
```

Provider-specific values should normalize into these canonical values.

Store additional source metadata separately.

Examples:

```text
source
source_detail
campaign_name
campaign_external_id
ad_name
ad_external_id
form_name
form_external_id
external_lead_id
integration_connection_id
```

---

# 24. Minimum Lead Fields

Required:

* customer/contact name
* phone
* source

Recommended:

* email
* company
* service
* location
* requirement summary
* campaign
* budget range
* preferred contact method

A lead should be able to arrive without all optional sales information.

The Sales Executive completes qualification later.

---

# 25. Lead State Machine

Lead lifecycle:

```text
NEW
CONTACT_ATTEMPTED
CONTACTED
QUALIFIED
DISQUALIFIED
CONVERTED
LOST
```

Do not use lead lifecycle status for follow-up timing.

For example:

A lead can remain:

```text
NEW
```

while simultaneously being:

```text
SLA_RISK
```

SLA state and lifecycle status are separate.

---

# 26. Opportunity State Machine

```text
REQUIREMENT
MEETING_SCHEDULED
DEMO
ESTIMATE_PREPARATION
QUOTATION_SENT
NEGOTIATION
WON
LOST
```

---

# 27. Follow-Up State Machine

```text
SCHEDULED
DUE
COMPLETED
RESCHEDULED
MISSED
CANCELLED
```

---

# 28. Project State Machine

```text
ONBOARDING
PLANNING
IN_PROGRESS
BLOCKED
IN_REVIEW
QA_UAT
READY_FOR_DELIVERY
DELIVERED
SUPPORT
CLOSED
```

---

# 29. Task State Machine

```text
TODO
IN_PROGRESS
BLOCKED
REVIEW
DONE
CANCELLED
```

---

# 30. Invoice State Machine

```text
DRAFT
ISSUED
PARTIALLY_PAID
PAID
OVERDUE
CANCELLED
```

Do not mix these state machines together.

---

# 31. Complete Business Flow

```text
WEBSITE
META
MANUAL
OTHER FUTURE SOURCE
        ↓
LEAD INGESTION
        ↓
VALIDATION
        ↓
NORMALIZATION
        ↓
IDEMPOTENCY
        ↓
CONTACT MATCH
        ↓
CANONICAL LEAD
        ↓
ASSIGNMENT
        ↓
NEW LEAD
        ↓
CONTACT ATTEMPT
        ↓
CONTACTED
        ↓
QUALIFICATION
    ↙          ↘
LOST        QUALIFIED
                ↓
           OPPORTUNITY
                ↓
         MEETING / DEMO
                ↓
          REQUIREMENTS
                ↓
              BUDGET
                ↓
      ESTIMATE / QUOTATION
                ↓
           NEGOTIATION
          ↙           ↘
       LOST           WON
                       ↓
                    CLIENT
                       ↓
                    PROJECT
                       ↓
             PROJECT MANAGER
                       ↓
                   TEAM
                       ↓
            TASKS / MILESTONES
                       ↓
          DESIGN / DEVELOPMENT
                       ↓
                 REVIEW / QA
                       ↓
                   DELIVERY
                       ↓
                    INVOICE
                       ↓
                    PAYMENT
                       ↓
             SUPPORT / CLOSURE
```

---

# 32. Website Lead Integration

Website lead forms must not insert privileged CRM data directly from the browser.

Recommended flow:

```text
Star Tech Website
      ↓
POST /api/public/leads
      ↓
Next.js Route Handler
      ↓
Zod Validation
      ↓
Rate / Abuse Protection
      ↓
Canonical Ingestion RPC
      ↓
PostgreSQL
```

Public endpoint must:

* validate schema
* validate request size
* normalize values
* sanitize fields
* create idempotency key where applicable
* capture marketing metadata
* prevent arbitrary database fields
* use server-only privileged credentials where required
* return a generic safe response

---

# 33. Website Marketing Attribution

Capture when available:

```text
utm_source
utm_medium
utm_campaign
utm_content
utm_term
landing_url
referrer_url
```

This data must be available later for marketing conversion analytics.

Do not treat UTM fields as customer master data.

They belong to the enquiry/lead.

---

# 34. Generic External Lead API

Architecture should allow future external lead sources without redesigning CRM entities.

Optional future endpoint pattern:

```text
POST /api/v1/leads/ingest/:connectionId
```

Possible authentication:

```text
X-Timestamp
X-Signature
Idempotency-Key
```

Use HMAC for external integration connections when required.

Never reveal an existing integration secret after creation.

Allow:

```text
Rotate Secret
```

not:

```text
Reveal Secret
```

---

# 35. Meta Lead Ads Integration

Support Meta Lead Ads through an isolated provider adapter.

Architecture:

```text
Meta Lead Ad
     ↓
Meta Webhook
     ↓
Supabase Edge Function
     ↓
Verify Request
     ↓
Store Raw Event
     ↓
Idempotency Check
     ↓
Fetch Lead Details
     ↓
Field Mapping
     ↓
Normalize
     ↓
Canonical Lead Ingestion
     ↓
Assignment
```

Meta-specific logic must not be scattered through CRM pages.

Keep provider code under a dedicated module.

Example:

```text
src/lib/providers/meta/
supabase/functions/meta-webhook/
```

---

# 36. Meta Adapter Responsibilities

Meta integration should support:

* connection status
* page mapping
* form synchronization
* campaign metadata
* form metadata
* field mappings
* test connection
* lead webhook ingestion
* duplicate event protection
* retries
* integration health
* disconnect
* reconnect
* token replacement

Do not hard-code provider API versions/scopes permanently inside this PRD.

At implementation time, follow current official Meta API documentation.

---

# 37. Meta Idempotency

Meta may deliver the same event more than once.

Never assume:

```text
one webhook = one unique lead
```

Create uniqueness around provider/external identity.

Example conceptual uniqueness:

```text
provider + external_lead_id
```

Store processing status.

Possible values:

```text
PENDING
PROCESSING
SUCCEEDED
FAILED
RETRYING
DEAD
```

Repeated webhook delivery must not create duplicate CRM leads.

---

# 38. Canonical Lead Ingestion

All lead sources use one canonical ingestion service.

```text
External Input
     ↓
Verify
     ↓
Validate
     ↓
Map Fields
     ↓
Normalize
     ↓
Check Idempotency
     ↓
Match Contact
     ↓
Create Lead
     ↓
Assignment
     ↓
Activity
     ↓
Notification
     ↓
Audit where applicable
```

Prefer one controlled PostgreSQL RPC for transactional lead creation.

Example:

```text
ingest_lead(...)
```

Do not perform five unrelated browser requests to create one lead.

---

# 39. Lead Assignment

Support:

## Manual

Manager assigns a lead.

## Round Robin

```text
Executive A
Executive B
Executive C
Executive A
...
```

Only active eligible Sales Executives participate.

## Service-Based

Example:

```text
Web Development → Web Sales
Software CRM → Software Sales
Mobile Development → App Sales
```

## Source-Based

Example:

```text
Meta Campaign A → Sales Team A
Website Form B → Sales Team B
```

If no assignment rule matches:

```text
UNASSIGNED
```

Sales Manager must be able to see the unassigned queue.

---

# 40. SLA

SLA configuration must be adjustable.

Do not hard-code one permanent SLA.

Example derived states:

```text
OK
RISK
BREACHED
```

SLA is derived from time and business rules.

It must not overwrite the lifecycle status.

---

# 41. Contacts

Contact should contain:

* UUID
* name
* normalized phone
* display phone
* email
* company relationship
* location
* notes
* created timestamp
* updated timestamp

Future optional extension:

* multiple contact methods
* secondary phone
* secondary email

Do not overbuild those until required.

---

# 42. Opportunities

Opportunity fields should include:

* UUID
* lead ID
* contact ID
* service
* title
* owner
* stage
* estimated value
* probability
* expected close date
* budget
* requirement summary
* lost reason if lost
* won timestamp if won

Opportunity is the commercial deal.

Do not store opportunity-specific value directly on Contact.

---

# 43. Requirements

Requirements can include:

* title
* description
* service
* business goals
* features
* references
* budget range
* expected deadline
* decision maker
* priority
* attachments
* notes

Do not create a giant fixed form for every possible service.

Use stable common fields plus configurable/custom requirement values where justified.

---

# 44. Meetings / Demo

Store:

* lead/opportunity
* meeting type
* start time
* end time
* participants
* location or meeting URL
* agenda
* notes
* outcome
* next action
* created by

Meeting outcome does not automatically equal lead lifecycle status.

---

# 45. Follow-Ups

Follow-up types:

```text
CALL
EMAIL
WHATSAPP
MEETING
DEMO
DOCUMENT
PAYMENT
OTHER
```

Views:

* Overdue
* Today
* Tomorrow
* Upcoming
* Completed

Actions:

* Complete
* Reschedule
* Cancel

---

# 46. Quotations

Quotation must support versions.

```text
Quotation
├── Version 1
├── Version 2
└── Version 3
```

Do not overwrite history.

Quotation status:

```text
DRAFT
INTERNAL_REVIEW
SENT
ACCEPTED
REJECTED
EXPIRED
REVISED
```

Fields:

* quotation number
* opportunity
* customer
* service
* version
* valid until
* subtotal
* discount
* tax
* total
* terms
* notes
* status
* created by
* created at

Line items:

* description
* quantity
* rate
* discount if applicable
* tax if applicable
* total

Accepted commercial terms must remain historically traceable.

---

# 47. Won Opportunity Transaction

When an opportunity becomes `WON`, perform the conversion in a controlled server-side transaction.

```text
Opportunity WON
      ↓
Link/Create Client
      ↓
Create Project
      ↓
Copy Requirement
      ↓
Reference Accepted Quotation
      ↓
Assign Project Manager if selected
      ↓
Create Timeline Activity
```

Prefer a PostgreSQL RPC such as:

```text
mark_opportunity_won(...)
```

Either the complete conversion succeeds or it rolls back.

---

# 48. Lost Leads / Opportunities

Lost records must include:

* lost reason
* optional note
* lost at
* lost by

Configurable reasons may include:

* Budget
* No Response
* Competitor
* Timing
* Requirement Mismatch
* Not Genuine
* Duplicate
* Internal Delay
* Other

This enables useful lost-business analytics.

---

# 49. Clients

A Client is not a completely separate duplicate contact database.

Client status represents an established commercial customer relationship.

Keep references to original:

* contact
* opportunities
* projects
* quotations
* invoices

Customer history should remain connected.

---

# 50. Projects

Project fields:

* project ID
* client
* source opportunity
* source quotation
* service
* title
* description
* project manager
* start date
* target date
* status
* priority
* progress
* current blocker
* created at
* updated at

Project Detail should provide:

* overview
* requirements
* team
* milestones
* tasks
* files
* comments/notes
* timeline
* financial summary where permission allows
* delivery information

---

# 51. Project Members

A project can have multiple members.

Fields:

```text
project_id
user_id
project_role
joined_at
removed_at
```

Do not put multiple user IDs into a JSON array on `projects`.

Use relational tables.

---

# 52. Milestones

Fields:

* project
* title
* description
* due date
* status
* completion timestamp
* sort/order position

Milestones should help determine project health.

---

# 53. Tasks

Fields:

* project
* milestone if applicable
* title
* description
* assignee
* status
* priority
* start date
* due date
* blocked reason
* created by
* completed at

Task history must remain traceable.

---

# 54. Project Progress

Do not require Project Managers to type arbitrary percentages if structured task/milestone data can calculate progress.

Possible calculation should be deterministic and documented.

Manual project progress override, if added later, must be explicitly audited.

---

# 55. Vendors / External Costs

Keep this lightweight.

Vendor:

* name
* contact
* service
* tax details if required
* notes
* active status

External project cost:

* project
* vendor
* description
* estimate
* actual amount
* payment state
* reference document

Do not expand this into a procurement ERP.

---

# 56. Invoices

Invoice fields:

* invoice number
* client
* project
* issue date
* due date
* subtotal
* discount
* tax
* total
* paid total
* outstanding total
* status
* notes

Use invoice line items rather than storing one large description blob.

---

# 57. Payments

Payments are append-oriented records.

Store:

* payment UUID
* invoice
* amount
* payment date
* payment method
* transaction/reference number
* notes
* entered by

Never simply overwrite:

```text
invoice.paid_amount
```

without retaining individual payment history.

Calculated concept:

```text
paid_total = SUM(valid payments)
outstanding = invoice_total - paid_total
```

Controlled reversal/correction must be auditable.

---

# 58. Notifications

There are two notification channels in MVP:

1. In-App
2. Brevo Email

Do not build Android push notifications.

---

# 59. In-App Notifications

Useful notification events:

* New lead assigned
* Lead reassigned
* Follow-up due
* Follow-up overdue
* Meeting approaching
* Opportunity won
* Project assigned
* Task assigned
* Task overdue
* Project deadline risk
* Invoice due
* Invoice overdue
* Payment received

Notification fields:

```text
id
user_id
type
title
body
entity_type
entity_id
read_at
created_at
```

---

# 60. Brevo Email Architecture

Brevo must be called server-side.

Never expose the Brevo API key to the browser.

Architecture:

```text
Business Event
    ↓
Notification Service
    ├── In-App Notification
    └── Email Job
            ↓
       Brevo API
            ↓
         Delivery Log
```

Recommended email job status:

```text
PENDING
PROCESSING
SENT
FAILED
RETRYING
DEAD
```

Do not block important business transactions while waiting for an external email API.

Example:

Saving a lead assignment should succeed even if Brevo temporarily fails.

Email delivery should retry independently.

---

# 61. Email Templates

Maintain reusable email templates for important events.

Examples:

* New Lead Assigned
* Follow-Up Reminder
* Meeting Reminder
* Project Assigned
* Task Assigned
* Task Overdue
* Project Deadline
* Invoice Due
* Invoice Overdue
* Payment Received

Avoid sending email for every small field update.

---

# 62. Database — Recommended Core Tables

Suggested schema:

```text
profiles
roles
permissions
role_permissions
user_roles

teams
team_members

contacts
client_companies

services
lead_sources
assignment_rules

leads
lead_assignment_history
lead_activities
lead_notes

follow_ups
meetings

opportunities
opportunity_requirements

quotations
quotation_versions
quotation_items

projects
project_members
project_milestones
project_tasks
task_comments

vendors
project_external_costs

invoices
invoice_items
payments

notifications
email_jobs
email_delivery_logs

integration_connections
integration_field_mappings
integration_events
integration_jobs

meta_pages
meta_forms
meta_campaigns

attachments

audit_logs
app_settings

daily_sales_metrics
daily_source_metrics
daily_project_metrics
daily_finance_metrics
```

Do not create every table before the related feature is implemented.

Use migrations incrementally.

---

# 63. Database Primary Keys

Use UUID primary keys for core business entities.

Examples:

* contacts
* leads
* opportunities
* projects
* invoices
* payments
* tasks

Human-friendly numbers such as:

```text
STI-LD-000142
STI-PROJ-0021
STI-INV-2026-0048
```

are display/business identifiers, not primary keys.

---

# 64. Timestamps

Mutable domain tables should normally include:

```text
created_at
updated_at
```

Important workflow transitions should store explicit timestamps where useful.

Examples:

```text
won_at
lost_at
completed_at
paid_at
archived_at
```

Use timezone-aware PostgreSQL timestamps.

---

# 65. Soft Delete / Archive

Important business records must not be casually hard-deleted.

Use:

```text
archived_at
archived_by
```

where appropriate.

Records to protect include:

* contacts
* leads
* opportunities
* quotations
* projects
* invoices

Hard deletion should only happen through an Owner-controlled retention process.

---

# 66. Audit Logs

Audit privileged or material changes.

Examples:

* role assignment
* permission change
* lead reassignment
* lead archive
* opportunity Won/Lost
* quotation approval/version changes
* project status override
* invoice changes
* payment added/reversed
* integration connected
* integration disconnected
* integration credential replaced
* settings changes

Suggested audit fields:

```text
id
actor_user_id
action
entity_type
entity_id
before_json
after_json
metadata_json
created_at
```

Ordinary users cannot edit audit logs.

---

# 67. Files

Use private Supabase Storage.

Do not store file bytes in PostgreSQL.

Store metadata:

```text
id
bucket
object_path
entity_type
entity_id
file_name
mime_type
size_bytes
uploaded_by
created_at
archived_at
```

Use signed URLs for private access where required.

Examples:

* requirement attachments
* reference files
* quotation PDFs
* agreements
* invoices
* payment receipts
* design files
* deliverables

---

# 68. Integration Raw Payload Retention

Raw integration payloads are useful for troubleshooting but should not grow forever.

Keep:

```text
Canonical CRM data → long-term
Raw integration events → limited retention
```

Use a configurable retention period.

Suggested starting range:

```text
30–90 days
```

Do not delete raw events while an integration job is unresolved.

---

# 69. Performance Principles

Every implementation must assume data grows over time.

Do not wait until performance becomes bad.

Mandatory:

* server-side pagination
* server-side filters
* server-side sorting
* indexed search
* debounced text search
* limited result payloads
* targeted TanStack Query cache invalidation
* aggregate dashboard APIs
* no giant browser datasets
* no unbounded queries
* no accidental N+1 patterns

---

# 70. TanStack Table Rules

List pages must use manual server-driven mode.

Conceptually:

```ts
manualPagination: true
manualSorting: true
manualFiltering: true
```

Default page size:

```text
25
```

Allowed:

```text
25
50
100
```

Do not load 20,000 leads into JavaScript and paginate locally.

---

# 71. Server-Side Pagination

API/query input should include:

```text
page
pageSize
sort
filters
search
```

Return:

```text
rows
pagination metadata
```

For very large datasets, cursor/keyset pagination can replace offset pagination where it meaningfully improves performance.

Do not prematurely complicate small datasets.

---

# 72. Virtualization

Do not virtualize normal 25-row tables.

Use `@tanstack/react-virtual` only when justified, such as:

* very long timelines
* large kanban columns
* intentionally large result lists
* large command/search results

Server pagination remains the primary table scaling method.

---

# 73. Search Debounce

Text search should generally use:

```text
350 ms
```

Acceptable range:

```text
300–400 ms
```

Do not send a database request for every keystroke.

---

# 74. URL State

List filters should be reflected in URL parameters when practical.

Example:

```text
/leads?status=QUALIFIED&source=META_FACEBOOK&page=2
```

Benefits:

* refresh persistence
* browser navigation
* shareable filtered views
* easier debugging

---

# 75. PostgreSQL Search

Do not blindly use:

```sql
ILIKE '%text%'
```

across many columns.

Phone:

* normalized exact or prefix matching

Email:

* normalized exact or prefix matching

Name:

* indexed matching strategy

Only enable/use `pg_trgm` for fields where fuzzy search provides genuine value.

---

# 76. Database Indexing

Create indexes based on actual list/filter patterns.

Likely indexes:

```text
leads(created_at desc)
leads(status, created_at desc)
leads(assigned_to, status, created_at desc)
leads(source, created_at desc)
leads(normalized_phone)
leads(normalized_email)

follow_ups(assigned_to, status, due_at)

opportunities(owner_id, stage, created_at desc)

projects(project_manager_id, status, target_date)

project_tasks(assignee_id, status, due_date)

invoices(status, due_date)

integration_events(provider, external_event_id)
```

Use partial indexes where useful.

Do not create dozens of speculative indexes without query justification.

---

# 77. SELECT Rules

Do not use `SELECT *` for large list pages.

Example lead list should select only what it displays.

Likely:

```text
id
display_id
contact_name
phone
service
source
status
assigned_to
created_at
next_follow_up
sla_state
```

Lead Detail can issue a separate targeted query.

---

# 78. TanStack Query Defaults

Recommended baseline:

```text
staleTime = 60 seconds
gcTime = 30 minutes
refetchOnWindowFocus = false
refetchOnReconnect = true
```

For paginated tables, retain previous page data while the new page loads.

Use sensible query keys.

Examples:

```text
['leads', filters]
['lead', leadId]

['opportunities', filters]
['opportunity', opportunityId]

['projects', filters]
['project', projectId]

['invoices', filters]
['invoice', invoiceId]

['dashboard', 'sales', filters]
```

Do not use one global query key such as:

```text
['crm']
```

for unrelated resources.

---

# 79. Cache Invalidation

After a mutation, invalidate only affected resources.

Example lead update:

```text
Lead Detail
Affected Lead List
Relevant Dashboard Summary
```

Do not:

```text
invalidate all CRM queries
```

after every mutation.

---

# 80. Supabase Realtime

Realtime is not the default data-fetching strategy.

Use it selectively for:

* notification badge
* newly assigned critical lead if required
* selected collaboration features later

Most pages should use TanStack Query.

Do not subscribe every user to every business table.

---

# 81. Dashboard Query Architecture

Bad:

```text
Dashboard
├── Card → full query
├── Card → full query
├── Card → full query
├── Chart → full query
├── Chart → full query
└── Chart → full query
```

Preferred:

```text
Dashboard
    ↓
1–3 optimized RPC/summary queries
    ↓
Aggregated response
    ↓
KPI Cards + ECharts
```

---

# 82. Aggregate Metrics

For historical reporting, create daily metric rollups where useful.

Examples:

```text
daily_sales_metrics
daily_source_metrics
daily_project_metrics
daily_finance_metrics
```

Do not recalculate years of raw transactional history every time an Owner opens the dashboard.

---

# 83. Sales Dashboard

KPI cards:

* Total Leads
* New Leads
* Contacted
* Qualified
* Opportunities
* Quotations
* Won
* Lost
* Conversion %
* Pipeline Value

ECharts:

### Sales Funnel

```text
Lead
→ Contacted
→ Qualified
→ Opportunity
→ Quotation
→ Won
```

### Lead Source Performance

Bar chart.

### Sales Trend

Line chart.

### Team Performance

Bar chart.

---

# 84. Owner Dashboard

Owner dashboard provides company-wide visibility.

Sections:

## Sales

* Leads
* Qualified
* Won
* Lost
* Conversion
* Pipeline Value

## Marketing

* Website Leads
* Meta Leads
* Conversion by Source
* Conversion by Campaign

## Projects

* Active
* On Track
* At Risk
* Overdue
* Delivered

## Finance

* Invoiced
* Collected
* Outstanding
* Overdue

Use ECharts sparingly.

Do not fill the dashboard with unnecessary charts.

---

# 85. Project Dashboard

Show:

* Active Projects
* Overdue Projects
* Blocked Projects
* Tasks Due
* Tasks Overdue
* Projects Near Deadline
* Recently Delivered

ECharts:

* Project status donut
* Delivery/on-time trend if sufficient data exists

---

# 86. Accounts Dashboard

KPI:

* Total Invoiced
* Received
* Outstanding
* Overdue
* Due This Week
* Partially Paid

Tables:

* overdue invoices
* upcoming due invoices
* recent payments

Charts:

* collection trend
* outstanding trend where useful

Apache ECharts only.

---

# 87. Lead Detail

Header:

* customer
* phone
* email
* service
* source
* current lifecycle
* assigned user
* temperature if used

Actions:

* Log Call
* Add Follow-Up
* Schedule Meeting
* Add Requirement
* Create Opportunity
* Update Status
* Reassign
* Mark Lost

Timeline should include:

* Lead Created
* Assignment
* Calls
* Follow-Ups
* Meetings
* Notes
* Status Changes
* Opportunity
* Quotation
* Project conversion

---

# 88. Customer 360

Contact/customer detail should connect:

* contact information
* leads
* opportunities
* quotations
* projects
* invoices where permitted
* payments where permitted
* notes
* activity timeline

Access must respect role permissions.

Delivery staff should not automatically see confidential financial sections.

---

# 89. Main Navigation

Permission-driven navigation:

```text
Dashboard

CRM
├── Leads
├── Follow-Ups
├── Meetings
└── Opportunities

Sales
├── Quotations
└── Pipeline

Clients

Projects
├── All Projects
├── My Projects
├── Tasks
└── Milestones

Accounts
├── Invoices
├── Payments
└── Outstanding

Reports

Administration
├── Users
├── Teams
├── Roles & Permissions
├── Services
├── Integrations
├── Audit Logs
└── Settings
```

Users see only permitted sections.

---

# 90. Responsive Web

The application is web-only but must be usable on mobile browsers.

Desktop:

* tables
* sidebar
* multi-column dashboards

Mobile:

* responsive cards
* compact filters
* Sheet/Drawer patterns
* essential quick actions

Do not squeeze a 12-column desktop table into a narrow screen.

Use a mobile-specific presentation for complex tables where necessary.

---

# 91. Forms

All important forms use:

```text
React Hook Form
+
Zod
+
shadcn Form
```

Validation exists both:

* client-side for UX
* server-side for security

Never trust client validation.

---

# 92. Mutations

Important multi-step operations should use server-side transactions/RPC.

Examples:

```text
ingest_lead()
assign_lead()
qualify_lead()
mark_opportunity_won()
record_payment()
archive_record()
```

Do not allow partial state.

Example payment transaction:

```text
Validate Invoice
    ↓
Insert Payment
    ↓
Recalculate Totals
    ↓
Update Invoice Status
    ↓
Create Activity
    ↓
Write Audit
```

Either all succeeds or all rolls back.

---

# 93. Background Jobs

Do not use Trigger.dev in this project.

Use:

* Supabase Cron
* PostgreSQL job tables
* Supabase Edge Functions

Jobs include:

* Meta retries
* Meta metadata sync
* Brevo email retries
* reminders
* overdue calculations
* daily analytics rollups
* raw-event retention cleanup

---

# 94. Job Table Pattern

Recommended conceptual structure:

```text
id
job_type
payload
status
attempt_count
next_attempt_at
locked_at
last_error
created_at
completed_at
```

For workers, use safe database locking such as:

```sql
FOR UPDATE SKIP LOCKED
```

when implementing concurrent queue processing.

---

# 95. Vercel Responsibilities

Vercel should handle:

* Next.js application
* server rendering
* route handlers
* authenticated server actions
* lightweight API/BFF logic
* website public lead intake

Do not use interactive Vercel requests for:

* long-running batch processing
* large report generation loops
* repeated integration retries
* long synchronization jobs

Move durable background processing into Supabase.

---

# 96. Supabase Responsibilities

Supabase owns:

* PostgreSQL
* Auth
* RLS
* Storage
* Edge Functions
* RPC
* Cron
* integration event persistence
* durable job state
* audit
* analytics rollups

---

# 97. Service Role

Supabase `service_role`:

* server only
* Edge Function/server route only where required
* never browser
* never exposed through API response
* never stored in client state

Use authenticated session-scoped Supabase clients for ordinary user operations.

---

# 98. Redis Decision

Do not install Redis now.

The current stack already provides:

```text
PostgreSQL → durable state
TanStack Query → frontend cache
Database uniqueness → idempotency
Supabase Cron/jobs → background work
```

Redis can be considered later only when metrics demonstrate need for:

* distributed rate limiting
* expensive shared cache
* high-volume locking
* extreme webhook traffic

Do not pre-emptively complicate the MVP.

---

# 99. Rate Protection

Public endpoints must have abuse protection.

Use:

* input validation
* request size limit
* simple IP/request rate records where appropriate
* idempotency
* honeypot for website forms
* duplicate-window protection

If abuse later becomes meaningful, add a dedicated external rate-limit service only after measured need.

---

# 100. Security Requirements

Mandatory:

* HTTPS
* Supabase Auth
* RLS
* server-side authorization
* no client secrets
* webhook verification
* integration idempotency
* private Storage
* signed file URLs where required
* audit logs
* input validation
* request limits
* sensitive mutation logging
* soft deletion
* no SQL built from unsanitized user input

---

# 101. Integration Credential Handling

Store dynamic provider credentials only in secure server-side storage.

UI can show:

```text
Meta Account
Connected
Last Sync
Token Health
```

Never show:

```text
Actual Access Token
```

Actions:

* Test
* Reconnect
* Replace Credential
* Disconnect

Do not implement "Reveal Secret."

---

# 102. Loading States

Every substantial route must include loading UI.

Use shadcn Skeleton.

Skeleton should match the eventual layout rather than displaying a generic spinner over an empty page.

---

# 103. Empty States

Every list must define a meaningful empty state.

Example:

```text
No new leads

New Website and Meta leads will appear here when received.
```

Provide a relevant action when appropriate.

---

# 104. Error States

Errors must provide:

* useful message
* safe technical identifier where appropriate
* retry action where possible

Do not expose:

* secrets
* SQL
* provider tokens
* stack traces

to ordinary users.

---

# 105. Integration Health

Administration → Integrations should show:

* Provider
* Connection status
* Last successful sync
* Last failure
* Pending jobs
* Recent error
* Test action
* Reconnect action

Example:

```text
Website Leads    Healthy
Meta Lead Ads    Healthy
Brevo Email      Healthy
```

---

# 106. Observability

Record operational metrics such as:

* website lead ingestion count
* website ingestion failures
* Meta webhook count
* Meta duplicate events
* Meta processing failures
* lead ingestion latency
* email sent
* email failures
* background job retries
* slow RPCs
* permission failures
* failed integration authentication

Do not log secret values.

---

# 107. Reports

Initial reports:

## Lead Reports

* Leads by Source
* Leads by Campaign
* Leads by Service
* Leads by Sales Executive
* Lead Conversion

## Sales Reports

* Funnel
* Won/Lost
* Pipeline Value
* Quotation Conversion
* Sales Performance

## Marketing Reports

* Meta Leads
* Website Leads
* Source Conversion
* Campaign Conversion

## Project Reports

* Project Status
* Overdue Projects
* Project Delivery
* Task Completion
* Deadline Performance

## Finance Reports

* Invoiced
* Received
* Outstanding
* Overdue
* Collection Trend

---

# 108. Export

CSV export may be provided for authorized report/list pages.

Exports must:

* obey permissions
* obey current filters
* avoid loading huge datasets in the browser first
* be generated server-side when dataset size is large

---

# 109. Suggested Project Structure

```text
src/
├── app/
│   ├── (auth)/
│   │   └── login/
│   │
│   ├── (crm)/
│   │   ├── dashboard/
│   │   ├── leads/
│   │   ├── follow-ups/
│   │   ├── meetings/
│   │   ├── opportunities/
│   │   ├── clients/
│   │   ├── quotations/
│   │   ├── projects/
│   │   ├── tasks/
│   │   ├── accounts/
│   │   ├── reports/
│   │   └── admin/
│   │
│   └── api/
│       ├── public/
│       │   └── leads/
│       └── integrations/
│           └── meta/
│
├── components/
│   ├── ui/
│   ├── layout/
│   ├── charts/
│   ├── dashboard/
│   ├── leads/
│   ├── sales/
│   ├── projects/
│   ├── accounts/
│   └── integrations/
│
├── lib/
│   ├── supabase/
│   ├── auth/
│   ├── permissions/
│   ├── validation/
│   ├── query/
│   ├── audit/
│   ├── email/
│   └── providers/
│       ├── meta/
│       └── website/
│
├── server/
│   ├── actions/
│   ├── queries/
│   └── services/
│       ├── leads/
│       ├── contacts/
│       ├── opportunities/
│       ├── quotations/
│       ├── projects/
│       ├── finance/
│       ├── notifications/
│       └── integrations/
│
└── types/
```

Supabase:

```text
supabase/
├── migrations/
├── functions/
│   ├── meta-webhook/
│   ├── job-worker/
│   └── scheduled-maintenance/
└── seed.sql
```

---

# 110. Provider Adapter Architecture

Keep integrations isolated.

Conceptual interface:

```ts
interface LeadSourceAdapter {
  testConnection(): Promise<ConnectionResult>
  normalize(input: unknown): Promise<CanonicalLeadInput[]>
}
```

Implementations:

```text
MetaLeadAdapter
WebsiteLeadAdapter
```

Future providers plug into the same canonical ingestion pipeline.

Core Lead UI should not contain provider-specific parsing code.

---

# 111. TypeScript Rules

Use strict TypeScript.

Avoid:

```text
any
```

unless integrating with an unavoidable untyped boundary and the value is immediately validated.

External input must be treated as:

```text
unknown
```

then parsed through Zod or equivalent validated schema.

---

# 112. Coding Style

Prefer:

* small domain services
* explicit names
* predictable modules
* server/client boundaries
* reusable shadcn composition
* database transactions
* simple architecture

Avoid:

* huge React components
* giant `utils.ts`
* provider logic in UI
* database access scattered everywhere
* implicit permission rules
* duplicate query implementations

---

# 113. Server Components vs Client Components

Prefer Server Components where interactive client behavior is unnecessary.

Use Client Components only when required for:

* interactive filters
* TanStack Query
* forms
* table state
* ECharts
* dialogs
* client-side interaction

Do not mark entire route trees `"use client"` unnecessarily.

---

# 114. ECharts React Implementation

Use the Apache `echarts` package directly.

A reusable client component should:

1. hold a DOM ref
2. initialize ECharts after mount
3. set option
4. resize on container/window changes
5. dispose on unmount
6. update when option/data changes

Do not install another chart wrapper unless explicitly approved.

---

# 115. Accessibility

Use accessible shadcn primitives.

Requirements:

* keyboard navigation
* labels for form inputs
* meaningful button text/tooltips
* focus visibility
* sensible color contrast
* semantic status text
* charts accompanied by meaningful labels/summary where needed

Do not rely only on color to communicate state.

---

# 116. Page Definition of Done

A page is complete only when:

* correct route exists
* correct sidebar visibility exists
* role permission is correct
* server-side authorization is correct
* RLS is correct
* shadcn-only UI is used
* Apache ECharts only is used for charts
* tables use server pagination where needed
* filters work
* search is debounced
* loading state exists
* empty state exists
* error state exists
* mobile responsive behavior exists
* actions are server validated
* audit is written where required
* TanStack Query invalidation is correct
* no secret is exposed
* list query avoids unnecessary fields
* relevant indexes exist
* no placeholder business data remains

A page that only renders cards and a sidebar is **not complete**.

---

# 117. Integration Definition of Done

An integration is complete only when:

* authorized user can configure it
* server credentials are protected
* connection can be tested
* incoming request verification works
* field mapping works
* normalization works
* canonical ingestion works
* idempotency exists
* retries exist
* errors are persisted
* integration can be disabled
* integration can reconnect
* health is visible
* secrets are masked
* failure cases are tested
* duplicate delivery does not create duplicate leads

---

# 118. Database Migration Rules

All schema changes go through migrations.

Do not manually change production database state without migration source.

Migration naming should be clear.

Example:

```text
20260823_create_contacts.sql
20260823_create_leads.sql
20260823_create_roles_permissions.sql
```

Migrations should include:

* constraints
* indexes
* RLS
* policies
* functions where necessary

---

# 119. Seed Data

Development seed data may include:

* Owner
* sample roles
* permissions
* sample services
* sample teams
* sample leads
* sample projects

Never put real production credentials in `seed.sql`.

---

# 120. Environment Variables

Provide `.env.example`.

Examples of categories:

```text
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY

SUPABASE_SERVICE_ROLE_KEY

BREVO_API_KEY
BREVO_SENDER_EMAIL
BREVO_SENDER_NAME

META_APP_ID
META_APP_SECRET
META_VERIFY_TOKEN
```

Exact names may be adjusted consistently.

Never commit actual values.

---

# 121. Production Deployment

Recommended:

```text
Git Repository
     ↓
Vercel
     ↓
Next.js Production

Supabase Project
├── PostgreSQL
├── Auth
├── Storage
├── Functions
└── Cron
```

Use separate environments/projects where budget allows:

* Development
* Production

At minimum, production credentials must not be reused carelessly in local development.

---

# 122. Data Backup

Use Supabase-supported database backup capabilities according to the selected plan.

Critical records must not depend on browser storage.

Browser/local state is never the source of truth.

---

# 123. Implementation Order

## Phase 1 — Foundation

1. Next.js project
2. shadcn setup
3. official shadcn dashboard/sidebar shell
4. Supabase clients
5. database migrations
6. Auth
7. profiles
8. roles
9. permissions
10. RLS
11. audit infrastructure

## Phase 2 — CRM Core

12. contacts
13. leads
14. activities
15. assignment
16. follow-ups
17. meetings
18. opportunities

## Phase 3 — Integrations

19. canonical ingestion service
20. website lead endpoint
21. integration event system
22. Meta adapter
23. Meta webhook
24. Meta field mapping
25. Meta retry/health

## Phase 4 — Sales Commercial

26. requirements
27. quotations
28. quotation versions
29. negotiation
30. Won/Lost workflow

## Phase 5 — Projects

31. client conversion
32. projects
33. project members
34. milestones
35. tasks
36. comments
37. files
38. project delivery

## Phase 6 — Accounts

39. invoices
40. invoice lines
41. payments
42. outstanding
43. accounts dashboard

## Phase 7 — Notifications

44. in-app notifications
45. Brevo integration
46. email job queue
47. retry handling
48. email delivery logs

## Phase 8 — Analytics

49. Owner dashboard
50. Sales dashboard
51. Project dashboard
52. Accounts dashboard
53. reports
54. daily rollups
55. ECharts visualization

## Phase 9 — Hardening

56. index review
57. query profiling
58. RLS review
59. integration failure testing
60. permission testing
61. responsive QA
62. security review
63. production deployment review

---

# 124. Anti-Patterns — Never Do These

Do not:

* use Material UI
* use Ant Design
* use Chakra
* use Mantine
* use Bootstrap
* introduce another design system
* use Recharts
* use shadcn's Recharts-based Chart component
* use Chart.js
* use ApexCharts
* fetch full tables to paginate in React
* use `SELECT *` on large list pages
* treat frontend hiding as authorization
* expose Supabase service role
* expose Brevo key
* expose Meta credentials
* use phone number as primary customer ID
* automatically merge customers by phone
* combine lead/project/invoice statuses
* use Realtime everywhere
* introduce Redis without demonstrated need
* create an Android app in current scope
* add Trigger.dev
* run long jobs inside normal Vercel requests
* silently discard integration failures
* overwrite quotation history
* overwrite payment history
* hard-delete important business records from normal UI
* create a separate lead schema for every provider
* add custom UI when shadcn already supplies an appropriate primitive

---

# 125. Final Architecture

```text
                     STAR TECH INDIA WEBSITE
                              │
                              ▼
                       VERCEL / NEXT.JS
                              │
                              ▼
                    PUBLIC LEAD ENDPOINT
                              │
                              │
META LEAD ADS ──────► SUPABASE EDGE FUNCTION
                              │
                              ▼
                  CANONICAL LEAD INGESTION
                              │
                              ▼
                       POSTGRESQL + RLS
                              │
            ┌─────────────────┼────────────────┐
            │                 │                │
            ▼                 ▼                ▼
          SALES            PROJECTS          ACCOUNTS
            │                 │                │
            └─────────────────┼────────────────┘
                              │
                              ▼
                         AUDIT / REPORTS
                              │
                  ┌───────────┴───────────┐
                  │                       │
                  ▼                       ▼
         IN-APP NOTIFICATIONS         BREVO EMAIL
                  │
                  └───────────┬───────────┘
                              ▼
                        NEXT.JS WEB CRM
```

---

# 126. Final Product Principle

The CRM should answer, from one connected system:

```text
Where did this lead come from?

Who received it?

Who contacted them?

What is the requirement?

What happened in the meeting/demo?

What is the budget?

What did we quote?

Was the opportunity won or lost?

If won, what project was created?

Who manages the project?

Who is working on it?

Which tasks are pending?

Is the project delayed?

Was it delivered?

What was invoiced?

How much has been paid?

How much remains outstanding?

What communication and activity happened throughout?
```

The final Star Tech India CRM architecture is therefore:

> **Website + Meta → CRM → Sales → Client → Project → Delivery → Invoice → Payment**

with:

> **Owner-controlled authorization, multi-role Admin accounts, Supabase-backed security, shadcn-only UI, Apache ECharts-only charts, Brevo email notifications, server-driven tables and cost-optimized Vercel + Supabase infrastructure.**

This document is the implementation contract. Any agent building the CRM must preserve these rules unless the Product Owner explicitly changes them.

---
> Source: [dhanushkumar-merndev/startechindiacrm](https://github.com/dhanushkumar-merndev/startechindiacrm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
