## senopati

> > SENOPATI - System Enterprise Nasional Operasional Pengelolaan Aset Terintegrasi

# AGENTS.md

> SENOPATI - System Enterprise Nasional Operasional Pengelolaan Aset Terintegrasi
>
> Version: 1.0
>
> Status: Production Engineering Constitution
>
> This document defines the mandatory engineering standards, architectural principles, AI behavior, development workflow, and quality requirements for the SENOPATI project.
>
> Every AI Agent, developer, contributor, or automation tool MUST follow this document.
>
> These instructions take precedence over convenience, development speed, or personal coding preferences.

---

# PRIMARY DIRECTIVE

The primary objective of this project is NOT to produce code.

The primary objective is to build an enterprise-grade internal government platform that remains maintainable, scalable, secure, performant, and beautiful for many years.

Every implementation decision MUST prioritize long-term software quality over short-term implementation speed.

If a requested implementation violates this directive, the AI MUST explain the conflict and propose an alternative solution instead of blindly implementing the request.

---

# PROJECT IDENTITY

Project Name

SENOPATI

Full Name

System Enterprise Nasional Operasional Pengelolaan Aset Terintegrasi

Project Type

Government Internal Enterprise Platform

Organization

Sekretariat Negara Republik Indonesia

Deployment Scope

Internal Network

Users

Internal Employees Only

Public Access

Not Allowed

Production Goal

A production-ready enterprise platform for managing government assets with strong visualization, location awareness, historical tracking, and long-term maintainability.

---

# PROJECT CONTEXT

SENOPATI is developed specifically for asset management within the Presidential Palace Secretariat environment in Yogyakarta.

This system is NOT intended to replace SIMAK BMN.

Instead, it complements existing national systems by solving operational problems encountered during daily asset management.

Primary problems currently identified include:

- Difficult asset tracking
- Unknown asset locations
- Manual stock opname
- Spreadsheet-based monitoring
- Slow asset searching
- Poor visualization
- Limited location history
- Inefficient internal workflow

Every engineering decision MUST contribute toward solving one or more of these problems.

---

# PRODUCT VISION

SENOPATI should become the most modern internal government asset platform in Indonesia.

The platform must provide a visual, intuitive, reliable, and elegant user experience while maintaining enterprise-grade software quality.

The system should be capable of supporting future expansion without major architectural redesign.

---

# PRODUCT MISSION

The mission of SENOPATI is to transform government asset management from static data administration into an interactive digital ecosystem.

Every asset should possess:

- Identity
- Location
- Ownership
- History
- Documentation
- Maintenance Record
- Visual Representation
- Digital Presence

No government asset should exist without traceability.

---

# PRODUCT DNA

Every feature developed for SENOPATI must reinforce the following characteristics.

Fast

Elegant

Minimal

Reliable

Scalable

Maintainable

Readable

Visual

Professional

Government Grade

Timeless

Human Friendly

AI Friendly

Enterprise Ready

If a feature weakens one or more of these characteristics, reconsider the implementation.

---

# ENGINEERING PHILOSOPHY

The project follows several non-negotiable engineering principles.

## Principle 1

Architecture First.

Code is temporary.

Architecture survives.

Never prioritize implementation speed over architecture quality.

---

## Principle 2

Performance Is A Feature.

Slow software is considered defective software.

Performance optimization is part of feature completion.

It is never optional.

---

## Principle 3

Every Feature Must Solve A Real Problem.

Never add features simply because they appear modern or impressive.

Every feature must answer:

- Why does this exist?
- Who benefits?
- What problem does it solve?
- Can it scale?

---

## Principle 4

Maintainability Beats Cleverness.

Prefer boring, readable, maintainable solutions over clever implementations.

Future developers must understand the code without requiring the original author.

---

## Principle 5

Consistency Over Creativity.

Consistency improves maintainability.

Do not create multiple solutions for the same problem.

Standardize everything.

---

## Principle 6

Visualization Before Complexity.

Whenever information can be understood faster through visualization, visualization should become the primary interface.

Examples include:

- Interactive maps
- Timelines
- Statistics
- Cards
- Heatmaps
- Diagrams

Large data tables should become secondary interfaces.

---

## Principle 7

Everything Has History.

Historical data is a core business requirement.

Important records must never disappear.

Historical information provides accountability.

---

# AI OPERATING PRINCIPLES

Every AI agent participating in this project MUST behave as a senior engineering team rather than a code generator.

The AI should continuously evaluate architecture quality before implementation.

The AI must behave as:

- Principal Software Architect
- Senior Backend Engineer
- Senior Frontend Engineer
- Senior UI Engineer
- Database Architect
- Security Engineer
- Performance Engineer
- Government System Analyst

The AI must never behave like a junior developer.

---

# AI DECISION PRIORITY

Whenever multiple technical solutions are possible, choose according to the following priority.

1. Architecture Quality
2. Long-Term Maintainability
3. Scalability
4. Performance
5. Security
6. Readability
7. User Experience
8. Development Speed

Development speed is intentionally placed last.

---

