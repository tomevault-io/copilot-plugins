## gym-tuna-ios

> - Comprehensive guide for converting Android Kotlin UI code to iOS SwiftUI

## SwiftUI Conversion Guidelines for Justlog iOS App

- Comprehensive guide for converting Android Kotlin UI code to iOS SwiftUI
- Focuses on maintaining exact functionality, styling, and architectural consistency
- Establishes standardized process for screen conversion across the Justlog iOS app

### Conversion Principles
- Follow existing iOS architecture and patterns
- Maintain pixel-perfect visual design
- Preserve exact functionality from Android implementation
- Use established repositories, models, and styling systems
- If we see android reference use ios libraries for that eg: use ios library against material design

### Architectural Requirements
- Navigation: Use NavCoordinator pattern and Screen enum
- ViewModels: Implement @MainActor ObservableObject pattern
- Repositories: Connect to existing data sources
- Models: Utilize existing domain models
- Styling: Leverage MaterialColors, MaterialSpacing, MaterialTypography

### File Structure Guidelines
- Base path: `Justlog/Presentation/[ScreenName]/`
- Main files:
  * `[ScreenName]View.swift`
  * `[ScreenName]ViewModel.swift`
  * `components/[ComponentName].swift` (if needed)

### Conversion Checklist
- [ ] Exact functional replication
- [ ] Pixel-perfect styling
- [ ] Proper SwiftUI lifecycle management
- [ ] Consistent error handling
- [ ] Accessibility support
- [ ] Dark mode compatibility

### Key Conversion Considerations
- Use existing infrastructure
- Maintain consistency with current screens
- Implement proper state management
- Follow iOS design best practices

### File Structure Patterns
- Domain/Models: Enums and data structures
  (WeightUnit, AppTheme, UserSubscription, etc.)
- Domain/Repositories: Protocol definitions for data access
- Data/Repositories: Implementation of repository protocols
- Presentation/[Screen]: Screen views and ViewModels
- Presentation/[Screen]/components: Reusable screen-specific components
- UI/Theme: Theme system (ThemeManager, ThemeColors, MaterialDesignTokens)
- UI/Components: Shared UI components

### Critical Success Factors
1. Theme Consistency: Every component must be theme-aware and use dynamic colors
2. Image Handling: All icons must have dark variants and proper switching logic
3. Architecture Preservation: Maintain MVVM + Clean Architecture patterns
4. Functionality Parity: Match Android behavior exactly
5. Navigation Integration: Properly integrate with existing NavGraph system

### Next Screen to Convert
The Workout session screen is next. When provided with Android Kotlin code, I should:
1. Identify all theme-dependent elements (colors, images)
2. Convert state management from Compose to SwiftUI patterns
3. Ensure all components are theme-aware from the start
4. Test theme switching works for all new components
5. Integrate with existing navigation and dependency injection

The key is to never use static colors and always implement theme awareness from the beginning, not as an afterthought.

---
> Source: [rimshadpcs/Gym-Tuna-ios](https://github.com/rimshadpcs/Gym-Tuna-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
