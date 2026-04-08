## website

> This document provides context for AI assistants working on the CGMHC International Cohort Explorer project.

# Claude Development Guide

This document provides context for AI assistants working on the CGMHC International Cohort Explorer project.

**Last Updated:** January 26, 2026
**Current Status:** Complete
**Latest Session:** Cohorts page grouping by data access type
**GitHub:** https://github.com/cgmhc/website
**Live Site:** https://cgmhc.github.io/website

## Project Overview

This is an interactive web application for exploring international research cohorts in the Covid-19 and Global Mental Health Consortium (CGMHC).

## Current Implementation Status

### What's Working (Phase 1 & 2)
- ✅ Next.js 14 with TypeScript and Tailwind CSS
- ✅ Excel-to-JSON data conversion pipeline
- ✅ Interactive map view with Leaflet markers
- ✅ List view with cohort table
- ✅ Filter sidebar (location, data access, data categories)
- ✅ Cohort detail pages with full data availability
- ✅ Global search in header
- ✅ Map/List view toggle
- ✅ Investigators page with team profiles, scientific aims, participating cohorts
- ✅ GitHub Pages deployment with auto-deploy

### Future Enhancements (Not Planned)
- Comparison page (`src/app/compare/page.tsx`)

## Key Files and Their Purpose

### Data
- `data/IHCC Covid Mental Health Master Cohort Table.xlsx` - Source data file (Excel, 27 cohorts, 68 columns)
- `public/cohorts.json` - Processed JSON data (regenerate with `npm run convert-data`)
- `public/data/` - Editable data files:
  - `project.json` - Grant info, mission statement, scientific aims
  - `institutions.json` - Institution definitions (MGB, Oxford, USP, etc.)
  - `team.json` - Team members/investigators array
  - `cohort-associations.json` - Cohort-PI mappings and data access info
- `public/headshots/` - Team member photos (mgb/, oxford/, usp/, other/)
- `scripts/convert-excel.js` - Excel to JSON conversion script

### Core Application
- `src/app/page.tsx` - Landing page with consortium overview (SERVER COMPONENT)
- `src/app/explorer.tsx` - Explorer component with map/list views (CLIENT COMPONENT)
- `src/app/explorer/page.tsx` - Explorer page wrapper
- `src/app/cohort/[id]/page.tsx` - Individual cohort detail page (SERVER COMPONENT, statically generated)
- `src/app/investigators/page.tsx` - Team/investigators page (SERVER COMPONENT)
- `src/app/publications/page.tsx` - Publications page placeholder (SERVER COMPONENT)
- `src/app/layout.tsx` - Root layout with FilterProvider

### Components
- `src/components/Map/CohortMap.tsx` - Leaflet map with dynamic import (avoids SSR issues)
- `src/components/Filters/FilterSidebar.tsx` - Filter sidebar with collapsible sections
- `src/components/CohortList/CohortList.tsx` - Table view of cohorts
- `src/components/ui/SiteHeader.tsx` - **Shared navigation header** used by all pages (logo, nav links, optional page title)
- `src/components/ui/Header.tsx` - Explorer-specific header extending SiteHeader with search bar and view toggle

### Data & Types
- `src/lib/types.ts` - TypeScript interfaces (Cohort, FilterState, measure interfaces)
- `src/lib/investigators.ts` - TypeScript interfaces (Investigator, Institution, ScientificAim)
- `src/lib/data.ts` - Data utilities (filtering, grouping, formatting)
- `src/lib/constants.ts` - Country coordinates, colors, map config

### State Management
- `src/contexts/FilterContext.tsx` - Global filter state with React Context
- Manages: filters, viewMode, selectedCohorts for comparison

## Data Structure

The Excel file has a two-row header structure:
- Row 1: Category names (Administration, Characteristics, Life History, etc.)
- Row 2: Column names (Cohort, Country, Baseline age range, etc.)
- Row 3+: Cohort data

### Main Categories
1. **Administration** (cols 1-4): Cohort name, Country, 5yrs waves, Data access
2. **Characteristics** (cols 5-7): Population, Age range, Longitudinal status
3. **Life History** (cols 8-13): Discrimination, Deprivation, Abuse measures
4. **Sociodemographic** (cols 14-24): Demographics and SES variables
5. **Psychological Status** (cols 25-51): Mental health measures
6. **Cognitive Status** (cols 52-56): Cognitive measures
7. **Lifestyle** (cols 57-60): Behavioral measures
8. **Covid-19** (cols 61-68): Pandemic-specific measures

