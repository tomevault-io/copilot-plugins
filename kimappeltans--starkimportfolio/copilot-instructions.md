## starkimportfolio

> When the user asks to sync or import courses from kim_portfolio or clean-portfolio, use this agent to pull course data and course page structure into this repo and recreate every course.


# Course sync agent

When the user asks to **sync courses**, **import courses from kim_portfolio / clean-portfolio**, or **recreate course pages** from the other portfolio(s), follow this rule.

## Source locations (on the user's machine)

- **kim_portfolio (root)**  
  - Course data: `kim_portfolio/data/courses.ts`  
  - Course list page: `kim_portfolio/app/courses/page.tsx`  
  - Course detail route: `kim_portfolio/app/courses/[slug]/page.tsx`  

- **clean-portfolio (inside kim_portfolio)** — **preferred source** (most complete: syllabus, learningOutcomes, assessmentMethods, testimonials, stats, facultyInfo, requiredTexts)  
  - Course data: `kim_portfolio/clean-portfolio/data/courses.ts`  
  - Course page component: `kim_portfolio/clean-portfolio/components/ui/sections/course-page.tsx`  

**Absolute paths** (use when reading files):
- `/Users/kimappeltans/projects/kim_portfolio/data/courses.ts`
- `/Users/kimappeltans/projects/kim_portfolio/clean-portfolio/data/courses.ts`
- `/Users/kimappeltans/projects/kim_portfolio/clean-portfolio/components/ui/sections/course-page.tsx`
- `/Users/kimappeltans/Desktop/kim_portfolio/` (alternative if the user keeps kim_portfolio on Desktop)

## What to do

1. **Read course data** from `clean-portfolio/data/courses.ts` (or kim_portfolio root if clean-portfolio is unavailable). Prefer clean-portfolio for full academic fields (syllabus, learningOutcomes, assessmentMethods, testimonials, stats, facultyInfo, requiredTexts).

2. **Normalize for this repo**  
   - This repo uses **React + Vite**, not Next.js. Use `image: string` only (no `StaticImageData`).  
   - If the source uses `image: string | StaticImageData`, treat every `image` as a string path (e.g. `"/images/marketingcourse.webp"`).  
   - Ensure every course has: `id`, `title`, `slug`, `image`, `description`, `longDescription`, `price`, `duration`, `category`, `technologies`, `highlights`.  
   - Copy over optional fields this repo supports: `courseCode`, `credits`, `prerequisites`, `corequisites`, `learningOutcomes`, `assessmentMethods`, `syllabus`, `requiredTexts`, `facultyInfo`, `academicLevel`, `difficultyLevel`, `testimonials`, `stats`.

3. **Update this repo**  
   - Write the full course list into **`src/data/courses.ts`** (replace the `courses` array and keep `teachingImpact`, `getCourseBySlug`; add `getAllCourseSlugs` if missing).  
   - Keep the **Course** and **Stat** types in this repo; they already support the above fields.

4. **Course detail page**  
   - This repo uses **`src/pages/CourseDetail.tsx`** (React Router) for `/course/:slug`.  
   - Ensure it renders all courses returned by `getCourseBySlug(slug)`.  
   - Optionally adapt layout or sections from `clean-portfolio/components/ui/sections/course-page.tsx` (syllabus timeline, testimonials, stats, faculty info) so that each course page here shows the same structure where data exists. Keep this site’s visual style (e.g. SpaceJourney, font-display, borders).

5. **Course list**  
   - **`src/components/CoursesSection.tsx`** (and any course grid) should list all courses from `src/data/courses.ts`. No change needed if it already maps over `courses`.

## Optional: run the sync script

If present, you can run:

```bash
node scripts/sync-courses-from-portfolio.js
```

The script reads from the clean-portfolio path and writes normalized course data to `src/data/courses.ts`. If the script is missing or fails, perform the steps above manually (read source file, normalize, write to this repo).

## Checklist

- [ ] Course data in `src/data/courses.ts` includes every course from the source.
- [ ] Each course has `image` as a string path.
- [ ] `getCourseBySlug(slug)` and (if used) `getAllCourseSlugs()` work.
- [ ] Course detail page renders all course fields used in the source (syllabus, outcomes, testimonials, etc.) where the data exists.
- [ ] Routes `/course/:slug` work for every course slug.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kimappeltans) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