# AI DEVELOPMENT MINDSET

Before writing any code, always think through the following sequence.

Understand the business problem.

↓

Understand the user.

↓

Understand the existing architecture.

↓

Search for reusable implementations.

↓

Evaluate scalability.

↓

Evaluate performance impact.

↓

Evaluate security impact.

↓

Design the solution.

↓

Implement.

↓

Validate.

↓

Refactor if necessary.

↓

Document.

Coding should never be the first step.

---

# PROJECT SCOPE

Current modules include:

- Vehicle Management
- Inventory Management
- Linen Management
- Supply Management
- Art Collection Management

Future modules may include:

- Procurement
- Building Management
- Land Management
- Museum Collection
- Archive Management
- IoT Monitoring
- Smart Tracking
- AI Analytics

Current engineering decisions must not prevent these future expansions.

---

# ARCHITECTURE PRINCIPLES

SENOPATI follows a Modular Monolith architecture.

Every module must be independent.

Every module owns its own business logic.

Modules communicate only through defined services.

Cross-module dependencies should remain minimal.

Business logic must never leak into presentation layers.

Controllers must remain thin.

Services contain business logic.

Repositories handle persistence.

The architecture must remain easy to split into microservices in the future if required, without forcing microservices today.

# SOFTWARE ARCHITECTURE

## Architecture Overview

SENOPATI follows an Enterprise Modular Monolith architecture.

This architecture has been intentionally selected because it provides:

- High maintainability
- Low operational complexity
- Excellent developer productivity
- Simple deployment
- Clear separation of business domains
- Future migration path toward microservices

Microservices are NOT required until the business actually demands them.

Avoid premature optimization.

---

## High Level Architecture

Client

↓

Next.js Application

↓

API Layer

↓

NestJS

↓

Application Layer

↓

Domain Services

↓

Repositories

↓

Prisma ORM

↓

MySQL

Every layer has a single responsibility.

Never skip architectural layers.

---

# DOMAIN DRIVEN MODULES

Every business domain owns itself.

Each module contains everything required to function independently.

Example

Vehicle

Inventory

Linen

Supplies

Art

Users

Roles

Permissions

Dashboard

Audit

Map

Notification

Every module must contain

- Controller
- Service
- Repository
- DTO
- Entity Types
- Validators
- Constants
- Tests

Do not place business logic outside its own module.

---

# FRONTEND ARCHITECTURE

Frontend uses Feature Based Architecture.

Example

src/

features/

vehicle/

inventory/

art/

linen/

supplies/

dashboard/

map/

shared/

Each feature owns

components/

hooks/

services/

schemas/

types/

constants/

utils/

Feature isolation is mandatory.

---

# COMPONENT ARCHITECTURE

Every component should have one responsibility.

Preferred hierarchy

Page

↓

Section

↓

Container

↓

Card

↓

Component

↓

Primitive

Never create a component that tries to solve multiple unrelated problems.

---

## Component Size

Preferred

Below 200 lines

Warning

200-350 lines

Critical

Above 350 lines

Split immediately.

---

# SHARED COMPONENTS

Shared components should only exist if they are reused by at least two independent modules.

Do not create shared components prematurely.

Examples

Button

Input

Table

Modal

Dialog

Badge

Avatar

Pagination

Tooltip

Skeleton

Good shared components are generic.

Business-specific components must remain inside their feature.

---

# BUSINESS COMPONENTS

The following components should NEVER exist inside shared folders.

VehicleCard

ArtTimeline

InventoryStatus

LinenHistory

SupplyMovement

These belong inside their own feature module.

---

# STATE MANAGEMENT

Use the smallest possible state scope.

Priority

1 Local State

2 Context

3 Zustand

4 Server State

Global state should only be used when truly necessary.

---

## Zustand Rules

Use Zustand for

Theme

Sidebar

Authentication

Preferences

Temporary UI State

Do NOT store

Server data

API responses

Database records

Lists

Search results

These belong to TanStack Query.

---

## TanStack Query

All server state must be managed using TanStack Query.

Always use

Query Cache

Mutation

Optimistic Update

Invalidate Query

Never manually synchronize API data.

---

# ROUTING

Use App Router.

Every feature owns its own routes.

Avoid deeply nested routing structures.

Routes should remain predictable.

---

# API DESIGN

Every API endpoint must be resource oriented.

Correct

/api/v1/vehicles

/api/v1/inventory

/api/v1/art

/api/v1/assets

Wrong

/getVehicle

/loadInventory

/updateSomething

/deleteItemNow

---

## HTTP Methods

GET

Read

POST

Create

PUT

Replace

PATCH

Update

DELETE

Delete

Use HTTP semantics correctly.

---

# API RESPONSE FORMAT

Every endpoint returns the same structure.

Example

{
  "success": true,
  "message": "",
  "data": {},
  "meta": {},
  "errors": null,
  "timestamp": ""
}

Consistency is mandatory.

---

# DATABASE DESIGN

Database is the single source of truth.

Never duplicate business data.

Normalize when appropriate.

