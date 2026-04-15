## arogya-sewa-admin-web

> This is the centralized administrative platform for **Arogya Sewa**. It serves as a multi-tenant system where Super Admins manage the global hospital network and Hospital Admins manage their specific facility's operations (doctors, appointments, and billing).

# Arogya Sewa - Admin & Super Admin Portal

This is the centralized administrative platform for **Arogya Sewa**. It serves as a multi-tenant system where Super Admins manage the global hospital network and Hospital Admins manage their specific facility's operations (doctors, appointments, and billing).

## 🚀 Tech Stack

- **Framework:** React 18+ with TypeScript
- **Build Tool:** Vite
- **Styling:** Tailwind CSS
- **UI Components:** Radix UI / Shadcn UI (Card, Button, etc.)
- **Icons:** Lucide React
- **State Management & Fetching:** TanStack Query (React Query)
- **Routing:** React Router DOM
- **Notifications:** React Hot Toast

## 📂 Project Structure

- `src/api/`: Service layer for backend communication (e.g., `doctor.api.ts`).
- `src/features/`: Domain-driven logic. Contains sub-folders like `hospital-admin/doctors` or `hospital-admin/appointments` which house specific hooks and local components.
- `src/pages/`: Route-level components (e.g., `DoctorsPage.tsx`, `AppointmentViewPage.tsx`).
- `src/components/ui/`: Generic, reusable UI primitives (Breadcrumbs, Tables, Modals).
- `src/types/`: Centralized TypeScript definitions for entities like `Doctor`, `Appointment`, and `Payment`.
- `src/lib/`: Utility functions (e.g., `cn` for Tailwind class merging).

## 🔐 Role-Based Access Control (RBAC)

The application implements strict permission boundaries:
- **SUPER_ADMIN:** Global access. Can create/edit any hospital and set up hospital admin accounts, view all users and their details.
- **HOSPITAL_ADMIN:** Scoped access. Can only manage data (doctors, schedules, billing) belonging to their assigned `hospitalId`, departments.

## � Key Features

- **Hospital Management (Super Admin):** Complete CRUD for hospital entities, including location mapping and license verification.
- **Dashboard:** Real-time stats for appointments and quick action shortcuts.
- **Doctor Management:** CRUD operations for medical staff, including department assignment and license tracking.
- **Availability Scheduling:** Managing doctor time slots (Open vs. Booked).
- **Appointment Tracking:** Comprehensive list and detailed view of patient visits.
- **Billing & Payments:** Tracking total amounts, paid status, and transaction history per appointment.

## 📝 Coding Conventions

1. **Component Pattern:** Use functional components with `memo` where performance is critical (e.g., large lists like `AvailabilityRow`).
2. **Data Fetching:** Prefer custom hooks (e.g., `useDoctors`) that wrap TanStack Query for abstraction.
3. **Styling:** Use Tailwind CSS utility classes. For complex conditional logic, use the `cn()` utility.
4. **Date/Time:** Standardize formatting using local-aware `Intl.DateTimeFormat` or helper functions (e.g., `formatDateTime`).
5. **Error Handling:** Implement `Retry` buttons and error states within cards to maintain layout consistency when API calls fail.
6. **Form Handling:** Centralize form logic (e.g., `DoctorForm`) to be shared between "Create" and "Edit" views.

## 🔗 Navigation & Routes

### Super Admin Routes
- `/hospitals`: Global list of registered hospitals.
- `/hospitals/create` | `/hospitals/:id/edit`: Hospital onboarding and metadata management.

### Hospital Admin Routes
- `/dashboard`: Hospital overview.
- `/doctors`: Doctor directory.
- `/doctors/:doctorId`: Specific doctor details and availability.
- `/doctors/create` | `/doctors/:doctorId/edit`: Staff management.
- `/appointments`: Master appointment list with filters.
- `/appointments/:appointmentId`: Detailed breakdown of a specific booking and payment logs.

---
*This document is intended to provide context for AI-assisted development.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sijalneupane)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sijalneupane)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
