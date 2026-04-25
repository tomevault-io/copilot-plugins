## bala-shadn-registry

> Payment routes implementation guide for clinic system


# Payment Routes Implementation Guide

## Overview
This rule provides guidance for implementing payment management functionality in the clinic system, including payment listing, creation, editing, and invoice generation.

## Core Requirements

### Data Integration
- **MUST** use [lib/data/mockDataService.ts](mdc:lib/data/mockDataService.ts) for payment data
- **MUST** use `MockDataService.getPaymentsByClinic(clinicName)` method
- **MUST** leverage existing `Payment` and `PaymentLineItem` interfaces
- **MUST** integrate with clinic context using [lib/data/clinics.ts](mdc:lib/data/clinics.ts)
- **MUST** use [lib/route-utils.ts](mdc:lib/route-utils.ts) `generateLink()` for navigation

### Route Structure
Replace content in [app/clinic/[clinic]/payments/page.tsx](mdc:app/clinic/[clinic]/payments/page.tsx) with:
- Payment listing with search, filtering, pagination
- Status-based filtering (completed, pending, failed, refunded)
- Integration with existing clinic layout from [app/clinic/[clinic]/layout.tsx](mdc:app/clinic/[clinic]/layout.tsx)

### Required New Routes
```
/app/clinic/[clinic]/payments/
├── page.tsx (main payments list)
├── [id]/page.tsx (payment details)
├── [id]/edit/page.tsx (edit payment)
├── new/page.tsx (create payment)
└── invoice/[id]/page.tsx (printable invoice)
```

## Design Patterns to Follow

### Follow Existing Patterns
- **Navigation**: Use patterns from [app/clinic/[clinic]/clients/page.tsx](mdc:app/clinic/[clinic]/clients/page.tsx)
- **Forms**: Follow patterns from [app/clinic/[clinic]/clients/new/page.tsx](mdc:app/clinic/[clinic]/clients/new/page.tsx)
- **Details Views**: Use patterns from [app/clinic/[clinic]/orders/[id]/page.tsx](mdc:app/clinic/[clinic]/orders/[id]/page.tsx)
- **Responsive Design**: Maintain responsive patterns as seen in [app/clinic/[clinic]/clients/all/page.tsx](mdc:app/clinic/[clinic]/clients/all/page.tsx)

### UI Components to Use
- [components/ui/card.tsx](mdc:components/ui/card.tsx) for layout structure
- [components/ui/button.tsx](mdc:components/ui/button.tsx) for actions
- [components/ui/badge.tsx](mdc:components/ui/badge.tsx) for payment status
- [components/ui/form/FormWrapper.tsx](mdc:components/ui/form/FormWrapper.tsx) for payment forms
- [components/ui/table/DataTable.tsx](mdc:components/ui/table/DataTable.tsx) for payment listings

## Status Color Coding
```tsx
const getStatusColor = (status: string) => {
  switch (status) {
    case 'completed': return 'bg-green-100 text-green-800';
    case 'pending': return 'bg-yellow-100 text-yellow-800';
    case 'failed': return 'bg-red-100 text-red-800';
    case 'refunded': return 'bg-blue-100 text-blue-800';
    default: return 'bg-gray-100 text-gray-800';
  }
};
```

## Responsive Design Requirements
- **Mobile-first approach** following [components/ui/client/ClientCard.tsx](mdc:components/ui/client/ClientCard.tsx)
- **Grid layouts**: Use `grid-cols-1 md:grid-cols-2 lg:grid-cols-3` pattern
- **Flexible layouts**: Use `flex-col sm:flex-row` for action bars
- **Responsive typography**: `text-2xl sm:text-3xl` for headings

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/conanedogawa23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