Denormalize only for measured performance improvements.

---

## Primary Key

Always use UUID.

Never expose incremental IDs publicly.

---

## Audit Fields

Every business table must include

created_at

updated_at

deleted_at

created_by

updated_by

deleted_by

These fields are mandatory.

---

## Soft Delete

Business data must never be permanently deleted.

Deleting means

mark as deleted.

Only system administrators may permanently purge records.

---

## History First

Every important business action must generate historical records.

Examples

Asset moved

Asset repaired

Asset assigned

Asset borrowed

Asset returned

Vehicle used

Vehicle serviced

Art relocated

Inventory adjusted

Nothing important should disappear without history.

---

# PRISMA RULES

Never access Prisma from Controllers.

Architecture

Controller

↓

Service

↓

Repository

↓

Prisma

Controllers are forbidden from querying databases directly.

---

Use

select

instead of

include

whenever possible.

Only request required fields.

Never query details tables dynamically unless explicitly requested by the specific feature service.

---

Avoid

N+1 Query

Duplicate Query

Repeated Count Query

---

Use Transactions for

Stock Opname

Asset Transfer

Bulk Import

Bulk Update

Bulk Delete

Role Assignment

---

# VALIDATION

Every incoming request must be validated.

Client validation

does NOT replace

Server validation.

Never trust client input.

---

# SECURITY

Authentication

JWT

Authorization

Scoped RBAC (Users are bound to specific department_id and building_id scopes)

Password

bcrypt

Helmet enabled

CORS configured

Rate limiting enabled

Input validation mandatory

Output sanitization required

Verification rules: NestJS Guards must intercept requests and verify that the user's scope matches the owner scope of the targeted asset.

---

Never

Trust request body

Trust query parameter

Trust uploaded file

Trust client permission

Everything must be verified.

---

# FILE STORAGE

Uploaded files must never be mixed with source code. All file operations MUST go through a unified StorageService abstraction layer.

Preferred local structure (for local storage driver)

uploads/

vehicles/

art/

documents/

inventory/

linen/

supplies/

temporary/

Use unique generated filenames.

Never trust original filenames. Storage driver must be configurable (Local, MinIO, or S3 compatible).

---

# ERROR HANDLING

Never expose

Stack trace

Prisma error

SQL error

Internal path

Unhandled exception

Users receive friendly messages.

Developers receive detailed logs.

---

# LOGGING

Every critical business action must be logged.

Examples

Login

Logout

Import

Export

Asset Transfer

Stock Opname

Location Change

Permission Change

Role Change

Delete

Restore

Approval

Maintenance

Logs are immutable.

---

# CONFIGURATION

Never hardcode

URLs

Colors

Permissions

Roles

Environment variables

API Keys

Secrets

Magic numbers

Business constants

Everything configurable belongs inside configuration files.

---

# DEPENDENCY MANAGEMENT

Before installing a new dependency, answer

Does the project already solve this problem?

Can native APIs solve it?

Is the package actively maintained?

Does it increase bundle size?

Does it affect performance?

Do not install packages simply because they are popular.

---

# PERFORMANCE PHILOSOPHY

Performance is measured.

Never assumed.

Every feature must answer

How much JavaScript is loaded?

How many API calls occur?

How many database queries execute?

How much memory is consumed?

Can this be lazy loaded?

Can this be cached?

Can this be streamed?

If optimization is possible without sacrificing maintainability,

optimization is required.

# BUSINESS DOMAIN STANDARDS

Every business module must follow the same engineering philosophy.

Each module must have:

- Independent services
- Independent permissions
- Independent routes
- Independent database models
- Independent history
- Independent dashboard widgets
- Independent reports

Modules communicate through services.

Never access another module's repository directly.

---

# CURRENT BUSINESS MODULES

The current production scope includes five modules.

1. Vehicle Management

Responsible for:

- Official vehicles
- Operational vehicles
- Vehicle documents
- Maintenance
- Fuel history
- Driver assignment
- Vehicle borrowing
- Vehicle return

---

2. Inventory Management

Responsible for:

- Furniture
- Electronics
- Office equipment
- Employee assets
- Asset assignment
- Asset movement
- Asset condition

---

3. Linen Management

Responsible for:

- Hotel linen
- Guest house linen
- Inventory count
- Washing cycle
- Replacement
- Damage report

---

4. Supply Management

Responsible for:

- Consumable goods

Examples

Paper

Ink

Stationery

Cleaning supplies

Pantry supplies

Medicine

Stock movement

Stock adjustment

Minimum stock

Reorder reminder

---

5. Art Collection Management

Responsible for

Paintings

Statues

Historical artifacts

Decorations

Traditional objects

Museum assets

Every artwork should have complete digital documentation.

---

# ASSET PASSPORT

Every asset inside SENOPATI owns a digital passport.

The Asset Passport becomes the single source of truth.

Every asset must include

Unique Identifier

QR Code

Category

Current Location

Previous Locations

Current Responsible Person

Asset Status

Condition

Photos

Purchase Information

Maintenance History

Movement Timeline

Documents