### Data Values
- "yes" / "no" / "n/a" for boolean availability
- Some fields have detailed text (e.g., "yes (KSADS)", "yes (EHR)")
- Numeric values for population counts

## Design Decisions

### Tech Stack Choices
- **Next.js 14 App Router**: Modern React with server components, good for SEO
- **Tailwind CSS**: Rapid UI development, consistent design system
- **Leaflet**: Open-source maps, no API key required
- **Static JSON**: Pre-processed data for fast loading, no backend needed

### Architecture Decisions
- Static site generation where possible for performance
- Client-side filtering for instant user feedback
- URL-based filter state for shareability
- Responsive design with mobile-first approach

## Common Tasks

### Updating Data
When the Excel file is updated:
1. Run `npm run convert-data` to regenerate `public/cohorts.json`
2. Test that all cohorts display correctly
3. Check that filters still work with any new values

### Adding New Filters
1. Add filter type to `src/lib/types.ts`
2. Add filter component in `src/components/Filters/`
3. Update `FilterContext.tsx` with new filter state
4. Update filtering logic in `src/lib/data.ts`

### Adding New Data Fields
1. Update types in `src/lib/types.ts`
2. Update conversion script to include new field
3. Update detail page to display new field
4. Consider if field should be filterable

## Testing

```bash
npm run lint      # Check code style
npm run typecheck # TypeScript validation
npm run test      # Run tests (if configured)
```

## Deployment

The app is deployed to GitHub Pages via GitHub Actions:
1. Push to main branch triggers `.github/workflows/deploy.yml`
2. Build runs with static export (`output: 'export'` in next.config.ts)
3. Auto-deploys to https://cgmhc.github.io/website

Configuration notes:
- `basePath: '/website'` in next.config.ts for GitHub Pages subdirectory
- `images: { unoptimized: true }` required for static export

For local development:
```bash
npm run dev   # Development server at localhost:3000/website (basePath configured)
npm run build # Production build
```

## Performance Considerations

- Map markers are clustered to avoid rendering issues
- Images should be optimized before adding to public/
- Large data transformations happen at build time, not runtime
- Use React.memo for components that re-render frequently

## Accessibility

- All interactive elements should be keyboard accessible
- Map should have text alternatives
- Color is not the only indicator of information
- Filters maintain focus appropriately

## Known Limitations & Issues

