## web-dev-portfolio-website

> This is a personal portfolio website for Joseph Williams, a Master's student in Computer Science at Auburn University. The site showcases his experience, projects, achievements, and skills in Software Engineering, Data Science, and AI/ML.

# LLM Context for Joseph Williams' Portfolio Website

## Project Overview
This is a personal portfolio website for Joseph Williams, a Master's student in Computer Science at Auburn University. The site showcases his experience, projects, achievements, and skills in Software Engineering, Data Science, and AI/ML.

## Tech Stack
- **Frontend**: HTML5, CSS3 (with custom theme system), JavaScript (vanilla)
- **Build Tool**: Vite
- **Deployment**: Netlify
- **API Integration**: Hugging Face API for AI summarization feature via Netlify serverless function

## Key Features
1. **Dark/Light Theme System**: Auto-detects system preferences with manual toggle option
2. **AI Summarization**: Uses Hugging Face API to summarize the "About Me" section
3. **Responsive Design**: Mobile-first approach with clean, professional styling
4. **Expandable Sections**: "View More" toggles for About, Experience, and Achievements
5. **Version Display**: Auto-updates version from package.json in footer

## Project Structure
- `/index.html` - Main HTML file with all content sections
- `/src/style.css` - All styling with CSS variables for theming
- `/src/main.js` - JavaScript for theme toggling, expandable sections, and AI features
- `/netlify/functions/summarize.js` - Serverless function for AI summarization
- `/public/` - Static assets (resume PDFs, images, etc.)
- `vite.config.js` - Vite build configuration
- `netlify.toml` - Netlify deployment configuration

## Sections
1. **Navigation**: Sticky navbar with links to all sections
2. **Introduction/Hero**: Name, tagline, social links
3. **About Me**: Bio, skills, tools, AI summarization feature
4. **Experience**: Work experience and research positions (expandable)
5. **Projects**: Technical projects with links and descriptions
6. **Achievements**: Scholarships, awards, certifications, memberships (expandable)
7. **Contact**: Email and social media links
8. **Footer**: Credits and version number

