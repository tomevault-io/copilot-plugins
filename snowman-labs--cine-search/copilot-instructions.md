## cine-search

> 🎨 **Design & UI Rules - CineSearch**


🎨 **Design & UI Rules - CineSearch**

**📱 Multi-Platform Responsiveness**

**Mandatory Platform Structure:**
- `mobile/` - Widgets optimized for smartphones (< 768px)
- `tablet/` - Widgets optimized for tablets (768px - 1024px) 
- `web/` - Widgets optimized for browsers (> 1024px)
- `shared/` - Reusable widgets across platforms

**Responsive Layout:**
- Always use [ResponsiveLayout](cci:2://file:///Users/glaidsonmontanhini/CascadeProjects/cine_search/lib/features/movies/presentation/layouts/responsive_layout.dart:3:0-26:1) to switch between platforms
- Use [ResponsiveContainer](cci:2://file:///Users/glaidsonmontanhini/CascadeProjects/cine_search/lib/features/movies/presentation/layouts/responsive_layout.dart:28:0-51:1) for containers with max width
- Apply [ResponsiveUtils](cci:2://file:///Users/glaidsonmontanhini/CascadeProjects/cine_search/lib/shared/utils/responsive_utils.dart:4:0-58:1) for paddings, grids and breakpoints

**🏗️ Separation of Concerns**

**Pages (Screens):**
- Only structure and navigation logic
- **FORBIDDEN**: Business logic in pages
- **MANDATORY**: Use specific widgets for each section

**Widgets:**
- All visual logic must be in dedicated widgets
- Widgets should be **stateless** whenever possible
- Separate widgets by responsibility (e.g., `MovieCard`, `SearchBar`)

**📁 Widget Organization**

**Core Widgets** (`core/presentation/widgets/`):
- Reusable widgets across multiple features
- Design system components (buttons, cards, inputs)
- Base layouts and containers

**Feature Widgets** (`features/{feature}/presentation/widgets/`):
- Feature-specific widgets
- Organized by platform when necessary
- Shared widgets in `shared/` folder

**🎨 Design System**

**Centralized Theme:**
- Use only [AppTheme](cci:2://file:///Users/glaidsonmontanhini/CascadeProjects/cine_search/lib/core/presentation/theme/app_theme.dart:3:0-118:1) for colors and styles
- **FORBIDDEN**: Hardcoded colors in widgets
- Apply Material Design 3 consistently

**Standardized Components:**
- Reuse theme components (cards, buttons, inputs)
- Maintain visual consistency across platforms
- Use `BorderRadius.circular(12)` for cards
- Use `BorderRadius.circular(8)` for buttons/inputs

**📐 Responsiveness**

**Standard Breakpoints:**
- Mobile: < 768px (padding: 16px, grid: 2 columns)
- Tablet: 768px - 1024px (padding: 32px, grid: 3 columns)  
- Web: > 1024px (padding: 64px, grid: 4 columns, max-width: 1200px)

**Implementation:**
```dart
// ✅ CORRECT
ResponsiveLayout(
  mobile: MobileMovieList(),
  tablet: TabletMovieList(), 
  web: WebMovieList(),
)

// ❌ INCORRECT - logic in page
class HomePage extends StatelessWidget {
  Widget build(context) {
    return Column(
      children: [
        // UI logic here
      ],
    );
  }
}
undefined
🚫 Restrictive Rules

Pages CANNOT:

Contain complex presentation logic
Have extensive inline widgets
Make direct calls to controllers/providers
Widgets MUST:

Have single, well-defined responsibility
Be reusable when possible
Follow clear naming ({Purpose}Widget)
Implement proper keys for performance
✅ Best Practices

Performance:

Use const constructors whenever possible
Implement keys in list widgets
Avoid unnecessary rebuilds
Maintainability:

Document complex widgets
Use descriptive names for widgets
Keep widgets small (< 100 lines)
Accessibility:

Implement semanticsLabel on images
Use Tooltip on action buttons
Ensure adequate contrast
Testing:

All custom widgets must have widget tests
Test responsive behavior across breakpoints
Mock external dependencies in widget tests
Validate accessibility features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Snowman-Labs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Snowman-Labs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
