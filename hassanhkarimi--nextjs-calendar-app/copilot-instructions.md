## nextjs-calendar-app

> - MUST maintain `output: 'export'` in next.config.js

# Next.js Calendar App - Cursor Rules

## Project Patterns

### Static Export Mode
- MUST maintain `output: 'export'` in next.config.js
- All functionality must work in static HTML export mode
- No server-side rendering or API routes should be relied upon

### Routing and Views
- URL-based navigation with format: `/calendar?view=month|week|day&date=yyyy-MM-dd`
- Single main calendar page with conditional rendering based on view parameter
- All views (month, week, day) should use consistent component structure
- Prefer component-based views over separate page files
- Navigation uses URL parameters rather than distinct page routes

### Styling
- Use Tailwind CSS with custom component classes
- Component-specific styles are in dedicated CSS files under styles/components/
- Global styles in styles/index.css
- Use semantic color coding for events based on their type/title
- Fixed height for calendar cells ensures consistent layout
- Import all component styles in styles/index.css

### UI Components
- Use SVG icons for better visual appearance
- Consistent spacing and font sizes across components
- Modern, clean UI with minimal borders and subtle shadows
- Use grid layouts for calendar display
- Navigation buttons should follow the design in the monthly view
- Implement subtle hover effects on interactive elements

### Calendar View Components
- Each view (Month/Week/Day) should have its own component
- Views should share a common component structure pattern
- Calendar cell components should be reusable across views
- Calendar event components should be reusable across views
- Navigation should be handled by the parent calendar component

### Calendar Event Display
- Color code events based on their type/title for visual organization
- Limit displayed events in calendar cells to prevent overflow
- Provide clear date navigation with previous/next controls
- Maintain consistent event styling across all calendar views
- Use scale, shadow, and z-index for hover effects on events
- Keep hover effect styling consistent across day, week, and month views

### Event Interactions
- Add hover effects to all calendar events for better UX
- Use transform: scale(1.02) and box-shadow on hover
- Manage z-index to bring hovered events to the foreground
- Consider position changes for month view to prevent layout issues
- Implement onMouseOver and onMouseOut event handlers for interactions
- Use transition properties for smooth animation effects

### Authentication
- Support both demo mode and database authentication
- Demo mode uses client-side storage
- Database mode uses NextAuth.js with PostgreSQL
- Type safety for auth properties needs attention (authUser.name issue)
- Always provide fallback values for auth properties that might be undefined

### Component Structure
- Pages should be in the `/pages` directory following Next.js Pages Router patterns
- Reusable components should be in the `/components` directory
- Context providers should be in the `/context` directory
- Use typed state management with useState<any> for complex objects
- Component hierarchy: Calendar > [MonthView/WeekView/DayView] > CalendarCell > CalendarEvent

### Calendar Views
- Support day, week, and month views
- Handle event display and creation in all views
- Filter events properly for each calendar cell
- Ensure responsive design across all calendar views
- Use consistent styling between all calendar views
- Implement similar hover effects across all views

### Type Safety
- Properly type all React components
- Use TypeScript interfaces for event objects
- Handle nullable values with optional chaining
- Provide fallback values for potentially undefined properties
- Pay attention to authUser type usage
- Define Event interface for use across components

### Deployment
- Build configuration is critical - follow the Vercel deployment instructions
- Use environment variables for configuration
- Test static export locally before deployment

## User Preferences
- Clean, responsive UI
- Simple event management
- Intuitive navigation between calendar views
- Fast performance
- Visually distinct event categories
- Interactive UI elements with hover effects

## Project Evolution
- Original implementation used inline styles, now transitioning to Tailwind CSS classes
- Project supports both Pages Router and App Router, but deployment uses Pages Router
- Demo mode was added to simplify usage without database
- Recent UI update to monthly view established new design patterns
- Event color coding now based on event type/title
- Hover effects added to all calendar views for consistent interaction
- Month view refactored to use component-based structure and CSS classes

## Known Challenges
- Maintaining static export compatibility
- Handling authentication in static mode
- Ensuring proper calendar view rendering
- Managing events without server-side persistence in demo mode
- Type safety with authentication objects
- Limited space for displaying multiple events in calendar cells
- Consistent event styling across different calendar views
- Performance optimization for hover effects on mobile devices
- Z-index management for overlapping events on hover
- Inconsistent implementation between month and week/day views
- Conflicts between URL parameters and separate page files 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HassanHKarimi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
