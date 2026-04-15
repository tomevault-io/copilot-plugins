## adaptusdms

> - **Theme:** Professional Dashboard, clean whites/grays with specific color accents.

# Project Context: Adaptus DMS

## Design System
- **Theme:** Professional Dashboard, clean whites/grays with specific color accents.
- **Chart Colors:** 
  - Leads: Multi-colored bars.
  - Tasks: Yellow (1-10 days), Pink (>10 days).
  - Deals: Teal Pie Chart.
  - Inventory: Light Blue Area Chart.

## Critical Business Logic Rules
1. **Duplicate Customers:** Always check Phone and Name on entry. Flag immediately.
2. **Inventory Math:** 
   - Gross Profit = Retail Price - (Purchase Price + Recon Cost + Extra Costs).
3. **Test Drive Compliance:** Cannot schedule if License Expired.
4. **Lead Pipeline:** Statuses must match: "Not Started", "In Progress", "Qualified", "Lost", "Won".

## Navigation Structure
- **Dashboard:** Sales summary & task alerts.
- **Inventory:** The core vehicle grid.
- **Leads:** Kanban or List view of prospects.
- **Deals:** Active negotiations.
- **Financials:** Invoices & Expense ledger.
- **Users:** Staff management.
- **Settings:** Business Profile (Tax IDs), Personal Profile.

## Tech Constraints
- Use Recharts for all visualizations.
- Use React Hook Form for complex forms (Vehicle Entry).
- Use Zod for validation.
- Use Shadcn/UI for tables and cards.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abhishekkushwaha1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