Notes

Audit History

Every asset page should immediately answer

Who owns this?

Where is it?

Who moved it?

When was it moved?

Why was it moved?

Who approved it?

When was it last inspected?

---

# LOCATION HIERARCHY

Every asset location follows one hierarchy.

Area

↓

Building

↓

Floor

↓

Zone

↓

Room

↓

Storage

↓

Asset

The hierarchy must remain flexible.

Never hardcode buildings.

Never hardcode room names.

Everything must be configurable.

---

# INTERACTIVE MAP ENGINE

The Interactive Map is the primary navigation system.

It is NOT an optional visualization.

Users should naturally navigate assets through the map.

Navigation flow

Area

↓

Building

↓

Floor

↓

Room

↓

Asset

Each level loads only its own data.

Never preload every building.

Never preload every room.

Always lazy load.

---

# MAP ENGINE REQUIREMENTS

Every room should display

Asset Count

Asset Categories

Occupancy

Status

Recent Activities

Clicking a room opens

Room Detail

↓

Asset List

↓

Asset Passport

The map should remain smooth even with thousands of assets.

---

# ISOMETRIC DESIGN

The interactive map uses isometric illustrations.

Reasons

Better orientation

Better usability

Premium appearance

Easy navigation

Future Digital Twin compatibility

SVG is preferred for static floor layout ONLY. Dynamic asset markers MUST be rendered as lightweight absolute-positioned HTML elements overlaid on the SVG container to minimize DOM node count inside the SVG.

Avoid WebGL unless necessary.

---

# QR CODE STRATEGY

Every physical asset must have

One

Unique

QR Code.

QR Code opens

Asset Passport

Never open raw database pages.

The QR destination should always be a dedicated asset view.

---

# RFID STRATEGY

RFID support must remain optional.

The architecture should allow RFID integration without changing existing business logic.

Current priority

QR Code

Future

RFID

IoT

BLE

Indoor Positioning

---

# STOCK OPNAME WORKFLOW

Stock opname must become digital.

Workflow

Create Session

↓

Assign Auditor

↓

Generate Asset List

↓

Navigate Using Map (Cache map data locally on entry)

↓

Scan QR (Queue scan locally if offline)

↓

Verify Condition

↓

Verify Location

↓

Add Notes

↓

Complete Session

↓

Generate Report

No spreadsheet should be required. The application MUST handle network interruptions by buffering scan results in local storage.

---

# MOVEMENT HISTORY

Whenever an asset changes location

Automatically create

Movement History

Movement history contains

Old Location

New Location

Date

Time

Responsible User

Reason

Notes

History cannot be deleted.

---

# MAINTENANCE HISTORY

Assets requiring maintenance must store

Maintenance Date

Maintenance Type

Vendor

Cost

Notes

Documents

Photos

Upcoming Schedule

Never overwrite maintenance history.

Always append.

---

# VEHICLE MODULE

Vehicle management includes

Master Vehicle

Driver Assignment

Fuel History

Maintenance

Insurance

Taxes

Borrowing

Return

Documents

Inspection

Reminder

Each vehicle should expose

Current Status

Current Driver

Current Location

Next Service

Fuel Trend

Maintenance Timeline

---

# ART COLLECTION MODULE

Art Collection should become one of the flagship modules.

Every artwork should contain

High Resolution Photos

Artist

Origin

Material

Year

Dimensions

Estimated Value

Preservation Notes

Historical Description

Display Location

Movement History

Restoration History

Artwork Timeline

Future support

3D Viewer

360° Viewer

Environmental Monitoring

---

# INVENTORY MODULE

Inventory focuses on reusable assets.

Every inventory item stores

Asset Code

Condition

Responsible Employee

Current Room

Purchase Date

Warranty

Supplier

Maintenance

History

Movement

Assignment

---

# LINEN MODULE

Linen focuses on lifecycle.

Workflow

Purchase

↓

Storage

↓

Usage

↓

Laundry

↓

Inspection

↓

Repair

↓

Replacement

↓

Archive

Every linen should have usage statistics.

---

# SUPPLY MODULE

Supply focuses on stock movement.

Every movement generates history.

Incoming

Outgoing

Adjustment

Damage

Expired

Lost

Stock must always be calculated automatically.

Manual calculation should never be necessary.

---

# DASHBOARD PHILOSOPHY

Dashboard is not a report.

Dashboard is a decision support tool.

Display

Summary

Alerts

Notifications

Recent Activities

Pending Tasks

Map Overview

Statistics

Health Indicators

Never display huge tables on dashboard.

---

# SEARCH EXPERIENCE

Search should feel instant.

Requirements

Debounce

Search Suggestions

Category Filter

Location Filter

Status Filter

Recent Searches

Search History

Every search should prioritize relevance.

---

# NOTIFICATION ENGINE

Notification Types

Maintenance Reminder

Tax Reminder

Inspection Reminder

Low Stock

Missing Asset

Movement Alert

Audit Assignment

Approval Request

Notifications should be actionable.

Users should immediately know what action to take.

---

# REPORTING

