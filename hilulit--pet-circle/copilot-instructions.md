## design-system-enforcement

> Enforces design system usage for every code change. All widgets must use centralized tokens from app_theme.dart — never hardcode colors, spacing, radii, or typography values. This rule ALWAYS applies.


# Design System Enforcement

Every code change in this project MUST use the centralized design system. This is non-negotiable.

## Mandatory Token Usage

### Colors
- ALWAYS use `AppColorsTheme.of(context)` for colors: `final c = AppColorsTheme.of(context);`
- NEVER hardcode hex values (`Color(0xFF...)`, `Colors.white`, etc.)
- Use: `c.white`, `c.offWhite`, `c.chocolate`, `c.pink`, `c.cherry`, `c.lightBlue`, `c.blue`, `c.lightYellow`, `c.black`
- For opacity: `c.chocolate.withValues(alpha: 0.5)` (NOT `withOpacity`)

### Spacing
- ALWAYS use `AppSpacing` tokens for padding, margins, and gaps:
  - `AppSpacing.xs` = 4, `AppSpacing.sm` = 8, `AppSpacing.md` = 16, `AppSpacing.lg` = 24, `AppSpacing.xl` = 32
- Use: `EdgeInsets.all(AppSpacing.md)`, `SizedBox(height: AppSpacing.sm)`
- For values between tokens (e.g., 12), use `AppSpacing.sm + 4` or the nearest token

### Border Radius
- ALWAYS use `AppRadii` tokens: `const BorderRadius.all(AppRadii.medium)`
- NEVER use `BorderRadius.circular(N)` with a raw number
- Tokens: `AppRadii.xs`(4), `.sm`(8), `.small`(12), `.medium`(16), `.large`(20), `.full`(100), `.pill`(999)

### Typography
- ALWAYS use `AppTextStyles` for text: `.heading1`, `.heading2`, `.heading3`, `.body`, `.bodyMuted`, `.caption`, `.badge`, `.button`
- Override only color or weight: `AppTextStyles.body.copyWith(color: c.chocolate, fontWeight: FontWeight.w600)`
- NEVER create inline `TextStyle` from scratch

## Mandatory Component Reuse

Before creating any new UI element, check `lib/widgets/` for existing components:
- Buttons: `PrimaryButton` (filled/outlined)
- Cards: `NeumorphicCard`
- Text fields: `LabeledTextField`
- Dropdowns: `AppDropdown`, `BreedSearchField`
- Toggles: `TogglePill`
- Status: `StatusBadge`
- Layout: `OnboardingShell`, `AppHeader`, `BottomNavBar`

## Mandatory Internationalization

ALL user-facing text MUST use localized strings. No hardcoded text in widgets.

- Access via `final l10n = AppLocalizations.of(context)!;` then `l10n.keyName`
- Localization files: `lib/l10n/app_en.arb` (English) and `lib/l10n/app_he.arb` (Hebrew)
- After adding new keys to `.arb` files, run `flutter gen-l10n` to regenerate
- NEVER hardcode user-facing strings like button labels, titles, descriptions, empty states, snackbar messages, or dialog text
- Hint text with example values (e.g., "e.g., 5mg") is acceptable as-is
- Data format strings (CSV headers, technical IDs) do not need localization
- Dropdown display text must be localized even if the stored value is an English key

### Adding a new l10n key
1. Add the key to `lib/l10n/app_en.arb` with English text
2. Add the same key to `lib/l10n/app_he.arb` with Hebrew translation
3. Run `flutter gen-l10n`
4. Use `l10n.newKey` in the widget

## Checklist for Every Code Change

Before completing any edit to a `.dart` file:
1. Are all colors from `AppColorsTheme.of(context)`?
2. Are all spacing values using `AppSpacing` tokens?
3. Are all border radii using `AppRadii` tokens?
4. Are all text styles using `AppTextStyles`?
5. Is `withValues(alpha:)` used instead of deprecated `withOpacity()`?
6. Are existing widgets from `lib/widgets/` reused where applicable?
7. Are ALL user-facing strings localized via `AppLocalizations`?
8. If new l10n keys were added, were they added to BOTH `app_en.arb` and `app_he.arb`?

---
> Source: [HiLuLiT/pet-circle](https://github.com/HiLuLiT/pet-circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