- Some cohorts span multiple countries (handled as single entry with first country's coordinates)
- **Age range data issue**: Some Excel cells containing "9-10" are interpreted as dates by Excel, showing as serial numbers like "45179" in the JSON. May need manual correction in Excel or conversion script fix.
- Map uses simple markers, not true clustering library - overlapping markers at same coordinates may be hard to distinguish
- LMIC Cohorts entry has empty country field
- Filter state not yet synced to URL params

## Next Steps (Phase 3 Priorities)

1. **Fix data issues** - Address age range date parsing issue
2. **Comparison page** - Side-by-side cohort comparison
3. **Mobile responsiveness** - Filter drawer, touch-friendly controls
4. **UI polish** - Loading states, animations, better typography

## Updating Data

### Data File Structure
Data is organized into separate files for easier maintenance:

```
public/
├── cohorts.json              # Auto-generated from Excel (don't edit directly)
└── data/
    ├── project.json          # Grant info, mission statement, scientific aims
    ├── institutions.json     # Institution definitions
    ├── team.json             # Team members/investigators
    └── cohort-associations.json  # Cohort-PI mappings
```

### Editing Team Members
To update team member information, edit `public/data/team.json`:
- Add `credentials` (e.g., "MD, PhD")
- Add `title` (e.g., "Associate Professor")
- Change `role` - supports custom strings like "Faculty", "Project Manager", "Collaborator", "Consultant"
- Add `email` for email contact icon
- Add `orcid` ID (e.g., "0000-0001-2345-6789") - will auto-link to orcid.org
- Set `isConsultant: true` to display in Consultants section
- Set `role: "Collaborator"` to display in Collaborators section
- Add new team members following the existing structure

### Editing Project Info
To update mission statement or grant info, edit `public/data/project.json`:
- `scientificMission` - Mission statement text on landing page
- `projectInfo.grantTitle` - Grant title
- `projectInfo.grantNumber` - NIH grant number
- `scientificAims` - Array of scientific aims (displayed on investigators page)

### Editing Institutions
To add or modify institutions, edit `public/data/institutions.json`:
- Add new institution with `id`, `name`, `shortName`, `location`, `website`

### Editing Cohort Associations
To update cohort PIs or add cohorts, edit `public/data/cohort-associations.json`:
- `cohortId` - Must match ID in cohorts.json
- `cohortName` - Display name
- `piNames` - Array of PI names
- `dataAccess` - "DPUK" or "Federated"
- `cgmhcCollaborators` - Array of linked team member IDs (e.g., `[{ "investigatorId": "brunoni-andre" }]`)
  - Links team members to cohorts, displays them in "CGMHC Collaborators" section on cohort detail pages
  - Use the `id` field from team.json (e.g., "brunoni-andre" for André Brunoni)

### Available Institutions
- `mgb` - Mass General Brigham
- `oxford` - University of Oxford
- `usp` - University of São Paulo
- `ufrgs` - Universidade Federal do Rio Grande do Sul
- `harvard` - Harvard University
- `dalhousie` - Dalhousie University

## Recent Changes (January 26, 2026)

### Latest Updates (Session 10 - Cohorts Page Grouping)

**Reorganized Cohorts page** (`src/app/cohorts/page.tsx`) by data access type:
- **DPUK section**: Shows all DPUK cohorts with description and "Learn more" link
- **Federated section**: Shows all federated cohorts with description
- Cohorts sorted alphabetically within each section
- DPUK cohorts display "Available" (green) or "Coming Soon" (amber) badges
- Federated cohorts have no badge (section header provides context)
- Removed redundant subgroup headers (Available/Coming Soon) since badges show status
- Country flag emojis displayed on each card

### Previous Updates (Session 9 - Data Availability Filter Enhancement)

**Replaced category-level filters with condition-specific filters** in Explorer sidebar:
- Removed old "Data Types" section with category checkboxes
- Added new "Data Availability" section with nested condition filters
- Filters organized by category with visual hierarchy (left border + indentation)

**New filter structure** (`src/lib/types.ts` - `conditionFilterGroups`):
- **Psychological Status** (nested): Depression, Anxiety, PTSD, Suicidality, Substance Use, Stress, Loneliness
- **Life History** (nested): Discrimination, Deprivation, Abuse
- **Lifestyle** (nested): Physical Activity, Social Connections, Smoking, Alcohol
- **COVID-19** (nested): COVID Status, Government Policies

**Removed filters**:
- Cognitive/Dementia (not relevant to consortium focus)
- Compare feature removed entirely

**Label updates** (`src/lib/types.ts` - `measureLabels`):
- "Private medical insurance" → "Health insurance"
- "Perceived stress scale" kept as instrument name (filter shows "Stress" as construct)

**Filter logic** (`src/lib/data.ts` - `hasConditionData`):
- Each condition checks relevant underlying measures
- AND logic: selecting multiple conditions shows cohorts with ALL selected
- New conditions added: stress, discrimination, deprivation, abuse, physicalActivity, socialConnections, smoking, alcohol, covidStatus, covidPolicies

**UI changes** (`src/components/Filters/FilterSidebar.tsx`):
- Data Availability section always open by default
- Data Access section always open by default
- Nested items have left border and indentation for visual hierarchy

### Previous Updates (Session 8 - Navigation Header Standardization + Team Emails)

**Added missing team member emails** (`public/data/team.json`):
- Jordan Smoller: jsmoller@mgb.org
- Lorenza Dall'Aglio: ldallaglio@mgh.harvard.edu
- Chris Kennedy: ckennedy13@mgb.org

**Created shared SiteHeader component** (`src/components/ui/SiteHeader.tsx`):
- Consistent navigation across ALL pages (logo left, nav links right)
- Props: `currentPage` (highlights active page), `title`, `subtitle` (for interior pages)
- Active page highlighted with primary color background
- Interior pages show page title and subtitle below the nav bar

**Updated all pages to use SiteHeader**:
- `src/app/page.tsx` - Home page now uses SiteHeader
- `src/app/investigators/page.tsx` - Leadership page
- `src/app/cohorts/page.tsx` - Cohorts listing page
- `src/app/publications/page.tsx` - Publications page
- `src/app/contact/page.tsx` - Contact page

**Updated Explorer Header** (`src/components/ui/Header.tsx`):
- Now matches SiteHeader styling (logo left, nav links right)
- Explorer-specific toolbar below nav with search bar and view toggle
- "Explorer" tab highlighted as active page

**Before vs After**:
- Before: Home page had different nav style than interior pages; Explorer had yet another layout
- After: All pages have identical header: CGMHC logo + nav links, with optional page title below

### Previous Updates (Session 7 - Cohorts Page UI & Data Consistency)

**Cohort Cards UI Improvements** (`src/app/cohorts/page.tsx`):
- **Country flag emojis**: Added flag emoji display in bottom-right corner of each cohort card
  - Supports 20 countries including multi-country cohorts (uses first country's flag)
  - Added `COUNTRY_FLAGS` mapping and `getFlags()` helper function
- **Data access status badges**: Added pill-style badges showing availability status
  - "DPUK (available)" - indigo color for cohorts with data ready on DPUK
  - "DPUK (coming soon)" - indigo color for cohorts preparing for DPUK
  - "Federated" - teal color for cohorts with federated access
  - Badges positioned in bottom-left corner, flags in bottom-right

**Data Consistency Fixes**:
- **Fixed all cohortId mismatches** between cohort-associations.json and cohorts.json
  - 14+ cohorts had full names as IDs (e.g., "einstein-aging-study" → "einstein")
  - This was causing 404 errors on cohort detail pages
- **Synced cohortName values** to match between cohort-associations.json and cohorts.json
  - 11 cohorts had abbreviated names vs full names
- **Fixed dataAccess inconsistencies** in cohorts.json:
  - UKB, BHRS, Qatar Biobank, TILDA: Changed from "DPUK" to "Federated"

**DPUK Categorization Corrections**:
- Only 5 cohorts are currently available on DPUK:
  - ELSA UK, ELSA Brazil, MRC NSHD, Generation Scotland, BDR
- BDR: Changed dpukStatus from "coming_soon" to "available"
- UKB: Changed from DPUK to Federated access

**New CGMHC Collaborators Added** (`public/data/team.json`):
- Sun Jae Jung (MD, PhD) - Yonsei University, linked to CMERC cohort
- Richard Lipton (MD) - Albert Einstein College of Medicine, linked to Einstein cohort
- Mindy Katz (MPH) - Albert Einstein College of Medicine, linked to Einstein cohort

**Missing PI Information Added** (`public/data/cohort-associations.json`):
- MAP-19: Dr. Omid Ebrahimi (also linked as CGMHC Collaborator)
- ORCHESTRA: Dr. Jochen Ohnmacht (Jochen.Ohnmacht@lih.lu)
- CON-VINCE: Dr. Jochen Ohnmacht (Jochen.Ohnmacht@lih.lu)

### Previous Updates (Session 6 - Team Data Updates)
- **Added Jiahui Huang's email** to team.json:
  - Email: `jiahui.huang@psych.ox.ac.uk`
  - Email icon now displays on investigators page
- **Linked André Brunoni to ELSA Brazil**:
  - Added André Brunoni (`brunoni-andre`) as CGMHC Collaborator for ELSA Brazil cohort
  - Profile now appears in "CGMHC Collaborators" section on ELSA Brazil detail page

### Updates (Session 5 - DPUK & Missing Cohorts)
- **Added 3 missing cohorts** to cohorts.json and cohort-associations.json:
  - **MAP-19** (Norway): The Norwegian COVID-19, Mental Health and Adherence Project
    - Omid Ebrahimi linked as CGMHC Collaborator
    - Federated data access
  - **ORCHESTRA** (Italy): EU Horizon 2020 cohort studying COVID-19 long-term effects
    - Federated data access
  - **CON-VINCE** (Luxembourg): COVID-19 prevalence and immunity cohort
    - Federated data access
- **Updated totalCohorts** from 27 to 30
- **Added country coordinates** for Norway, Italy, Luxembourg to `src/lib/constants.ts`
- **DPUK data access context added**:
  - Added `DATA_ACCESS_INFO` and `DATA_ACCESS_LABELS` to `src/lib/constants.ts`
  - FilterSidebar now shows descriptions for DPUK and Federated access types
  - DPUK label changed to "Dementias Platform UK (DPUK)" with "Learn more" link
  - Cohort detail pages: DPUK badge is now clickable, linking to DPUK portal
- **Contact page updates**:
  - Added "DPUK Data Access" section with Sarah Bauermeister and Jiahui Huang contacts
  - Removed mailing address section
- **Cohort detail page updates**:
  - Changed "CGMHC Team" to "CGMHC Collaborators"
  - Removed role display from collaborators (cleaner presentation)
- **Cohort name fixes**:
  - STARRS-LS: Updated to full name "Study to Assess Risk and Resilience in Servicemembers - Longitudinal Study (STARRS-LS)"
  - COVIDMENT: Added website link (https://covidment.is/)

### Updates (Session 4 - Data Restructuring)
- **Split investigators.json into 4 separate files** for easier maintenance:
  - `public/data/project.json` - Grant info, mission statement, scientific aims
  - `public/data/institutions.json` - Institution definitions
  - `public/data/team.json` - All team members/investigators
  - `public/data/cohort-associations.json` - Cohort-PI mappings and data access
- **Deleted old `public/investigators.json`** - no longer needed
- **Updated all page imports** to use new data file paths:
  - `src/app/page.tsx` - imports projectData, cohortAssociationsData
  - `src/app/investigators/page.tsx` - imports teamData, institutionsData
  - `src/app/cohorts/page.tsx` - imports cohortAssociationsData
  - `src/app/cohort/[id]/page.tsx` - imports cohortAssociationsData, teamData
- **Updated type definitions** in `src/lib/investigators.ts`:
  - Added ProjectData, InstitutionsData, TeamData, CohortAssociationsData interfaces
  - Made role and institution fields more flexible (string instead of strict types)

### Updates (Session 3)
- **Our Team page restructured**: Team members now grouped by roles WITHIN each institution (not across all sites)
  - Each institution shows its own Faculty, Postdocs, Data Analysts, Program Staff sections
  - Institutions ordered: MGB, Oxford, USP, UFRGS
- **CGMHC Collaborators feature**: Added support for linking CGMHC collaborators to specific cohorts
  - Added `cgmhcCollaborators` field to participatingCohorts in investigators.json
  - Cohort detail pages now show "CGMHC Collaborators" section with linked collaborators
  - Removed "(Main Contact)" labels from PI names
- **New team members added**:
  - Ellen Sweeney (Dalhousie University) - CGMHC Collaborator for CanPath
  - Yunsong Cui (Dalhousie University) - Data Analyst for CanPath
  - Added Dalhousie University as new institution
- **Landing page improvements**:
  - Clearer consortium description (what CGMHC does and its goals)
  - Added "Join the Consortium" section for interested cohorts
- **Cohort website links**: Added optional `website` field to cohorts
  - Added websites for: ABCD, All of Us, HRS, MGB Biobank, CanPath, UK Biobank, FinnGen
  - Displays as "Visit cohort website" link on cohort detail pages
- **Type updates**:
  - Added `website?: string` to Cohort interface
  - Added `CGMHCCollaborator` interface for collaborator linking

### Updates (Session 2)
- Added Contact page at `/contact` with consortium email (cgmhc@mgb.org) and program manager info
- Changed map coloring from national/international to **LMIC vs High-Income** classification
  - LMIC countries (amber): Brazil, India, Uganda
  - High-income countries (indigo): all others
  - Updated in `src/lib/constants.ts` with `LMIC_COUNTRIES` constant
- Restructured cohort detail page:
  - Grouped measures by construct with timing columns (Pre/Post-pandemic, Childhood/Adult)
  - Removed yes/no/partial text labels (icons only with instrument names)
  - Added PI contact info from investigators.json participatingCohorts
  - Removed Share button
- Enhanced collaborators section with larger 80x80px photo cards (like team members)
- Added headshots for consultants (Patel, Kessler, Hernán) and collaborators (Ebrahimi, Manfro)
- Changed scientific aims layout to single column (3 stacked rows)
- Added Contact link to navigation across all pages
- **Security fix**: Replaced `xlsx` with `exceljs` to resolve high severity vulnerability
- Added `data/` folder to `.gitignore` to keep source Excel files private
- Deleted unused default Next.js SVG files (file.svg, globe.svg, next.svg, vercel.svg, window.svg)

### Site Structure Updates
- Created new landing page at `/` with consortium overview, statistics, and scientific aims
- Moved cohort explorer to `/explorer` route
- Added Publications page placeholder at `/publications`
- Updated navigation order: Home | Our Team | Explorer | Publications | Contact
- Replaced "Junior Faculty" with "Faculty" in investigators.json
- Added `object-top` CSS to headshots for better face centering in circular crops

### Investigators Page Updates (January 25, 2026)
- Fixed headshot image paths for GitHub Pages deployment (added `assetPrefix` and `getImagePath` helper)
- Changed navigation from nested breadcrumb to parallel tabs
- Replaced Google Scholar with email contact icons
- Added ORCID icon support (icons appear when fields are populated in JSON)
- Participating cohorts now sorted alphabetically (removed DPUK/Federated grouping)
- Roles display from JSON data (supports custom role strings)
- Fixed role display to fall back to raw role string when not in predefined roleLabels mapping
- Added Consultants section (between Leadership and Team Members)
- Added Collaborators section (after Team Members)
- Added new institutions: UFRGS, Harvard
- Added email addresses for all team members
- Added new team members: Karmel Choi, Dia Kabir, Omid Ebrahimi, Gisele Manfro
- Added Consultants: Vikram Patel, Ronald Kessler, Miguel Hernán

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cgmhc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cgmhc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
