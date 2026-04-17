## unit-scan-notify

> - ALWAYS design and test for iPhone 13+ FIRST, then enhance for desktop


---
trigger: always_on
---

# SPR Vice City - Development Standards

## MOBILE-FIRST DEVELOPMENT
- ALWAYS design and test for iPhone 13+ FIRST, then enhance for desktop
- Use iPhone-specific breakpoints from tailwind.config.ts
- Minimum touch target size: 44px (iOS standard)
- Test on actual iPhone device via Lovable production URL before considering features complete
- Viewport targets: 390px (iPhone 13/14/15), 393px (Pro), 428px (Pro Max)

## PHOTO STORAGE PATTERN (CRITICAL - Fixed Oct 18, 2025)
- NEVER store base64 images in the database
- ALWAYS upload photos to Supabase Storage bucket: violation-photos
- Store ONLY the storage path in violation_photos.storage_path (format: {user_id}/{filename}.jpg)
- Use client-side compression: max 1600px, JPEG 80% quality
- Generate public URLs via supabase.storage.getPublicUrl(path) when displaying

## DATE FILTERING CONSISTENCY (Unified Oct 18, 2025)
- "This Week" = past 6 days + today (7 days total)
- "This Month" = 1st of current month through today
- ALWAYS normalize dates to midnight (strip time) before comparing
- ALWAYS prioritize occurred_at over created_at
- Use this pattern across ALL pages (Books, Export, Admin)

## TEAM VISIBILITY PRINCIPLE
- ALL users can view ALL violations (no user_id filtering on reads)
- Display user attribution via profile join
- Only admin can delete/edit violations
- NEVER filter violation_forms by user_id in Books, Export, or Admin pages

## 3D CAROUSEL CONSISTENCY
Reference: docs/3d-carousel.md
- Touch controls MUST be isolated to thumbnail cards only
- Parent container: pointerEvents: 'none', touchAction: 'pan-y'
- Individual cards: drag="x" when carousel is active
- Use same filtering logic across Books.tsx, Export.tsx, Admin.tsx

## DATABASE SCHEMA
- Primary table: violation_forms (bigint PK)
- Photos table: violation_photos (FK to violation_forms.id with ON DELETE CASCADE)
- Never use legacy violation_forms_backup_before_migration
- Always use proper joins for photos and profiles
- Storage bucket: violation-photos

## TYPESCRIPT & CODE QUALITY
- NO @ts-ignore comments allowed
- Regenerate Supabase types after schema changes: npx supabase gen types typescript --project-id fvqojgifgevrwicyhmvj > src/integrations/supabase/types.ts
- Use proper TypeScript types from src/integrations/supabase/types.ts
- Remove all debug console.log statements before committing

## GIT & DEPLOYMENT
- Commit frequently with descriptive messages
- Push to GitHub triggers auto-deploy on Lovable (1-2 min)
- Update CHANGELOG.md for all significant changes
- Update WORKFLOW_REVIEW.md when fixing critical issues
- Test on localhost first, then production URL
- Verify on actual iPhone device before considering complete

## DOCUMENTATION STANDARDS
- Keep docs/WORKFLOW_REVIEW.md as source of truth for system status
- Update docs/3d-carousel.md for carousel changes
- Reference existing .md files before asking questions
- Create new .md files for major features or fixes

## CRITICAL RULES - NEVER VIOLATE

NEVER:
- Store base64 images in database
- Filter violation_forms by user_id in Books/Export/Admin
- Use @ts-ignore without regenerating types
- Commit debug console.log statements
- Deploy without testing on actual iPhone
- Modify carousel without checking all three pages
- Change date filtering logic without updating all pages

ALWAYS:
- Upload photos to Supabase Storage
- Test mobile-first on iPhone viewport
- Use consistent date filtering across all pages
- Isolate carousel touch controls to cards
- Show all violations to all users (team transparency)
- Update CHANGELOG.md for significant changes
- Regenerate types after schema changes
- Test on production URL after deployment

## QUICK REFERENCE
- Source of Truth: docs/WORKFLOW_REVIEW.md
- Carousel Spec: docs/3d-carousel.md
- Production URL: https://spr-vicecity.lovable.app/
- Deploy: Auto from GitHub main (1-2 min)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RTSII) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