## Design System
- **Primary Color**: Bright blue (#384bf7)
- **Theme Variables**: CSS custom properties for light/dark mode
- **Typography**: Clean, readable fonts with consistent sizing
- **Spacing**: Consistent padding and margins using rem units
- **Cards**: Unified styling with max-width, shadows, and hover effects

## Development Notes
- The site follows semantic HTML practices
- All icons are from CDN sources (devicons, Wikipedia)
- Theme persistence uses localStorage
- Version number is dynamically injected from package.json during build
- Left-aligned text and headings for consistency across all sections

## Change History (Changelog)

All notable changes to this project are documented below.

**Note:** This changelog was formalized after initial development. Earlier entries are inferred from git commit history.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

### [Unreleased]

### [1.2.0] - 2026-02-25
#### Added
- ICPC 2025 USA Southeast Division 1 Regional 4th place achievement
- ColorStack organization membership (August 2024 - Present)
- Certifications section with AI/ML Engineering Certificate and Databricks credentials
- AI/ML Engineering Certificate mention in About Me section
- Handshake AI Fellow experience (September 2025 - February 2026)
- USAA Decision Science Intern position (May 2026 - August 2026)
- SVG icons for theme toggle (moon and sun) replacing emoji
- Validation link for Auburn AI/ML Engineering Certificate

#### Changed
- Updated About Me to reflect Master's student status at Auburn University
- Mentioned AI/ML Engineering Certificate alongside Magna Cum Laude
- Reordered sections: Experience now appears before Projects
- Left-aligned all section headings (h2, h3) and body text for consistency
- Updated all experience dates to full month names (e.g., "June" instead of "Jun")
- Consolidated Auburn research positions into single entry (August 2023 - December 2025)
- Added Computer Vision research under Dr. Brian Thurow to research experience
- Updated ACM membership end date to 2025 (from Present)
- Changed Competitive Programming Team role from "Active member" to "Competitor"
- Moved theme toggle button to vertical center of screen (right side)
- Simplified footer to only show "Back to Top" link
- Updated resume link to 2026 version
- Email address updated to @proton.mail domain
- Removed max-width constraints on experience and achievement cards for better alignment

#### Fixed
- Fixed footer buttons stacking issue - now properly displays inline
- Removed empty space between Business & Innovation and Contact sections
- Fixed alignment inconsistencies across Achievements section
- Fixed resume button path to correctly point to 2026 resume file

#### Removed
- Removed "Quick Links" navigation from footer
- Removed duplicate Licenses & Certifications section
- Removed Remote/Hybrid location designations from experience entries
- Removed ICPC competition mentions from ACM description (redundant with dedicated ICPC entries)

### [1.1.2] - 2025-01-27
#### Added
- **Smart theme system** with automatic system preference detection
- Auto-detects and follows user's system theme (light/dark mode)
- Real-time theme updates when system theme changes
- Smart button tooltips showing current theme mode
- Netlify configuration file for proper deployment settings

#### Fixed
- **CRITICAL**: Fixed navbar theme switching - navbar now properly adapts to light/dark mode
- Updated all hardcoded colors in navbar to use CSS theme variables
- Fixed navbar text readability in both light and dark themes
- Added smooth transitions for navbar background and border color changes
- Fixed version number display in footer (was showing "v{version}" placeholder)
- Updated resume files to latest versions
- **Netlify deployment issues** with proper Node version and build configuration

#### Changed
- Updated general typography (h1, h2, h3, p, ul) to use theme variables instead of hardcoded colors
- Improved overall theme consistency across all text elements
- Added subtle border to navbar for better visual definition in light mode
- Completely rewrote style guide to reflect current implementation with theme system
- Removed duplicate redirects file in favor of netlify.toml configuration

### [1.1.1] - 2025-01-27
#### Fixed
- **CRITICAL**: Fixed navbar theme switching - navbar now properly adapts to light/dark mode
- Updated all hardcoded colors in navbar to use CSS theme variables
- Fixed navbar text readability in both light and dark themes
- Added smooth transitions for navbar background and border color changes
- Fixed version number display in footer (was showing "v{version}" placeholder)
- Updated resume files to latest versions

#### Changed
- Updated general typography (h1, h2, h3, p, ul) to use theme variables instead of hardcoded colors
- Improved overall theme consistency across all text elements
- Added subtle border to navbar for better visual definition in light mode

### [1.1.0] - 2025-01-27
#### Added
- Floating dark/light mode toggle button with theme persistence
- Comprehensive "Achievements" section with scholarships, competitive programming awards, ACM membership, licenses & certifications, and business innovation award
- "View More" toggle functionality for About, Experience, and Achievements sections
- AI-powered bio summarization feature with Hugging Face API integration
- Formal changelog file linked in footer
- Theme-aware styling for all new components

#### Changed
- Unified icon sizes to 40px across hero and contact sections for consistency
- Improved card styling with unified max-width, centering, padding, and box shadows
- Centered and aligned all section headers (h2 and h3) with cards for visual consistency
- Limited visible cards in Experience and Achievements sections (showing 2 initially)
- Reordered About section content: teaser text with ellipsis, hidden extended content, grouped action buttons
- Updated project card links to display in blue accent color with proper hover effects
- Updated wording from "data analysis" to "data engineering" throughout site
- Removed "data science" from Contact section intro for clarity
- Styled AI summary card to use site's bright blue accent color with white text

#### Fixed
- Toggle button JavaScript to correctly show/hide hidden content without hiding teaser text
- Theme persistence across page reloads
- Consistent icon colors in both dark and light modes
- All "View More" toggles now work correctly across all sections

---

### [1.0.0] - 2024-06-09
#### Added
- Initial launch of portfolio website
- About, Projects, Experience, and Contact sections
- AI-powered bio summarization feature
- Versioning in footer (auto-updates from package.json)
- Changelog and style guide files

#### Changed
- Unified site styling and icon sizes for consistency
- Footer layout and content for clarity and fun
- Projects section restored to manual cards for reliability

#### Fixed
- Footer alignment and version display issues
- Contact and social icon sizing inconsistencies

---

### [Earlier Notable Changes] (2024-09 to 2025-05)

#### May 2025
- Updated resume (May 5)

#### February 2025
- Projects grammar fix (Feb 7)
- Resume fix (Feb 6)
- Footer edit (Feb 6)
- Fixed paths for Netlify deployment (Feb 6)

#### December 2024
- Updated resume (Dec 20)

#### November 2024
- Grammar improvements (Nov 25)
- View more button fixes (Nov 25)
- Multiple improvements: toggle for expandable sections, centered social icons, improved GitHub logo visibility, simplified JavaScript (Nov 25)
- Updated resume (Nov 12)
- Updated README.md (Nov 1)
- Refined about section, added contact icons, improved button hover styling, noted needed project section functionality (Nov 1)

#### October 2024
- Fixed navbar bugs (Oct 31)
- Added logo (Oct 31)
- Navbar color tweaks (Oct 31)
- Created working skeleton for website, fixed nav and hrefs, improved fonts and project previews (Oct 31)
- Improved nav and directory clarity (Oct 30)
- Simple nav bar (Oct 30)
- Deleted index.html (Oct 30)

#### September 2024
- Initial commits and project setup (Sep 19)
- Test and JSON file additions (Sep 19)

---

## Coding Conventions & Best Practices

### HTML
- Use semantic HTML5 elements
- Keep accessibility in mind (alt text, aria labels)
- Maintain consistent indentation (2 spaces)
- Use meaningful class names

### CSS
- Use CSS custom properties (variables) for theming
- Mobile-first responsive design
- Consistent spacing units (rem/em preferred)
- Group related styles together
- Theme variables for all colors

### JavaScript
- Vanilla JavaScript (no frameworks)
- Clear function names describing purpose
- Comment complex logic
- Use localStorage for persistence
- Handle errors gracefully

### Version Control
- Follow semantic versioning (MAJOR.MINOR.PATCH)
- Update CHANGELOG.md for all notable changes
- Update package.json version with releases
- Clear, descriptive commit messages

---

*This context file helps LLMs understand the project structure, history, and conventions when working with this codebase.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jwilliams2023) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
