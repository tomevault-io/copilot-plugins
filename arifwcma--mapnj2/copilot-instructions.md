## mapnj2

> 1. Be always brief and to-the-point with me.

1. Be always brief and to-the-point with me.

2. Turn off auto-web search mode. Don't invoke web search unless I explicitly ask.

3. Don't modify any code without asking me first. Sometimes I want explicit confirmation before writing code - wait for my "proceed" or similar confirmation.

4. Never write any comment in the code.

5. If my question is not specific enough, ask follow-up questions to specify it. But never give me broad answers or all possible cases answers.

6. PROJECT CONTEXT - mapnj2:
   - Next.js app for NDVI (Normalized Difference Vegetation Index) analysis using Google Earth Engine
   - Layout: 2/3 screen width for map (left), 1/3 screen width for work panel (right)
   - Work panel contains: Control panel (top, scrollable) and Info panel (bottom, fixed)
   - Features: Rectangle drawing, NDVI overlay, point analysis with historical chart, basemap switching
   - Chart library: Chart.js (react-chartjs-2) - replaced Recharts due to errors
   - Plot data stored in array (`plotData`) for navigation and future use
   - Arrow navigation: Left/right arrows below chart to expand range month by month
   - Boundaries: Left limit = Jan 2019, Right limit = current calendar month (not selected month)
   - Debouncing: 1 second delay for sliders and arrow clicks
   - State management: Uses hooks (useRectangleDraw, useNdviData), refs for debouncing
   - API endpoints: /api/count_available, /api/find_month, /api/ndvi/average, /api/ndvi/point, /api/ndvi/point/months
   - Earth Engine: Uses COPERNICUS/S2_SR_HARMONIZED collection, default cloud tolerance 30%
   - Point analysis: Shows NDVI value and 6-month historical chart (expandable with arrows)
   - Average NDVI displayed below chart arrows (excludes null values)

7. CODE STYLE:
   - Use absolute imports with @/ alias
   - Modular design: separate components, hooks, utilities
   - No comments in code
   - TypeScript/JavaScript mix (page.tsx is .tsx, some components are .jsx)
   - Follow React best practices: hooks, useCallback, useRef for performance

8. CURRENT FEATURE IN PROGRESS:
   - Adding "Compare with another point" functionality
   - State: `secondPointSelection` (boolean) to track if selecting second point
   - When active: hide "Click a point to analyse", show red "Click to choose the second point"
   - Second point clicks should NOT trigger first point analysis
   - For now: just show coordinates and place marker (testing phase)

9. When I say "@XXX" (or "XXX") - understand this:
   - Do not write code yet - we are talking. Confirm your understanding first.
   - Any opportunity for code refactoring to reduce duplication and make it more natural? Your opinion on feature. Your opinion on code risk.

10. When I say "@REFAC?" (or "REFAC?") - it means:
   - "Any refactor opportunity to reduce duplication and making things more simpler and natural (but without causing risk and complexities)"

11. When I say "@DC" (or "DC") - it means:
   - "Don't change code - just talking"

12. SHARE FEATURE REQUIREMENT:
   - Any new feature or state that affects the user's view/analysis must be included in the Share functionality
   - When adding new features, ensure:
     a) State is lifted to page.tsx if it needs to be shareable
     b) State is added to handleShare() serialization in page.tsx
     c) State is restored in the share URL loading useEffect in page.tsx
     d) State is passed as props to relevant components
   - Examples of shareable state: UI preferences, selected filters, chart settings, visible ranges, zoom levels, map bounds
   - If unsure whether something should be shareable, ask the user

---
> Source: [arifwcma/mapnj2](https://github.com/arifwcma/mapnj2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