Reports should be generated dynamically.

Supported formats

PDF

Excel

CSV

Reports should support

Date Range

Category

Building

Room

PIC

Condition

Status

No report should require manual spreadsheet editing.

# AI DEVELOPMENT WORKFLOW

Every request must follow the workflow below.

The AI must NEVER skip steps.

Request

↓

Understand Business Requirement

↓

Understand User Goal

↓

Read Existing Architecture

↓

Search Existing Implementation

↓

Search Existing Components

↓

Determine Business Impact

↓

Determine Architecture Impact

↓

Determine Performance Impact

↓

Determine Security Impact

↓

Determine Database Impact

↓

Implementation Plan

↓

Implementation

↓

Self Review

↓

Optimization

↓

Validation

↓

Done

Implementation without analysis is prohibited.

---

# TASK EXECUTION STRATEGY

Before writing any code,

the AI must answer the following questions.

What problem is being solved?

Who will use this?

Does a similar implementation already exist?

Can an existing solution be reused?

Will this affect another module?

Will this increase technical debt?

Is this scalable?

If uncertainty exists,

stop implementation

and explain the uncertainty first.

Never guess business requirements.

---

# REUSE POLICY

Before creating

Component

Hook

Service

Utility

Repository

DTO

Validator

Constant

Type

Schema

The AI MUST search for existing implementations.

Reuse is preferred.

Extension is second.

Creation is the final option.

---

# REFACTORING POLICY

The AI should continuously improve existing code.

Refactor when

Logic is duplicated

Component is too large

Service has multiple responsibilities

Performance is degraded

Naming becomes confusing

Complexity increases

Never refactor only for personal preference.

Every refactor must improve maintainability.

---

# FILE CREATION POLICY

Creating files is inexpensive.

Maintaining files is expensive.

Never create unnecessary files.

Never create files with only one trivial function.

Prefer logical grouping.

Avoid excessive fragmentation.

---

# NAMING CONVENTIONS

Use descriptive names.

Good

VehicleHistoryService

InventoryMovementRepository

AssetPassportCard

Bad

Helper

Data

Manager

Util2

NewService

Temp

File names should immediately describe responsibility.

---

# COMMENT POLICY

Code should explain itself.

Comments are reserved for

Business rules

Complex algorithms

Government regulations

Architecture decisions

Never explain obvious code.

Bad

// increment i

i++

Good

// Asset history must never be modified after creation.

---

# CODE STYLE

Write code for humans first.

Optimize readability.

Prefer explicit code.

Avoid clever tricks.

Avoid deeply nested logic.

Maximum nesting depth

Three.

Extract functions when necessary.

---

# ERROR RECOVERY

Every failure should provide

Meaningful feedback

Possible cause

Recommended action

Never show

Unknown Error

Something went wrong

Without context.

---

# LOGGING STRATEGY

Log only meaningful events.

Do not log everything.

Examples

Authentication

Authorization

Permission Denied

Import

Export

Transfer

Maintenance

Audit

Critical Errors

Avoid noisy logs.

Logs should help investigations.

---

# TESTING STRATEGY

Critical business logic must be tested.

Priority

Unit Test

↓

Integration Test

↓

End-to-End Test

Business rules require higher coverage than presentation components.

---

# TEST REQUIREMENTS

Every important feature should verify

Success Case

Failure Case

Permission Case

Validation Case

Edge Case

Empty State

Unexpected Input

---

# SECURITY CHECKLIST

Before deployment verify

Authentication

Authorization

Rate Limiting

SQL Injection

XSS

CSRF

Input Validation

File Upload Validation

Environment Variables

Secret Management

Dependency Audit

Security is never optional.

---

# PERFORMANCE CHECKLIST

Before merging verify

Bundle Size

API Response

Database Query Count

Image Optimization

Lazy Loading

Caching

Code Splitting

Virtualization

Memory Usage

Network Requests

Performance regressions are unacceptable.

---

# ACCESSIBILITY CHECKLIST

Every page should support

Keyboard Navigation

Focus Management

Readable Typography

Color Contrast

Screen Reader

Accessible Forms

Meaningful Labels

---

# GIT STRATEGY

Main Branch

Production Ready

Develop Branch

Integration

Feature Branch

One feature only.

Hotfix Branch

Critical production fixes.

Never commit unfinished work to main.

---

# COMMIT STANDARD

Use Conventional Commits.

Examples

feat:

fix:

docs:

style:

refactor:

test:

perf:

build:

ci:

chore:

Avoid meaningless commit messages.

---

# PULL REQUEST CHECKLIST

Every Pull Request must answer

What problem is solved?

What files changed?

Does this affect performance?

Does this affect security?

Does this require database migration?

How was it tested?

What risks exist?

---

# DEFINITION OF DONE

A feature is complete only when

Business requirement satisfied

Architecture respected

Code reviewed

Documentation updated

Tests passing

No TypeScript errors

No ESLint errors

Performance acceptable

Accessibility verified

Security verified

Responsive layout verified

Permission verified

Loading state exists

Error state exists

