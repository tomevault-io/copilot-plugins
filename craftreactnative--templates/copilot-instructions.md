## templates

> CraftReactNative Templates - A collection of copy-paste React Native templates built with Unistyles, TypeScript, and Expo. Contains 19 production-ready screen templates with dark mode support.

# AGENTS.md

## Project Overview

CraftReactNative Templates - A collection of copy-paste React Native templates built with Unistyles, TypeScript, and Expo. Contains 19 production-ready screen templates with dark mode support.

## Setup Commands

- Install dependencies: `cd demo-app && npm install`
- Start development server: `cd demo-app && npm start`
- Run on iOS: `cd demo-app && npm run ios`
- Run on Android: `cd demo-app && npm run android`

## Testing Instructions

- Run TypeScript check: `cd demo-app && npm run tsc`
- Run linter: `cd demo-app && npm run lint`
- Fix linting issues: `cd demo-app && npm run lint -- --fix`

## Code Style Guidelines

- **TypeScript strict mode** - Never use `any` type
- **No type assertions** - Use type guards instead of `as Type` casts
- **Feature-based colocation** - Group related files together by feature, not by type
- **Unistyles theming** - Use the unified theme system at `demo-app/craftrn-ui/themes/`
- **React Native best practices** - Follow performance and accessibility guidelines

## Template Structure

Each template in `demo-app/craftrn-templates/[TemplateName]/` contains:

- `AGENTS.md` - Template-specific agent instructions
- `info.json` - Dependencies, components, and file structure metadata
- Component files following colocation principles
- `data/` folder with mock data when applicable

## File Modification Rules

**When editing templates:**

- Always check `info.json` for dependencies and component requirements
- Preserve existing animation patterns (React Native Reanimated)
- Maintain craftrn-ui component usage for consistency
- Test on multiple platforms (iOS, Android, Web)
- Follow the existing file colocation structure

## Build and Deployment

- **Development builds**: Use `expo start` for live reloading
- **Production builds**: Use EAS Build for app store deployment
- **Testing on device**: Use development builds (Expo Go is not supported)

## Common Issues

- **Metro bundler issues**: Clear cache with `npx expo start --clear`
- **iOS simulator**: Ensure Xcode and simulators are installed
- **Android emulator**: Verify Android Studio and AVD setup
- **Type errors**: Run `npm run tsc` to check TypeScript issues
- **Missing dependencies**: Check template's `info.json` for required packages

---
> Source: [craftreactnative/templates](https://github.com/craftreactnative/templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
