## vehicle-insurance-system

> Vehicle Management System (VMS)


Vehicle Management System (VMS)

Authoritative Architecture & Domain Rules

1. System Reorientation (Non-Negotiable)

The system is vehicle-centric, not insurance-centric.

A Vehicle is the core aggregate root.
Insurance, LATRA, permits, inspections, and ownership records orbit the vehicle.

If a design treats Insurance as the main entity, it is wrong.

2. Core Domain Aggregates
2.1 Vehicle (Primary Aggregate Root)

Each vehicle must have:

Unique identifier (UUID)

Registration number (plate)

Chassis number

Engine number

Vehicle type (private, commercial, motorcycle, truck, etc.)

Usage category (private, public transport, cargo, rental)

Status (active, suspended, retired)

A vehicle exists independently of insurance or permits.

2.2 Insurance (Sub-Domain)

Insurance records:

Belong to one vehicle

Have validity periods

Cannot overlap for the same vehicle

Can be historical (expired, cancelled)

Rules:

A vehicle may have zero or many insurance records

Only one active insurance per vehicle at a time

Insurance never owns the vehicle

2.3 LATRA & Regulatory Permits (Sub-Domain)

The system must support multiple permit types, including but not limited to:

LATRA road license

Route permit

PSV badge

Inspection certificate

Temporary permits

Rules:

Permits are vehicle-bound

Permit types must be extensible (do not hardcode)

Each permit has:

Issuing authority

Issue date

Expiry date

Status (valid, expired, revoked)

2.4 Ownership & Assignment

Vehicles may be:

Owned by individuals

Owned by companies

Assigned to drivers or employees

Rules:

Ownership history must be preserved

Assignment does not imply ownership

A vehicle can exist without an active owner record

3. Temporal Integrity Rules

Time matters. The system must respect it.

No overlapping validity periods for:

Insurance

Same-type permits

Historical records are immutable

Edits create new versions, not silent overwrites

4. Data Modeling Rules

Use Vehicle as the foreign key anchor

Avoid circular dependencies

Prefer:

OneToMany for records (insurance, permits)

ManyToMany only when unavoidable

Soft delete for regulatory and financial records

5. Application Layer Responsibilities

The system must provide:

Vehicle compliance status (insurance + permits)

Expiry alerts (background tasks)

Historical audit trails

Role-based access:

Admin

Company manager

Officer / staff

Read-only auditor

6. Scalability & Extensibility Rules

New permit types must require zero schema rewrites

Regulatory authorities must be configurable

Country-specific rules must be isolated, not baked into core logic

7. What This System Is NOT

Not an insurance sales platform

Not a payment gateway

Not a government system replacement

It is a vehicle lifecycle & compliance intelligence system.

8. Failure Conditions (Instant Rejection)

Any implementation that:

Treats insurance as the root entity

Hardcodes LATRA logic

Deletes historical compliance data

Allows multiple active insurance per vehicle

…is architecturally invalid.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dkalal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