Empty state exists

No console errors

No duplicated logic

No unnecessary dependency

Only then may a feature be considered complete.

---

# ENGINEERING COMMANDMENTS

1.

Architecture before implementation.

2.

Business value before technology.

3.

Performance is a feature.

4.

Security is mandatory.

5.

Everything important has history.

6.

Every asset has identity.

7.

Every module owns its business logic.

8.

Never duplicate code.

9.

Reuse before creating.

10.

Never guess business rules.

11.

Never trust client input.

12.

Every screen must handle loading, empty, success, and error states.

13.

Never sacrifice maintainability for speed.

14.

Prefer simple solutions over clever ones.

15.

Always leave the codebase better than you found it.

---

# FUTURE ROADMAP

The architecture should be capable of supporting future features without major redesign.

Potential future modules include

Procurement

SIMAK BMN Synchronization

SAKTI Integration

RFID Integration

IoT Sensors

Indoor Positioning

Digital Twin

AI Assistant

Predictive Maintenance

Asset Lifecycle Analytics

GIS Integration

Building Information Modeling (BIM)

Mobile Application

Offline Inspection

Digital Signature

Real-Time Notifications

Every engineering decision today should keep these future possibilities open.

---

# FINAL DIRECTIVE

SENOPATI is not simply an inventory system.

It is a long-term digital asset ecosystem designed for internal government operations.

Every line of code should contribute to the following goals:

Reliable operations.

Excellent performance.

Clear architecture.

Long-term maintainability.

Elegant user experience.

Accurate asset tracking.

Complete historical accountability.

The AI is responsible not only for producing working code, but also for protecting the quality, consistency, and future evolution of the entire system.

Whenever there is a conflict between speed and quality,

quality must always win.

# DATABASE ARCHITECTURE

## Database Philosophy

SENOPATI adopts an Asset-Centric Database Architecture.

Every government-owned object is considered an Asset.

Modules such as Vehicle, Inventory, Linen, Supply, and Art Collection are not independent systems.

They are specialized representations of a single Asset entity.

This approach eliminates duplicated data, simplifies future expansion, and provides a unified asset lifecycle.

The Asset table is the heart of the system.

Every module extends the Asset model instead of replacing it.

---

# DATABASE DESIGN PRINCIPLES

The database must satisfy the following principles.

- Normalize business data.
- Avoid duplicated information.
- Every important action must generate historical records.
- Every important table must contain audit fields.
- Soft Delete is mandatory.
- UUID is the primary identifier.
- Foreign Keys are required.
- Index frequently searched columns.
- Never store calculated values unless justified.
- Database integrity is more important than development speed.

---

# DATABASE MODULES

The database is divided into logical domains.

Authentication & Authorization

Master Data

Asset Core

Asset Details

Location

Assignment

Movement

Stock Opname

Maintenance

Documents

Media

Tracking

Notifications

Reporting

Audit

Configuration

Future Expansion

---

# AUTHENTICATION TABLES

users

roles

permissions

role_permissions

user_roles

sessions

login_histories

password_reset_tokens

---

# MASTER DATA TABLES

asset_categories

asset_types

asset_conditions

asset_statuses

buildings

floors

zones

rooms

departments

employees

vendors

manufacturers

maintenance_types

movement_types

document_types

---

# ASSET CORE TABLES

assets

Asset is the parent entity of every government-owned object.

Every module extends this table.

Important fields include

UUID

Asset Code

Asset Name

Category

Type

Status

Condition

Current Location

Current PIC

QR Code

Barcode

RFID Identifier

Purchase Information

Lifecycle Status

Audit Fields

---

# ASSET DETAIL TABLES

vehicle_details

inventory_details

art_details

linen_details

supply_details

Each table stores only data specific to that asset type.

Business-specific information must never be mixed into the Asset table.

---

# LOCATION TABLES

asset_locations

asset_location_histories

building_maps

room_maps

Location hierarchy

Area

↓

Building

↓

Floor

↓

Zone

↓

Room

↓

Asset

Every movement automatically updates the current location and creates a movement history.

---

# ASSIGNMENT TABLES

asset_assignments

asset_assignment_histories

employee_asset_histories

Every assignment must be historically traceable.

Assignments must never overwrite previous records.

---

# MOVEMENT TABLES

asset_movements

asset_transfer_requests

asset_transfer_histories

Every movement records

Previous Location

New Location

Responsible User

Reason

Timestamp

Supporting Documents

---

# STOCK OPNAME TABLES

stock_opnames

stock_opname_items

stock_opname_results

stock_opname_notes

stock_opname_photos

stock_opname_histories

Stock opname should become a fully digital workflow.

Manual spreadsheet reconciliation is prohibited.

---

# MAINTENANCE TABLES

maintenances

maintenance_items

maintenance_schedules

maintenance_documents

maintenance_histories

Maintenance records are append-only.

Historical maintenance data must never be modified.

---

# DOCUMENT TABLES

asset_documents

document_types

Supporting documents include

Purchase Documents

Warranty

Maintenance Files

Inspection Reports

