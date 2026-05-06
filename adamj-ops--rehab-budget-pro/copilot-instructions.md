## roadmap

> > See `STATUS.md` in project root for the canonical implementation status.

# Development Roadmap

> See `STATUS.md` in project root for the canonical implementation status.
> This file provides quick reference for development planning.

## ✅ Completed Features

### Phase 1: Core Foundation
- [x] Three-column budget model (underwriting/forecast/actual)
- [x] Project creation with Google Places autocomplete
- [x] 18 pre-seeded budget categories on project creation
- [x] Budget display with inline editing
- [x] Deal Summary tab with MAO calculations
- [x] Cost Reference tab (Minneapolis pricing)

### Phase 2: Full CRUD Operations
- [x] Budget item add/edit/delete
- [x] Bulk operations (select, status update, delete)
- [x] Vendor CRUD with tags and contact history
- [x] CSV import/export for vendors
- [x] Draw management with status workflow

### Phase 3: Enhanced Features
- [x] Photo upload per line item
- [x] Photo gallery with type classification
- [x] Kanban pipeline dashboard
- [x] Portfolio health metrics
- [x] Drag & drop project status changes

## 🚧 In Progress

### PDF Exports
- [ ] Underwriting summary PDF
- [ ] Full project report PDF
- [ ] Draw request PDF with photos

### Drag & Drop Reordering
- [ ] Reorder budget categories
- [ ] Reorder line items within category
- [ ] Persist sort_order to database

## 📋 Planned Features

### High Priority
1. **User Authentication**
   - Supabase Auth integration
   - Protected routes
   - RLS policy enforcement

2. **PDF Exports**
   - Underwriting summary for lenders
   - Investor packets
   - Draw documentation

3. **Real-time Updates**
   - Supabase subscriptions
   - Live collaboration

### Medium Priority
4. **Budget Templates**
   - Save budget structure as template
   - Apply template to new projects
   - Share templates

5. **Advanced Analytics**
   - Historical performance tracking
   - Cost benchmarking
   - Trend analysis

6. **Mobile Optimization**
   - Responsive budget table
   - Touch-friendly interactions
   - Offline capability

### Low Priority
7. **Team Collaboration**
   - Multi-user projects
   - Role-based permissions
   - Activity feed

8. **Integrations**
   - QuickBooks sync
   - Contractor bid requests
   - Lender portals

## Technical Debt

### Must Address
- [ ] Error boundaries for all tabs
- [ ] Loading skeletons for async operations
- [ ] Form validation with error messages
- [ ] Mobile responsive budget table

### Should Address
- [ ] Consolidate Zustand + React Query patterns
- [ ] Add comprehensive error handling
- [ ] Implement keyboard shortcuts
- [ ] Add unit tests for calculations

### Nice to Have
- [ ] Budget version history
- [ ] Undo/redo for edits
- [ ] Offline mode with sync
- [ ] Advanced search/filtering

## Architecture Improvements

### Performance
- [ ] Virtual scrolling for large budget lists
- [ ] Lazy load tabs
- [ ] Image optimization for photos
- [ ] Query result caching

### Developer Experience
- [ ] Storybook for components
- [ ] E2E tests with Playwright
- [ ] CI/CD pipeline
- [ ] API documentation

## Feature Requests (Backlog)

From user feedback:
- Contractor bid comparison tool
- Material cost tracking
- Timeline/Gantt view for projects
- Automated draw scheduling
- Photo annotation
- Receipt OCR

---
> Source: [adamj-ops/rehab-budget-pro](https://github.com/adamj-ops/rehab-budget-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
