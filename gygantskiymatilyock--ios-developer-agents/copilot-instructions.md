## ios-accessibility-validator

> Accessibility audit for iOS UI code


# iOS Accessibility Validator

When working with iOS UI code, check for accessibility compliance with Apple HIG and WCAG 2.2.

## VoiceOver Compatibility

Check accessibility labels:
- All interactive elements have meaningful accessibilityLabel
- Labels are concise and descriptive (not "Button" or "Image")
- Labels don't include element type (VoiceOver announces automatically)
- Images have accessibilityLabel or marked as decorative (.accessibilityHidden(true))
- Icon-only buttons have descriptive labels
- Labels update when state changes ("Play" -> "Pause")

Check traits:
- .button for tappable non-Button elements
- .header for section headers
- .link for navigation links
- .adjustable for sliders, steppers, pickers
- .selected for current selection

## Dynamic Type

Check text scaling:
- Text uses UIFont.preferredFont(forTextStyle:) or SwiftUI .font(.body/.title/etc.)
- Custom fonts use UIFontMetrics for scaling
- No hardcoded font sizes that won't scale
- Layouts don't clip at large accessibility sizes
- @ScaledMetric for custom spacing in SwiftUI

## Color Accessibility

Check contrast and color usage:
- Text meets 4.5:1 contrast (normal) or 3:1 (large text 18pt+)
- Information not conveyed by color alone
- Error states have icons/text, not just red color
- Uses semantic colors (UIColor.label, .systemBackground)
- Supports both light and dark mode

## Touch Targets

Check target sizes:
- All interactive elements are minimum 44x44 points
- Adequate spacing between targets
- contentEdgeInsets extends small visuals

## Motion

Check animation handling:
- Respects UIAccessibility.isReduceMotionEnabled
- Provides reduced-motion alternatives

## Output

Flag issues as:
- CRITICAL: Blocks users (cite WCAG criterion)
- HIGH/MEDIUM/LOW: With code fix examples

---
> Source: [gygantskiyMatilyock/ios-developer-agents](https://github.com/gygantskiyMatilyock/ios-developer-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