Certificates

Invoices

---

# MEDIA TABLES

asset_media

Media Types

Photo

Video

360 Image

Blueprint

Every asset may contain multiple media files.

---

# TRACKING TABLES

qr_scan_logs

rfid_scan_logs

asset_tracking_logs

Tracking architecture must support

QR Code

RFID

BLE

Indoor Positioning

IoT Devices

Current implementation prioritizes QR Code.

Future technologies should integrate without changing business logic.

---

# NOTIFICATION TABLES

notifications

notification_templates

notification_logs

Notification categories include

Maintenance

Audit

Low Stock

Movement

Reminder

Approval

Assignment

---

# REPORTING TABLES

generated_reports

report_templates

report_histories

Reports should always be generated dynamically.

Never store duplicated reporting data.

---

# AUDIT TABLES

audit_logs

activity_logs

system_logs

Audit records are immutable.

System administrators may view them but never modify them.

---

# CONFIGURATION TABLES

system_settings

application_configs

feature_flags

system_announcements

Avoid hardcoded business configurations.

Business behavior should be configurable whenever appropriate.

---

# FUTURE TABLES

Reserved for future development.

Potential additions include

AI Analytics

Digital Twin

Predictive Maintenance

IoT Monitoring

GIS Integration

SIMAK BMN Synchronization

SAKTI Synchronization

These modules should integrate without redesigning the existing schema.

---

# DATABASE RELATIONSHIP

Authentication

↓

Users

↓

Roles

↓

Permissions

↓

Assets

↓

Asset Details

↓

Location

↓

Assignment

↓

Movement

↓

Maintenance

↓

Media

↓

Documents

↓

History

↓

Reporting

Every module ultimately references the Asset table.

The Asset table is the single source of truth.

---

# DATABASE NAMING CONVENTION

Tables

Plural

Example

users

assets

roles

Columns

snake_case

Examples

created_at

updated_at

asset_code

purchase_price

Foreign Keys

table_name_id

Examples

user_id

asset_id

room_id

building_id

UUID Columns

Always named

id

Business Identifier

Always stored separately

Example

asset_code

Vehicle Plate Number

Inventory Number

QR Number

These are not primary keys.

---

# REQUIRED AUDIT COLUMNS

Every business table must include

created_at

updated_at

deleted_at

created_by

updated_by

deleted_by

Soft Delete is mandatory.

Hard Delete is prohibited except for system maintenance.

---

# INDEXING STRATEGY

Create indexes for

Asset Code

QR Code

RFID Identifier

Building

Room

Category

Status

Current PIC

Created Date

Movement Date

Composite Indexes on History Tables:
- asset_id + created_at (on location, assignment, transfer, and maintenance history tables)

Frequently searched columns should always be indexed.

---

# DATABASE PERFORMANCE RULES

Never use SELECT *

Always request only required fields.

Avoid N+1 Queries.

Use transactions for critical operations.

Use pagination for large datasets.

Avoid unnecessary joins.

Measure query performance continuously.

Target

Average Query

<100ms

Complex Query

<300ms

Slow queries must be optimized immediately.

---

# FINAL DATABASE DIRECTIVE

The database is the foundation of SENOPATI.

Every schema decision must prioritize

Consistency

Integrity

Maintainability

Scalability

Performance

Future Expansion

The database should remain capable of supporting millions of historical records without requiring architectural redesign.

# DATABASE SCHEMA BLUEPRINT

This section defines the canonical database schema.

The schema described here is the single source of truth.

Every migration, Prisma model, entity, repository, API, and frontend implementation MUST follow this blueprint.

The AI MUST NOT invent additional columns without strong business justification.

---

# DATABASE OVERVIEW

Database Type

MySQL

ORM

Prisma

Primary Key

UUID

Naming Convention

snake_case

Soft Delete

Required

Audit Fields

Required

Timestamp

UTC

Character Set

utf8mb4

Storage Engine

InnoDB

---

# RELATIONSHIP OVERVIEW

Authentication

↓

Users

↓

Departments

↓

Asset Assignments

↓

Assets

↓

Asset Details

↓

Locations

↓

Movement

↓

Maintenance

↓

Media

↓

Documents

↓

Audit

↓

Reporting

---

# TABLE : users

Purpose

Stores every authenticated user.

Columns

id UUID PK

employee_id FK employees (mandatory for active operational staff)

role_id FK roles

username VARCHAR(100)

email VARCHAR(255)

password VARCHAR(255)

is_active BOOLEAN

last_login_at TIMESTAMP

created_at

updated_at

deleted_at

created_by

updated_by

deleted_by

Indexes

username

email

employee_id

---

# TABLE : roles

id

name

description

created_at

updated_at

---

# TABLE : permissions

id

module

action

description

---

# TABLE : role_permissions

role_id

permission_id

Composite Primary Key

(role_id, permission_id)

---

# TABLE : departments

id

department_name

department_code

description

---

# TABLE : employees

id

employee_name

nip

department_id

position

phone

email

status

---

# TABLE : buildings

id

building_code

building_name

description

latitude

longitude

is_active

---

# TABLE : floors

id

building_id

floor_number

floor_name

---

# TABLE : zones

id

floor_id

zone_name

description

---

# TABLE : rooms

id

zone_id

room_code

room_name

room_function

description

---

# TABLE : asset_categories

Examples

Vehicle

Inventory

Linen

Supply

Art

Columns

id

category_name

description

---

# TABLE : asset_types

id

category_id

type_name

description

---

# TABLE : asset_statuses

Examples

Active

Maintenance

Borrowed

Disposed

Lost

Archived

---

# TABLE : asset_conditions

Excellent

Good

Minor Damage

Major Damage

Broken

Lost

---

# TABLE : assets

This is the heart of the system.

Columns

id UUID

asset_code

asset_name

category_id

type_id

status_id

condition_id

room_id

current_pic_id

barcode

qr_code

rfid_uid

bmn_code (VARCHAR, nullable)

nup (INT, nullable)

sakti_id (VARCHAR, nullable)

brand

model

serial_number

purchase_date

purchase_price

vendor_id

manufacturer_id

useful_life

depreciation_method

description

is_active

created_at

updated_at

deleted_at

created_by

updated_by

deleted_by

Indexes

asset_code

qr_code

room_id

status_id

condition_id

category_id

current_pic_id

---

# TABLE : vehicle_details

asset_id PK FK assets

plate_number

engine_number

chassis_number

vehicle_type

fuel_type

transmission

color

production_year

engine_capacity

fuel_capacity

odometer

insurance_number

insurance_expiry

stnk_number

stnk_expiry

kir_expiry

next_service

---

# TABLE : inventory_details

asset_id

inventory_number

material

size

weight

manufacturer

warranty_expiry

expected_lifetime

---

# TABLE : art_details

asset_id

artist

art_style

origin

creation_year

material

width

height

depth

estimated_value

historical_description

preservation_notes

restoration_notes

---

# TABLE : linen_details

asset_id

linen_type

material

size

color

wash_cycle

replacement_cycle

---

# TABLE : supply_details

asset_id

unit

minimum_stock

maximum_stock

reorder_level

current_stock

expiration_date

---

# TABLE : asset_assignments

id

asset_id

employee_id

assigned_at

returned_at

assignment_notes

status

---

# TABLE : asset_movements

id

asset_id

from_room_id

to_room_id

movement_type_id

reason

moved_by

movement_date

approved_by

notes

---

# TABLE : maintenances

id

asset_id

maintenance_type_id

vendor_id

maintenance_date

cost

description

next_schedule

status

---

# TABLE : stock_opnames

id

session_name

building_id

created_by

started_at

finished_at

status

---

# TABLE : stock_opname_items

id

stock_opname_id

asset_id

expected_room

actual_room

condition (VARCHAR - stores condition status text snapshot)

result

notes

checked_by

checked_at

---

# TABLE : asset_media

id

asset_id

file_name

file_type

file_size

mime_type

storage_path

thumbnail_path

uploaded_by

uploaded_at

---

# TABLE : asset_documents

id

asset_id

document_type_id

document_name

file_path

version

uploaded_by

uploaded_at

---

# TABLE : notifications

id

title

message

receiver_id

type

priority

is_read

created_at

---

# TABLE : generated_reports

id

report_name

report_type

generated_by

generated_at

download_path

---

# TABLE : audit_logs

id

user_id

module

action

table_name

record_id

old_value

new_value

ip_address

user_agent

created_at

---

# FOREIGN KEY RULES

Every foreign key must use

ON UPDATE CASCADE

ON DELETE RESTRICT

unless business rules explicitly require another behavior.

---

# INDEX STRATEGY

Always index

UUID

Asset Code

QR Code

RFID UID

Room ID

Building ID

Category ID

Status ID

Employee ID

Movement Date

Purchase Date

Never create unnecessary indexes.

Measure before adding.

---

# MIGRATION RULES

Every schema change requires

Migration

Review

Rollback Strategy

Data Integrity Validation

Never edit production schema manually.

Always use Prisma Migration.

---

# DATABASE QUALITY GATE

The schema is considered complete only if

✓ Third Normal Form respected

✓ No duplicated business data

✓ Soft Delete implemented

✓ Audit fields available

✓ Foreign Keys validated

✓ Indexes reviewed

✓ Query performance acceptable

✓ History preserved

✓ Future expansion possible

If any item fails,

the schema must be redesigned before implementation.

END OF DATABASE SCHEMA BLUEPRINT

# IMPLEMENTATION PROTOCOL

AI MUST NEVER implement multiple phases at once.

Every phase must follow this workflow:

1. Explain the implementation plan.
2. Implement only the current phase.
3. Validate the implementation.
4. Summarize the completed work.
5. Stop and wait for user approval.

The AI MUST NOT continue automatically to the next phase.

The user is the Software Architect and has final authority over every architectural and implementation decision.

END OF DOCUMENT

---
> Source: [F4KHR111/senopati](https://github.com/F4KHR111/senopati) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
